---
title: Python Webscraping with BeautifulSoup
date: 2014-04-12T20:19:50+01:00
draft: false
categories:
  - Development
  - Programming
tags:
  - Python
---
> Quick note: the original post dates from 12-04-2014 but got updated at 20-04-2020 with latest versions and it was essentially written from scratch again.


```
(base) WAUTERW-M-65P7:Python_Scraping_BeautifulSoup wauterw$ python3 -m venv venv
(base) WAUTERW-M-65P7:Python_Scraping_BeautifulSoup wauterw$ source venv/bin/activate
(venv) (base) WAUTERW-M-65P7:Python_Scraping_BeautifulSoup wauterw$ pip3 install beautifulsoup4
(base) WAUTERW-M-65P7:Python_Scraping_BeautifulSoup wauterw$ python3 -m pip install bs4
WAUTERW-M-65P7:site-packages wauterw$ /Applications/Python\ 3.8/Install\ Certificates.command
```

```python
from bs4 import BeautifulSoup
from urllib.request import urlopen

url="https://www.imdb.com/chart/top"
page=urlopen(url)
soup = BeautifulSoup(page, 'html.parser')

data = []
info = []
table = soup.find('table', attrs={'class':'chart'})
table_body = table.find('tbody')

rows = table_body.findAll('tr')
for row in rows:
   cols = row.findAll('td')
   for item in cols:
      text = item.find('a', href=True)
      if text:
         #print(text.get_text())
         data = text.get_text().rstrip()
         info.append(data)

i = 1
for item in info:
   if item:
      print(str(i) + ": "+ item)
      i  = i + 1
```