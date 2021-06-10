# clean-code-js
Some cases we find around reviewing prs

### formatting

**Bad:**

```javascript
formatShipToLabel = (shipToAddress) => SHIP_TO_DETAILS.map(item =>
    shipToAddress.address[`${item}`],
).filter(item =>
     item,
).join(', ') || shipToAddress.name;
```

**Good:**

```javascript
formatShipToLabel = (shipToAddress) => 
  SHIP_TO_DETAILS
    .map(item => shipToAddress.address[`${item}`])
    .filter(item => item)
    .join(', ') || shipToAddress.name;
```


### conitional rendering

**Bad:**

```javascript
const shipToDetails = salesOrder.get('ship_to') !== null ? (<ShipToDetails salesOrder={salesOrder} />) : null;
```

**Good:**

```javascript
const shipToDetails = salesOrder.get('ship_to') && (<ShipToDetails salesOrder={salesOrder} />);
```


### find a value in a list and return a default

**Bad:**

```javascript
getShipToValueFromSelectItems = (shipTo, addresses) => {
    if (shipTo)
      return addresses.find(a => JSON.stringify(shipTo) === JSON.stringify(a.address)).value;
    return '';
}
```

**Good:**

```javascript
getShipToValueFromSelectItems = (shipTo, addresses) => 
    addresses.find(
        address => JSON.stringify(shipTo) === JSON.stringify(address.address)
    )?.value || '';
```


**Bad:**

```javascript
formatShipToValue = () => {
    const shipToIndex = salesOrderState.get('ship_to');
    if (shipToIndex !== '') {
      const shipTo = shipToAddresses.find(a =>
        a.value === shipToIndex,
      );
      return shipTo.address;
    }
    return null;
}
```

**Good:**

```javascript
formatShipToValue = () => {
    const address = shipToAddresses.find(a => 
            a.value && a.value === salesOrderState.get('ship_to')
            );
    return address?.address || null;
}
```
 


### unnecessary size check to map a list with default

**Bad:**

```javascript
salesOrderState.get('thirdPartyAddresses').size > 0 ?
  salesOrderState.get('thirdPartyAddresses').map(
    tpa => (
      <MenuItem key={tpa.get('value')} value={tpa.get('value')}>
        {tpa.get('label')}
      </MenuItem>
    )
  )
  : 
  []
```

**Good:**

```javascript
salesOrderState.get('thirdPartyAddresses').map(tpa => 
  (
    <MenuItem key={tpa.get('value')} value={tpa.get('value')}>
      {tpa.get('label')}
    </MenuItem>
  )
)
```


### flow control

**Bad:**

```javascript
if (shipTo) {
    const _selectItem = addresses.find(item => shipTo === item.address);

if (!_selectItem)
  otherOptions.push(Object.assign({}, {
    label: formatShipToLabel(shipTo.address),
    value: addressId,
    address: shipTo,
  }));
}
```

**Good:**

```javascript
if (shipTo && !addresses.some(item => shipTo === item.address))
    otherOptions.push(Object.assign({}, {
      label: formatShipToLabel(shipTo.address),
      value: addresses.length,
      address: shipTo,
    }));
```
       
**Bad:**
```javascript
const val = state.get('val');
if (val) {
  const nextVal = state.getIn(['nextObjext', 'nextval']);
  if (nextVal) {
    const result = doSomething(nextVal);
    return result.attr;
  }
}
return null;
```

**Good:**

```javascript
const val = state.get('val');
const nextVal = state.getIn(['nextObjext', 'nextval']);
if(!val || !nextVal) return null

return doSomething(nextVal).attr;
```   

### error handling

**Bad:**
```javascript
await apiClient.postSomething(params)
    .then(response => {
      if (response.status === HTTP_STATUS_CODES.GATEWAY_TIMEOUT)
        throw response;
      else
        return response.json();
    })
    .then(response => {
       // handles all possible 422 as a single one :(
      if (response.code === 422) {
        response.general_error = i18nIntl().formatMessage({ id: 'error.no_stock' });
      } else
        response.general_error = i18nIntl().formatMessage({ id: 'error.creatingMultiplePicking' });
        
      dispatch(showErrors(response));
    })
    .catch(errorResponse => {
        // handles all possible errors as a timeout :(
      response.general_error = i18nIntl().formatMessage({ id: 'request.timeout.error' });
      throw new Error('Error', errorResponse);
    });
```
  
