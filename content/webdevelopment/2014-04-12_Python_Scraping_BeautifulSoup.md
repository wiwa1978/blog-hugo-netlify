---
title: Python Webscraping with BeautifulSoup
date: 2014-04-12T20:19:50+01:00
draft: false
categories:
  - Development
  - Programming
  - All
tags:
  - Python
---
> Quick note: the original post dates from 12-04-2014 but got updated at 20-04-2020 with latest versions and it was essentially written from scratch again.
### Introduction
In this post, I will perform a little scraping exercise. Scraping is a software technique to automatically collect information from a webpage. Note: I have provided this example for illustrative purposes. It should be noted though scraping websites is not always allowed.

### What will we be doing?

In this post, I will be building a very small program that will scrape the top 250 of movies listed on the IMDB website.Luckily there is a URL provided by IMDB that will give us the 250 most popular movies already. This URL is http://www.imdb.com/chart/top. From that list, we are interested to find the titles and the rating for each movie

### What tools will we use?

Python seems to be the perfect candidate for this, although ruby could also be used in fact. Since we are using Python, we’ll also be using a little tool called BeautifulSoup (http://www.crummy.com/software/BeautifulSoup). This tool provides a couple of methods for navigating, searching and modifying a parse tree. So in other words, you provide the tool with the page you want to get info from, and it will allow you to find the particular piece of information you are searching for.

### Installation

Let's start with installing BeautifulSoup. In all honesty, had some issues installing it on my MAC despite the clear documentation. I tried the following:
```
(base) WAUTERW-M-65P7:Python_Scraping_BeautifulSoup wauterw$ python3 -m venv venv
(base) WAUTERW-M-65P7:Python_Scraping_BeautifulSoup wauterw$ source venv/bin/activate
(venv) (base) WAUTERW-M-65P7:Python_Scraping_BeautifulSoup wauterw$ pip3 install beautifulsoup4
```
Howeverm the above did not work. It appeared it worked but I always received a `module not found` error. I then found the following on StackExchange and luckily that helped.
```
(base) WAUTERW-M-65P7:Python_Scraping_BeautifulSoup wauterw$ python3 -m pip install bs4
WAUTERW-M-65P7:site-packages wauterw$ /Applications/Python\ 3.8/Install\ Certificates.command
``` 
With the installation out of the way, let's continue with the code. Create a file called scrape.py (or whatever you feel like). Import the BeatifulSoup tool as well as the urllib library. As mentioned above, BeautifulSoup will provide us all the methods needed for scraping the website while urllib is a library to open and handle URLs.

```python
from bs4 import BeautifulSoup
from urllib.request import urlopen
```
Obviously, we need to provide the url we would like to scrape, in our case this is the Top 250 IMDB list. Eventually, the complete html page will be loaded in the variable ‘soup’. We can now apply some methods to find the piece of info we are interested in.
```python
url="https://www.imdb.com/chart/top"
page=urlopen(url)
soup = BeautifulSoup(page, 'html.parser')
```
If you look carefully at the html code of the page, you will see that all data is in fact part of a table.
```html
<table class="chart full-width" data-caller-name="chart-top250movie">
   <colgroup>
      <col class="chartTableColumnPoster"/>
      <col class="chartTableColumnTitle"/>
      <col class="chartTableColumnIMDbRating"/>
      <col class="chartTableColumnYourRating"/>
      <col class="chartTableColumnWatchlistRibbon"/>
   </colgroup>
   <thead>
      <tr>
         <th></th>
         <th>Rank &amp; Title</th>
         <th>IMDb Rating</th>
         <th>Your Rating</th>
         <th></th>
      </tr>
   </thead>
   <tbody class="lister-list">
      <tr>
         <td class="posterColumn">...</td>
         <td class="titleColumn"><a href></a></td>
         <td class="ratingColumn">...</td>
         <td class="ratingColumn">...</td>
         <td class="watchlistColumn">...</td>
      </tr>
   </tbody> 
```
So with BeatifulSoup, we can find all the relevant data easily. First of all, we will need to find the <table> and the <tbody> element through BeautifulSoup. Once we have found the <tbody> element, we will find the <tr> element. Each title will correspond to a row, so we expect a list of rows through which we can iterate with a for-loop. For each row, we find the <td> elements (columns). Again this is a list through which we can iterate. There we will find the <a> attribute amd store the text in a variable 'text' which we eventually append to a list called 'data'.
```python
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
         data = text.get_text().rstrip()
         info.append(data)
```
In info, you will find the following type of info:
```
'The Lord of the Rings: The Fellowship of the Ring', '', 'Fight Club', '', 'Forrest Gump', '', 'Inception', '', 'Star Wars: Episode V - The Empire Strikes Back', '', 'The Lord of the Rings: The Two Towers', '', 'The Matrix', '', 'Goodfellas', '', "One Flew Over the Cuckoo's Nest", '', 'Seven Samurai', '', 'Se7en', '', 'City of God', '', 'Life Is Beautiful', '', 'The Silence of the Lambs', '', "It's a Wonderful Life", '', 'Star Wars: Episode IV - A New Hope', '', 'Parasite', '', 'Saving Private Ryan', '', 'Spirited Away', '', 'The Green Mile', '', 'Interstellar', ''
```
We can loop through list list in order to print it out to the screen. You will have noticed that there is a empty element each time, followed by a title. To filter out this whitespace, I have build in a small check to verify is item is not empty.
```python
i = 1
for item in info:
   if item:
      print(str(i) + ": "+ item)
      i  = i + 1
```
Hope you enjoyed this little example. Code can be found [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Python_Scraping_BeautifulSoup).