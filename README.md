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
  