**Better:**
```javascript
await apiClient.postSomething(params)
  .then(response => {
    if (response.status === HTTP_STATUS_CODES.GATEWAY_TIMEOUT) {
      //throw structured errors
      const error = new Error('timeout error');
      error.status = response.status;
      error.response = response;
      throw error;
    } else
      return response.json();
  })
  .then(response => {
    // define strucutured erores
    const error = {};
    
    // this should be a function but for the sake of the pattern
    if (response.code === 422 && response.message.contains('no_stock'))
      error.error_message = i18nIntl().formatMessage({ id: 'error.no_stock' });
    else if (response.code === 422)
      error.error_message = i18nIntl().formatMessage({ id: 'error.422' });
    else
      error.error_message = i18nIntl().formatMessage({ id: 'error.creatingMultiplePicking' });

    dispatch(showErrors(error));
  })
  .catch(error => {
    //throw structured errors
    const _error = new Error();
    if (error.code === HTTP_STATUS_CODES.GATEWAY_TIMEOUT)
      _error.error_message = i18nIntl().formatMessage({ id: 'request.timeout.error' });
    else
      _error.error_message = i18nIntl().formatMessage({ id: 'request.general.error' });
    throw error;
  });
```

### React test library - wait for elements
The following test will be unstable because it does not wait until the action is done
**Bad:**
```
describe('Period filter ', () => {
  test('should assert period filter range after clear all filters ', async () => {

    const salesOrdersMockData = { sales_orders: [], total_results: 0 };
    await renderApp(salesOrdersMockData);

    const mocks = await getNocksWhenFetchingSalesOrder(salesOrdersMockData);
    
    const periodFilter = screen.getByTestId('e2e-period-filter');
    userEvent.click(periodFilter);
    
    const periodFilterContent = screen.getByTestId('e2e-date-range-content');
    const next30Days = within(periodFilterContent).getByText('Next 30 days');
    expect(next30Days).toBeVisible();
    userEvent.click(next30Days.parentElement);
    
    await waitForMocksDone(mocks);
    
    const periodFilter = screen.getByTestId('e2e-period-filter');
    expect(periodFilter).toHaveTextContent('Next 30 days');

  }, 25000);

});
```

**Better:**
This test waits until the orders are rendered making sure the action was executed properly
```
describe('Period filter ', () => {
  test('should assert period filter range after clear all filters ', async () => {

    const salesOrdersMockData = { sales_orders: [], total_results: 0 };
    await renderApp(salesOrdersMockData);

    const mocks = await getNocksWhenFetchingSalesOrder(salesOrdersMockData);
    
    const periodFilter = screen.getByTestId('e2e-period-filter');
    userEvent.click(periodFilter);
    
    const periodFilterContent = screen.getByTestId('e2e-date-range-content');
    const next30Days = within(periodFilterContent).getByText('Next 30 days');
    expect(next30Days).toBeVisible();
    userEvent.click(next30Days.parentElement);
    
    await waitForMocksDone(mocks);
    const orders = screen.getByTestId('e2e-orders');
    await waitFor(() => expect(orders).toBeVisible(), { timeout: global.DEFAULT_TIMEOUT_2 });
    
    const periodFilter = screen.getByTestId('e2e-period-filter');
    expect(periodFilter).toHaveTextContent('Next 30 days');

  }, 25000);

});
```

### use of constants and avoiding making operations multiple times
**Bad:**
```
export function fetchMe() {
  return (dispatch) => apiClient.getMe()
    .then(response => response.json())
    .then(me => {
      if (!isEmpty(me.warehouses))
        apiClient.getWarehouses({ 'id[]': me.warehouses, sort_by: 'name', limit: 50 })
          .then(response => response.json())
          .then(warehouses => {
            dispatch(receiveMe(me, warehouses.warehouses));
          });
      else
        dispatch(receiveMe(me, []));
    });
}

...

export const fetchContext = (callback, authResponse = undefined) => async (dispatch) => {
  await authApi.getMe()
    .then(response => {
      const me = response.data;
      axios.all(
        [
          me.type === 'admin' ?
            api.getSettings({
              limit: 2147483647,
            }) :
            noop(),
          !isEmpty(me.warehouses) ?
            api.getWarehouses({
              'id[]': me.warehouses,
              sort_by: 'name',
              limit: 50,
            }) :
            noop(),
          me.type !== 'root' ?
            api.getClients({ limit: 2147483647 }) :
            noop(),
        ])
        .then(...)
    })
  };
}

....

render() {
  const { value, warehouses } = this.props;
  const menuItems = warehouses.slice(0, 50).map(w => <MenuItem key={w.get('id').toString()} value={w.get('id').toString()} disableRipple >{w.get('name')}</MenuItem>);
  ...
  return (...)
}
```

