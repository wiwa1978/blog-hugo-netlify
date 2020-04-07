---
title: Parse JSON file with Python
date: 2020-01-11T10:19:50+01:00
draft: True
categories:
  - Network Programming
  - Programming
tags:
  - Python
  - JSON
---
### Introduction

### Sample file
I downloaded [this](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/sample-xml-file-customers-and-orders-in-a-namespace) XML file and converted it to JSON.

```json
{
   "Customers": {
      "Customer": [
         {
            "@CustomerID": "GREAL",
            "CompanyName": "Great Lakes Food Market",
            "ContactName": "Howard Snyder",
            "ContactTitle": "Marketing Manager",
            "Phone": "(503) 555-7555",
            "FullAddress": {
               "Address": "2732 Baker Blvd.",
               "City": "Eugene",
               "Region": "OR",
               "PostalCode": "97403",
               "Country": "USA"
            }
         },
         ...
   "Orders": {
      "Order": [
         {
            "CustomerID": "GREAL",
            "EmployeeID": "6",
            "OrderDate": "1997-05-06T00:00:00",
            "RequiredDate": "1997-05-20T00:00:00",
            "ShipInfo": {
               "@ShippedDate": "1997-05-09T00:00:00",
               "ShipVia": "2",
               "Freight": "3.35",
               "ShipName": "Great Lakes Food Market",
               "ShipAddress": "2732 Baker Blvd.",
               "ShipCity": "Eugene",
               "ShipRegion": "OR",
               "ShipPostalCode": "97403",
               "ShipCountry": "USA"
            }
         },

```


```bash
WAUTERW-M-65P7:Parse_JSON_Python wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Parse_JSON_Python wauterw$ source venv/bin/activate
```

```python3
import json

with open('sample.json') as f:
   json_content = f.read()

print(json_content)
```

```python3
import json

with open('sample.json') as f:
   json_content = f.read()

#print(json_content)

json_dict = json.loads(json_content)
print(type(json_dict))
```

```bash
(venv) WAUTERW-M-65P7:Parse_JSON_Python wauterw$ python3 parseCustomers.py 
<class 'dict'>
```


### Overview of customers
```python3
import json

with open('sample.json') as f:
   json_content = f.read()

#print(json_content)

json_dict = json.loads(json_content)
#print(type(json_dict))

customers = json_dict['Customers']
#print(customers)

for customer in customers['Customer']:
   print(f"Customer ID: {customer['@CustomerID']}")
   print(f"Company Name: {customer['CompanyName']}")
   print(f"Contact Name: {customer['ContactName']}")
   print(f"  ==>  Street: {customer['FullAddress']['Address']}")
   print(f"  ==>  City: {customer['FullAddress']['City']}")
   print(50* "-")
```

```bash
(venv) WAUTERW-M-65P7:Parse_JSON_Python wauterw$ python3 parseCustomers.py  
Customer ID: GREAL
Company Name: Great Lakes Food Market
Contact Name: Howard Snyder
  ==>  Street: 2732 Baker Blvd.
  ==>  City: Eugene
--------------------------------------------------
Customer ID: HUNGC
Company Name: Hungry Coyote Import Store
Contact Name: Yoshi Latimer
  ==>  Street: City Center Plaza 516 Main St.
  ==>  City: Elgin
--------------------------------------------------
Customer ID: LAZYK
Company Name: Lazy K Kountry Store
Contact Name: John Steel
  ==>  Street: 12 Orchestra Terrace
  ==>  City: Walla Walla
--------------------------------------------------
Customer ID: LETSS
Company Name: Let's Stop N Shop
Contact Name: Jaime Yorres
  ==>  Street: 87 Polk St. Suite 5
  ==>  City: San Francisco
--------------------------------------------------
```

### Overview of orders per customer
```python3
import json

with open('sample.json') as f:
   json_content = f.read()

json_dict = json.loads(json_content)

customers = json_dict['Customers']
orders = json_dict['Orders']
#print(customers)
customer_list = []
for customer in customers['Customer']:
   customer_list.append(customer['@CustomerID'])

for customer in customer_list: 
   print(f"Orders for: {customer}")
   for order in orders['Order']:
      if(customer == order['CustomerID']):
         print(f"  ==>Employee {order['EmployeeID']} placed an order on {order['OrderDate']}")
         
```

```bash
(venv) WAUTERW-M-65P7:Parse_JSON_Python wauterw$ python3 parseOrders.py 
Orders for: GREAL
  ==>Employee 6 placed an order on 1997-05-06T00:00:00
  ==>Employee 8 placed an order on 1997-07-04T00:00:00
  ==>Employee 1 placed an order on 1997-07-31T00:00:00
  ==>Employee 4 placed an order on 1997-07-31T00:00:00
  ==>Employee 6 placed an order on 1997-09-04T00:00:00
  ==>Employee 3 placed an order on 1997-09-25T00:00:00
  ==>Employee 4 placed an order on 1998-01-06T00:00:00
  ==>Employee 3 placed an order on 1998-03-09T00:00:00
  ==>Employee 3 placed an order on 1998-04-07T00:00:00
  ==>Employee 4 placed an order on 1998-04-22T00:00:00
  ==>Employee 4 placed an order on 1998-04-30T00:00:00
Orders for: HUNGC
  ==>Employee 3 placed an order on 1996-12-06T00:00:00
  ==>Employee 1 placed an order on 1996-12-25T00:00:00
  ==>Employee 3 placed an order on 1997-01-15T00:00:00
  ==>Employee 4 placed an order on 1997-07-16T00:00:00
  ==>Employee 8 placed an order on 1997-09-08T00:00:00
Orders for: LAZYK
  ==>Employee 1 placed an order on 1997-03-21T00:00:00
  ==>Employee 8 placed an order on 1997-05-22T00:00:00
Orders for: LETSS
  ==>Employee 1 placed an order on 1997-06-25T00:00:00
  ==>Employee 8 placed an order on 1997-10-27T00:00:00
  ==>Employee 6 placed an order on 1997-11-10T00:00:00
  ==>Employee 4 placed an order on 1998-02-12T00:00:00
WAUTERW-M-65P7:Parse_JSON_Python wauterw$ 
  ```


  [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Parse_JSON_Python)