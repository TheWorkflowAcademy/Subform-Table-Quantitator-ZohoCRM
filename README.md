# Subform-Table-Quantitator-ZohoCRM
A method to sum up the values of items in a subform in Zoho CRM.

## Core Idea
Suppose you have a subform in Deals that stores information (product, qty, subtotal) of every related sale. In order to find out how well each product is selling, you need get the sum of the qty/subtotal of each product sold in the subform across every Deal. This script does just that by creating a map of each product and going through each row on the subform that adds up qty/subtotal for each product per iteration. Upon completion, the product map will be updated with the final sum of qty/subtotal.

## Tutorial

### Create the Product Map
When a subform is created, all information on the form is stored in a "module" with the subform API name where you can simply execute a `zoho.crm.getRecords` to access it. To get the subform API name, you can either execute a `zoho.crm.getRecordsbyID` function or run a "Get List of Modules" API call (https://www.zoho.com/crm/developer/docs/api/v2/modules-api.html). 

Once you have the API name, iterate through each row in the subform to get the *key* that's the **Product**, and assign the default *value* of 0 for both **Quantity** and **Subtotal** in a map. At this point, you will have a map of all the products with 0 Quantity and Subtotal.

Note: `zoho.crm.getRecords` has a default limit of 200 records per page. To account for more than 200 records, pagination is required (learn more: https://github.com/TheWorkflowAcademy/api-pagination-zohocrm).

```javascript
producttable = zoho.crm.getRecords("INSERT SUBFORM API NAME HERE"); 
productmap = Map();
for each  p in producttable
{
	mp = Map();
	mp.put("Quantity",0);
	mp.put("Subtotal",0);
	prod = p.get("Product_Name");             //Replace this with the API name of your key
	productmap.put(prod.get("name"),mp);      //Replace this with the API name of your key
}
```
### Perform the Summation
Creat another `loop` to get the sum of the qty/subtotal for each product by adding them up at every iteration while remapping var *producttable*. This works because when the specified key is already present in the map-variable, the key's associated value is replaced with the new given value (learn more: 
https://www.zoho.com/creator/help/script/put-key.html). Once the iteration is complete, var *productmap* will contain the total qty & subtotal of the subform.

```javascript
for each  p in producttable
{
	mp = Map();
	mp.put("Quantity",productmap.get(p.get("Product_Name").get("name")).get("Quantity").toLong() + p.get("Quantity"));  
	mp.put("Subtotal",productmap.get(p.get("Product_Name").get("name")).get("Subtotal").toLong() + p.get("Subtotal"));
	productmap.put(p.get("Product_Name").get("name"),mp);
}
  //Replace the following with your API name - "Quantity", "Subtotal", "Product_Name", "name"
```

### Disclaimer
If you have null values in any of the subform columns, you'll need to add **null checks** to prevent the script from erroring.
