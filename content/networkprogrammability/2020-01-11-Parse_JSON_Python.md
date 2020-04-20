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
In this post, we have shown how to parse an XML file. In this one, we will focus on parsing the JSON variant. 

### Sample file
I downloaded [this](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/sample-xml-file-customers-and-orders-in-a-namespace) XML file and converted it to JSON. There's plenty of tools om the web to do this. I did it already for you, in case you want to follow along. Here it is:

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
### Code 

We’ll start with creating a Python virtual environment and the installation of the xmltodict library.

```bash
WAUTERW-M-65P7:Parse_JSON_Python wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Parse_JSON_Python wauterw$ source venv/bin/activate
```
Next, we’ll write the Python script. We'll first need to import the json package and read the JSON file into a variable.

```python3
import json

with open('sample.json') as f:
   json_content = f.read()

print(json_content)
```
Next, we will convert the file content into a dictionary, which is very easy with Python. See below. I'm printing also the type so you can see effectively it's a dict object.

```python3
import json

with open('sample.json') as f:
   json_content = f.read()

json_dict = json.loads(json_content)
print(type(json_dict))
```
Below is the output:

```bash
(venv) WAUTERW-M-65P7:Parse_JSON_Python wauterw$ python3 parseCustomers.py 
<class 'dict'>
```

### Overview of customers

Then let's make a small script to print an overview of all customers. Hence, in Python language, you can load all customers in a list by using json_dict['Customers'].

After that, it’s a matter of looping through a Python dictionary and printing the values

```python3
import json

with open('sample.json') as f:
   json_content = f.read()

json_dict = json.loads(json_content)

customers = json_dict['Customers']

for customer in customers['Customer']:
   print(f"Customer ID: {customer['@CustomerID']}")
   print(f"Company Name: {customer['CompanyName']}")
   print(f"Contact Name: {customer['ContactName']}")
   print(f"  ==>  Street: {customer['FullAddress']['Address']}")
   print(f"  ==>  City: {customer['FullAddress']['City']}")
   print(50* "-")
```
Here is the output:
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
Next, let's print an overview of all orders per customers. This information is available in the sample JSON file under the Orders attribute.

First of all, we will store all customers in a list called customer_list by using the append method. Hence, the variable customer_list contains a list of customers which we can loop through.

Next, we will loop through this list of customers (first for loop) and per customer we go over all the orders (second for loop) for that customer.

```python3
import json

with open('sample.json') as f:
   json_content = f.read()

json_dict = json.loads(json_content)

customers = json_dict['Customers']
orders = json_dict['Orders']

customer_list = []
for customer in customers['Customer']:
   customer_list.append(customer['@CustomerID'])

for customer in customer_list: 
   print(f"Orders for: {customer}")
   for order in orders['Order']:
      if(customer == order['CustomerID']):
         print(f"  ==>Employee {order['EmployeeID']} placed an order on {order['OrderDate']}")
         
```
Here is the output of this script:

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


You can find the code in my repo on [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Parse_JSON_Python).