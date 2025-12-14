# PowerBI_Paginated_Report
This is Power BI Paginated report using Snowflake 

## Requirement
1. Use Parameter in Power Query editor.
2. Use conditional where clause.
3. Defualt Parameter in Paginated report should be All and user can select multiple value.
4. DO not use any ODBC or other connect use only defualt connector to import data from Snowflake.

## Solution

1. Use Power Query to get data from Snowflake.
2. Use custom M-code and Parameters in Power Query.
3. Parameter name should be same in Power Query and Report Builder ( **Case sensitive** )


Pulling data from Snowflak  
  ```sql
#SQL Code that need to import in power query
SELECT * from   SALES.SALES 
WHERE product_id in (Parameter )  
and  SALES_BY in (Parameter);

```

We are creating dataset in paginated report by power query so ** we do not need to create Data Source ** 
just go to dataset and instaeat of add dataset click on Get Data

## Power Query 
1. Create connection with Snowflake and select your ** Table, View or Store Processor **
2. Goto transoform
3. Add new parameter and create text parameter
4. Name of parameters should be same with your report builder parameter ( Case Sensitive)
5. Go to advance Power Query editor and edit M-code

``` C#
// default M-code after import data from Snowflake to Power Quer
//  I did not use any snowflake role but you can use while connecting with snowflake 
let
  Source = Snowflake.Databases("Account identifier.snowflakecomputing.com", "COMPUTE_WH", [Role = null, CreateNavigationProperties = null, ConnectionTimeout = null, CommandTimeout = null, Implementation = "2.0"]),
  #"Navigation 1" = Source{[Name = "ROYAL_LTD", Kind = "Database"]}[Data],
  #"Navigation 2" = #"Navigation 1"{[Name = "SALES", Kind = "Schema"]}[Data],
  #"Navigation 3" = #"Navigation 2"{[Name = "SALES", Kind = "Table"]}[Data]
in
  #"Navigation 3"
```

Updated M-Code

``` C#

// Data query
let 

  Source = Snowflake.Databases(  "Account identifier.snowflakecomputing.com",  "COMPUTE_WH", 
    [Implementation = "2.0"] ), 

  DB = Source{[Name = "ROYAL_LTD", Kind = "Database"]}[Data], 
  
  // Build WHERE conditions dynamically 
  
  ProductFilter =  
    if  Product = "All" 
    then null 
    else "PRODUCT_ID IN ('" & Text.Replace(Product, ",", "','") & "')", 
  
  SalesByFilter =  
    if Sales_by = "All" then null  else "SALES_BY IN ('" & Text.Replace(Sales_by, ",", "','") & "')",  

// Combine conditions  

WhereClause =  
  Text.Combine( List.RemoveNulls(
          { ProductFilter, SalesByFilter }
          ),  " AND " ),  

 Sql  = 
 if  WhereClause = "" 
 then "SELECT * FROM SALES.SALES"  
 else "SELECT * FROM SALES.SALES WHERE " & WhereClause,  

 Result = Value.NativeQuery(DB, Sql)

 in 
Result

// First Parameter
"All" meta [IsParameterQuery = true, IsParameterQueryRequired = false, Type = type text]

// Second Paramenter
"All" meta [IsParameterQuery = true, IsParameterQueryRequired = false, Type = type text]

```

After Creating Dataset Now you need to create Dataset for Parameter if you have too many valuse in paramenter 
we have there way to pass value in parameter in Report builder
1. Create a Dataset and pass value from Dataset
2. Enter value manually
3. Make it free text do not pass any value

### Creating Paramenters in Report Builder

``` C#
// Fist Parameter Product_id
let
  Source = Snowflake.Databases("Account identifier.snowflakecomputing.com", "COMPUTE_WH", [Role = null, CreateNavigationProperties = null, ConnectionTimeout = null, CommandTimeout = null, Implementation = "2.0"]),
 DB = Source{[Name = "ROYAL_LTD", Kind = "Database"]}[Data],

SQL= 
"SELECT DISTINCT PRODUCT_ID From SALES.SALES
ORDER BY PRODUCT_ID ASC",
 
Result = Value.NativeQuery(DB,SQL)
in
Result

// Second Parameter Sales_by ( this is cascading by Product_id)

let
  Source = Snowflake.Databases("Account identifier.snowflakecomputing.com", "COMPUTE_WH", [Role = null, CreateNavigationProperties = null, ConnectionTimeout = null, CommandTimeout = null, Implementation = "2.0"]),
DB = Source{[Name = "ROYAL_LTD", Kind = "Database"]}[Data],
SQL = 
if Product = "All"
then
"SELECT DISTINCT SALES_BY FROM SALES.SALES
ORDER BY SALES_BY ASC;"
else
"SELECT DISTINCT SALES_BY FROM SALES.SALES
WHERE PRODUCT_ID in ('" & Text.Replace(Product, ",", "','") & "')
ORDER BY SALES_BY ASC;" ,


Result = Value.NativeQuery(DB,SQL)

in
 Result

``` 

After creating dataset of all parameter now you need to creat paramenter in Report Builder

## Creating Parameter in Report Builder
1. Name should be same as Power Query Paramenter name
2. Promnt you can give name as per your choise
3. ** Available Values** -  Get Value from Query ( Give value from Set)
4. ** Default Values  ** - you can give it No Default value or you can pass value from Specify values or you can pass value from a dataset
5. click on OK

Now create all require parameter after

### After creating all paramenter now set in Dataset
1. first go to cascading parameter dataset and set parameter setting

in my case  I have Sale_by Parameter cascading by Product_id  
So
1. I will goto Dataset properties by right clicking on dataset
2. Go Parameter
3. Add Parameter
4. Name should be same as Power Query Parameter
5. click on FX on Parameter Values. add Join Function
``` C#
=Join(Parameters!Product.Value, ",")
```
add paramenter to main dataset.
in my case I will
1. right click on **Sales** Dataset
2. click on Dataset Properties
3. Add Parameter (do same as on cascading parameter dataset)








