# microsoft-license
There doesn't appear to be a maintained public api to map sku guids to product names or costs. There's only this [Microsoft reference](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/licensing-service-plan-reference) page. Worse still, their table is missing some products.



## acquiring a clean dataset
transforming

## Update Procedure

Occasionally, Microsoft will update their reference. When that happens, a contributor to this repo can follow the steps below to update `raw.json`:

1. copy frustrating html table from official [Microsoft reference](https://docs.microsoft.com/en-us/azure/active-directory/users-groups-roles/licensing-service-plan-reference)
1. paste into blank sheet
1. copy the first four columns: `Product name` `String ID` `GUID` `Service plans included`
1. paste into [csv2json converter](https://csvjson.com/csv2json)
1. copy/paste output into `raw.json`