**Better:**
```

export function fetchMe() {
  return (dispatch) => apiClient.getMe()
    .then(response => response.json())
    .then(me => {
      if (!isEmpty(me.warehouses))
        apiClient.getWarehouses({ 'id[]': me.warehouses, sort_by: 'name', limit: MAX_WAREHOUSES_LIMIT })
          .then(response => response.json())
          .then(warehouses => {
            dispatch(receiveMe(me, warehouses.warehouses));
          });
      else
        dispatch(receiveMe(me, []));
    });
}

...

export const fetchContext = (callback, authResponse = undefined) => async (dispatch) => {
  await authApi.getMe()
    .then(response => {
      const me = response.data;
      axios.all(
        [
          me.type === 'admin' ?
            api.getSettings({
              limit: MAX_SAFE_LIMIT,
            }) :
            noop(),
          !isEmpty(me.warehouses) ?
            api.getWarehouses({
              'id[]': me.warehouses,
              sort_by: 'name',
              limit: MAX_WAREHOUSES_LIMIT,
            }) :
            noop(),
          me.type !== 'root' ?
            api.getClients({ limit: MAX_SAFE_LIMIT }) :
            noop(),
        ])
        .then(...)
    })
  };
}

....

render() {
  const { value, warehouses } = this.props;
  const menuItems = warehouses.map(w => <MenuItem key={w.get('id').toString()} value={w.get('id').toString()} disableRipple >{w.get('name')}</MenuItem>);
  ...
  return (...)
}
```


## Naming
General approach: One method should do one thing. Therefore it should be possible to name them in a precise way.

- Use CRUD and list operation names if possible: create, update, delete, get, map, filter, execute.
    - antipattern: execute + action
        - bad: executeAcceptPacking
        - good: acceptPacking
- Avoid general or non standardized names: process, turn, names without a verb in it, names proposing two actions in one method

### Naming Integers
Good: age, year, failureCount, numberOfFailures, tooltipShowDelayInMillis  
Bad: failures, tooltipShowDelay

For functions counting something: countFailures

### Naming Floating Point Numbers
price, height, moneyAmount
If more precision is needed: widthInCentimeters, angleInDegrees.

### Naming Booleans and validation Functions
Good: isDisabled, hasErrors, allowsWhitespace, didUpdate, shouldUpdate, willUpdate, areBoxesClosed  
Bad: disabled, errors, boxesClosed

### Naming Strings
In context of parsing values it is necessary to be more precise in the naming  
const year = parseInt(yearAsString, 10); 

### Naming Arrays, Lists, and Sets
Good: customers, errors, queueOfErrors, orderedCustomers  
Bad: customerList, handledOperationSet

if we respect the plurality agreement is not necessary to write the variable type

### Naming Maps
In general we should name them: keyToValueMap or for functions mapKeyToValue

Good: productIdToSuppliersMap or mapProductIdToSupplier  
Bad: mapProducts, productsMap

### Naming Pairs and Tuples
In general variable1AndVariable2

good: heightAndWidth, heightWidthAndLength  
bad: heigthWidth, result

### Naming Function Parameters
Modifying a function parameter is an anti-pattern. If the object needs to be modifies is it best to name the function parameters in a specific way:
```
function doSomething(person_){
  const person = {...person_};
  person.name = "pepito " + person.name
  return person;
}
```
 

### Actions and Handlers
Actions can be understood as something the user does and handlers as something the software does after the action

Good: closePacking, handlePackingClosed  
Bad: close, handlePacking
 

### Naming in testing
For Assertions the form assertSomethingPastAction is a good idea.

Good: renderOrderView, assertProductSelected, assertAcceptPackingOrderErrorDisplayed  
Bad: renderForm, check, checkProduct, assertAcceptPackingError

