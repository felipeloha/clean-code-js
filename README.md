# clean-code-js
Cases around reading code

**[⬆ back to top](#table-of-contents)**

### formatting

**Bad:**

```javascript
formatShipToLabel = (shipToAddress) => SHIP_TO_DETAILS.map(item =>
    shipToAddress.address[`${item}`],
).filter(item =>
     item,
).join(', ');
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
getShipToValueFromSelectItems = (shipTo, addresses) => addresses.find(address => JSON.stringify(shipTo) === JSON.stringify(address.address))?.value || '';
```


### unnecessary size check to map a list with default

**Bad:**

```javascript
salesOrderState.get('thirdPartyAddresses').size > 0 ?
  salesOrderState.get('thirdPartyAddresses').map(tpa => (<MenuItem key={tpa.get('value')} value={tpa.get('value')}>
    {tpa.get('label')}
  </MenuItem>))
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


                          

