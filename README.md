# microsoft-license

There doesn't appear to be a maintained public api to map sku guids to product names or costs. There's only this [Microsoft reference](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/licensing-service-plan-reference) page. Worse still, their table is missing some products.


## 🧼 Acquiring a Clean Dataset

Going from an incomplete html table to a clean dataset takes a little bit of work. Here's how we're accomplishing it in javascript (but pick your poison 🧪).

```javascript


function loadProductsDatabase(){
  const prodGets = [
    axios.get('https://raw.githubusercontent.com/oc1/microsoft-license/master/raw.json'),
    axios.get('https://raw.githubusercontent.com/oc1/microsoft-license/master/additions.json'),
    axios.get('https://raw.githubusercontent.com/oc1/microsoft-license/master/costs.json')
  ];

  axios.all(prodGets)
  .then(rsp => transformProductsDatabase(rsp[0].data,rsp[1].data,rsp[2].data))
  .catch(err => console.log(err));
}
loadProductsDatabase();
setInterval(() => {loadProductsDatabase()}, 24*60*60*1000);


function transformProductsDatabase(raw, additions, costs){

  let cleanArr = [];

  //first, iterate over raw and:
  //  change the property names
  //  transform the includes string into an array of guids
  raw.forEach(x => {
    cleanArr.push({
      guid: x["GUID"],
      string_id: x["String ID"],
      name: x["Product name"],
      services: getArrayFromBigProductsString(x["Service plans included"]),
      cost: { month: null, year: null }
    });
  });

  //inject the additions
  additions.forEach(x => cleanArr.push(x));

  //inject the costs
  costs.forEach(x => {
    const i = cleanArr.findIndex(y => y.guid === x.guid);
    if (i > -1) cleanArr[i].cost = { month: x.month, year: x.year };
  });

  //place in cache
  cache.products = cleanArr;
  console.log(`Finished products table refresh with ${cleanArr.length} objects`);
}

function getArrayFromBigProductsString(messyString){
  let cleanArr = [];
  const rx = /(\{){0,1}[0-9a-fA-F]{8}\-[0-9a-fA-F]{4}\-[0-9a-fA-F]{4}\-[0-9a-fA-F]{4}\-[0-9a-fA-F]{12}(\}){0,1}/gm;
  const split_by_line = messyString.split(/\r?\n/);
  split_by_line.forEach(x => {
    const guid = x.match(rx);
    if (guid && guid.length > 0) cleanArr.push(guid[0]);
  });
  return cleanArr;
}

```


## 📋 Update Procedures

### Official Master Products List (`raw.json`)

Occasionally, Microsoft will update their reference. When that happens, a contributor to this repo can follow the steps below to update `raw.json`:

1. copy frustrating html table from official [Microsoft reference](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/licensing-service-plan-reference)
1. paste into blank sheet
1. copy the first four columns: `Product name` `String ID` `GUID` `Service plans included`
1. paste into [csv2json converter](https://csvjson.com/csv2json)
1. copy/paste output into `raw.json`

### Missing Products (`additions.json`)

Manually add objects to the array.

### Pricing Information (`costs.json`)

Manually add objects to the array.

