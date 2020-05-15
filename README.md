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

## Final Result and Sample Output

Depending upon how you perform your merges and transforms your output may vary. Here's some example output based upon the code above:

```json

{
"guid": "cdd28e44-67e3-425e-be4c-737fab2899d3",
"string_id": "O365_BUSINESS",
"name": "MICROSOFT 365 APPS FOR BUSINESS",
"services": [
"159f4cd6-e380-449f-a816-af1a9ef76344",
"094e7854-93fc-4d55-b2c0-3ab5369ebdc1",
"13696edf-5a08-49f6-8134-03083ed8ba30",
"e95bec33-7c88-4a70-8e19-b10bd9d0c014",
"a23b959c-7ce8-4e57-9140-b90eb88a9e97"
],
"cost": {
"month": 10,
"year": 8.25
}
},
{
"guid": "c2273bd0-dff7-4215-9ef5-2c7bcfb06425",
"string_id": "OFFICESUBSCRIPTION",
"name": "MICROSOFT 365 APPS FOR ENTERPRISE",
"services": [
"159f4cd6-e380-449f-a816-af1a9ef76344",
"43de0ff5-c92c-492b-9116-175376d08c38",
"13696edf-5a08-49f6-8134-03083ed8ba30",
"e95bec33-7c88-4a70-8e19-b10bd9d0c014",
"a23b959c-7ce8-4e57-9140-b90eb88a9e97"
],
"cost": {
"month": null,
"year": null
}
},
{
"guid": "3b555118-da6a-4418-894f-7df1e2096870",
"string_id": "O365_BUSINESS_ESSENTIALS",
"name": "MICROSOFT 365 BUSINESS BASIC",
"services": [
"5e62787c-c316-451f-b873-1d05acd4d12c",
"9aaf7827-d63c-4b61-89c3-182f06f82e5c",
"0f9b09cb-62d1-4ff4-9129-43f4996f83f4",
"159f4cd6-e380-449f-a816-af1a9ef76344",
"0feaeb32-d00e-4d66-bd5a-43b5b83db82c",
"c63d4d19-e8cb-460e-b37c-4d6c34603745",
"92f7a6f3-b89b-4bbd-8c30-809e6da5ad1c",
"b737dad2-2f6c-4c65-90e3-ca563267e8b9",
"c7699d2e-19aa-44de-8edf-1736da088ca1",
"e95bec33-7c88-4a70-8e19-b10bd9d0c014",
"a23b959c-7ce8-4e57-9140-b90eb88a9e97",
"57ff2da0-773e-42df-b2af-ffb7a2317929",
"7547a3fe-08ee-4ccb-b430-5077c5041653"
],
"cost": {
"month": 6,
"year": 5
}
},
{
"guid": "f245ecc8-75af-4f8e-b61f-27d8114de5f3",
"string_id": "O365_BUSINESS_PREMIUM",
"name": "MICROSOFT 365 BUSINESS STANDARD",
"services": [
"5e62787c-c316-451f-b873-1d05acd4d12c",
"8c7d2df8-86f0-4902-b2ed-a0458298f3b3",
"9aaf7827-d63c-4b61-89c3-182f06f82e5c",
"0f9b09cb-62d1-4ff4-9129-43f4996f83f4",
"159f4cd6-e380-449f-a816-af1a9ef76344",
"0feaeb32-d00e-4d66-bd5a-43b5b83db82c",
"199a5c09-e0ca-4e37-8f7c-b05d533e1ea2",
"5bfe124c-bbdc-4494-8835-f1297d457d79",
"094e7854-93fc-4d55-b2c0-3ab5369ebdc1",
"92f7a6f3-b89b-4bbd-8c30-809e6da5ad1c",
"b737dad2-2f6c-4c65-90e3-ca563267e8b9",
"c7699d2e-19aa-44de-8edf-1736da088ca1",
"e95bec33-7c88-4a70-8e19-b10bd9d0c014",
"a23b959c-7ce8-4e57-9140-b90eb88a9e97",
"57ff2da0-773e-42df-b2af-ffb7a2317929",
"7547a3fe-08ee-4ccb-b430-5077c5041653"
],
"cost": {
"month": 15,
"year": 12.5
}
},
{
"guid": "cbdc14ab-d96c-4c30-b9f4-6ada7cdc1d46",
"string_id": "SPB",
"name": "MICROSOFT 365 BUSINESS PREMIUM",
"services": [
"de377cbc-0019-4ec2-b77c-3f223947e102",
"5e62787c-c316-451f-b873-1d05acd4d12c",
"8c7d2df8-86f0-4902-b2ed-a0458298f3b3",
"176a09a6-7ec5-4039-ac02-b2791c6ba793",
"9aaf7827-d63c-4b61-89c3-182f06f82e5c",
"0f9b09cb-62d1-4ff4-9129-43f4996f83f4",
"159f4cd6-e380-449f-a816-af1a9ef76344",
"c1ec4a95-1f05-45b3-a911-aa3fa01094f5",
"8e9ff0ff-aa7a-4b20-83c1-2f636b600ac2",
"0feaeb32-d00e-4d66-bd5a-43b5b83db82c",
"199a5c09-e0ca-4e37-8f7c-b05d533e1ea2",
"5bfe124c-bbdc-4494-8835-f1297d457d79",
"094e7854-93fc-4d55-b2c0-3ab5369ebdc1",
"92f7a6f3-b89b-4bbd-8c30-809e6da5ad1c",
"b737dad2-2f6c-4c65-90e3-ca563267e8b9",
"bea4c11e-220a-4e6d-8eb8-8ea15d019f90",
"6c57d4b6-3b23-47a5-9bc9-69f17b4947b3",
"c7699d2e-19aa-44de-8edf-1736da088ca1",
"e95bec33-7c88-4a70-8e19-b10bd9d0c014",
"743dd19e-1ce3-4c62-a3ad-49ba8f63a2f6",
"a23b959c-7ce8-4e57-9140-b90eb88a9e97",
"57ff2da0-773e-42df-b2af-ffb7a2317929",
"8e229017-d77b-43d5-9305-903395523b99",
"7547a3fe-08ee-4ccb-b430-5077c5041653"
],
"cost": {
"month": null,
"year": 20
}
},

```