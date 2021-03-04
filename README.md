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
      error.general_error = i18nIntl().formatMessage({ id: 'error.no_stock' });
    else if (response.code === 422)
      error.general_error = i18nIntl().formatMessage({ id: 'error.422' });
    else
      error.general_error = i18nIntl().formatMessage({ id: 'error.creatingMultiplePicking' });

    dispatch(showErrors(error));
  })
  .catch(error => {
    //throw structured errors
    const _error = new Error();
    if (error.code === HTTP_STATUS_CODES.GATEWAY_TIMEOUT)
      _error.general_error = i18nIntl().formatMessage({ id: 'request.timeout.error' });
    else
      _error.general_error = i18nIntl().formatMessage({ id: 'request.general.error' });
    throw error;
  });
```
