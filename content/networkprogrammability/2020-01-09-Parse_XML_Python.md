---
title: Parse XML file with Python
date: 2020-01-09T10:19:50+01:00
draft: false
categories:
  - Network Programming
  - Programming
tags:
  - Python
  - XML
---
### Introduction
We'll focus in this post on how to parse some data structures in Python. We'll start with XML, but in next posts we will also handle JSON and YAML.

### Sample file
I found a sample XML file [here](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/sample-xml-file-customers-and-orders-in-a-namespace) which we will use throughout this post. It contains an overview of customers and their respective orders. See below for a small excerpt. The complete file can be found in my Github repo.

```xml
<?xml version="1.0" encoding="utf-8"?>  
<Root xmlns="http://www.adventure-works.com">  
  <Customers>  
    <Customer CustomerID="GREAL">  
      <CompanyName>Great Lakes Food Market</CompanyName>  
      <ContactName>Howard Snyder</ContactName>  
      <ContactTitle>Marketing Manager</ContactTitle>  
      <Phone>(503) 555-7555</Phone>  
      <FullAddress>  
        <Address>2732 Baker Blvd.</Address>  
        <City>Eugene</City>  
        <Region>OR</Region>  
        <PostalCode>97403</PostalCode>  
        <Country>USA</Country>  
      </FullAddress>  
    </Customer>  
    <Customer CustomerID="HUNGC">   
    </Customer>  
    ...
  <Orders>  
    <Order>  
      <CustomerID>GREAL</CustomerID>  
      <EmployeeID>6</EmployeeID>  
      <OrderDate>1997-05-06T00:00:00</OrderDate>  
      <RequiredDate>1997-05-20T00:00:00</RequiredDate>  
      <ShipInfo ShippedDate="1997-05-09T00:00:00">  
        <ShipVia>2</ShipVia>  
        <Freight>3.35</Freight>  
        <ShipName>Great Lakes Food Market</ShipName>  
        <ShipAddress>2732 Baker Blvd.</ShipAddress>  
        <ShipCity>Eugene</ShipCity>  
        <ShipRegion>OR</ShipRegion>  
        <ShipPostalCode>97403</ShipPostalCode>  
        <ShipCountry>USA</ShipCountry>  
      </ShipInfo>  
   </Order>
   ...
  <Orders> 
```
### Code
We'll start with creating a Python virtual environment and the installation of the `xmltodict` library.
```bash
WAUTERW-M-65P7:Parse_XML_Python wauterw$ python3 -m venv venv
WAUTERW-M-65P7:Parse_XML_Python wauterw$ source venv/bin/activate
(venv) WAUTERW-M-65P7:Parse_XML_Python wauterw$ pip3 install xmltodict
```
Next, we'll write the Python script. As a first step, we need to import the `xmltodict` library and read the XML file into a variable. Hence (in our case), the entire XML file is stored into the variable called `xml_content`.

```python3
import xmltodict

with open('sample.xml') as f:
   xml_content = f.read()

#print(xml_content)
```
Obviously that doesn't really bring us something. Therefore we conver the XML content into a Python dictionary. Luckily, the `xmltodict` library has a method for doing this, e.g. the parse method. 

```python3
import xmltodict

with open('sample.xml') as f:
   xml_content = f.read()

xml_dict = xmltodict.parse(xml_content)
print(type(xml_dict))
```
If you print the type of this object, you will see we get returned an ordered dictionary.
```bash
(venv) WAUTERW-M-65P7:Parse_XML_Python wauterw$ python3 parseXML.py 
<class 'collections.OrderedDict'>
```
### Overview of customers
Let's write a small script to print an overview of all customers. If you take a look at the sample XML file, you'll see that the `Root` element contains a list of `Customers`.  Hence, in Python language, you can load all customers in a list by using `xml_dict['Root']['Customers']`.

After that, it's a matter of looping through a Python dictionary and printing the values.
```python3
import xmltodict

with open('sample.xml') as f:
   xml_content = f.read()

xml_dict = xmltodict.parse(xml_content)

customers = xml_dict['Root']['Customers']

for customer in customers['Customer']:
   print(f"Customer ID: {customer['@CustomerID']}")
   print(f"Company Name: {customer['CompanyName']}")
   print(f"Contact Name: {customer['ContactName']}")
   print(f"  ==>  Street: {customer['FullAddress']['Address']}")
   print(f"  ==>  City: {customer['FullAddress']['City']}")
   print(50 * "-")
```
This is the result of above script:
```bash
(venv) WAUTERW-M-65P7:Parse_XML_Python wauterw$ python3 parseCustomers.py 
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
Let's continue with something a little more complex. We want to print an overview of all orders per customers. This information is available in the sample XML file under the `Orders` attribute.

First of all, we will store all customers in a list called `customer_list` by using the `append` method. Hence, the variable `customer_list` contains a list of customers which we can loop through.

Next, we will loop through this list of customers (first for loop) and per customer we go over all the orders (second for loop) for that customer.

```python3
import xmltodict

with open('sample.xml') as f:
   xml_content = f.read()

xml_dict = xmltodict.parse(xml_content)

customers = xml_dict['Root']['Customers']
orders = xml_dict['Root']['Orders']
#print(orders)
customer_list = []
for customer in customers['Customer']:
   customer_list.append(customer['@CustomerID'])
   
for customer in customer_list: 
   print(f"Orders for: {customer}")
   for order in orders['Order']:
      if(customer == order['CustomerID']):
         print(f"  ==>Employee {order['EmployeeID']} placed an order on {order['OrderDate']}")
```
This will print the following output:
```bash
(venv) WAUTERW-M-65P7:Parse_XML_Python wauterw$ python3 parseOrders.py 
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
  ```

In this blog post, we demo'ed something fairly easy but it's one of those things you will use a  lot. The code can be found on my [Github](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Parse_XML_Python).