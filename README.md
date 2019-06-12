
### Questions

### Objectives
YWBAT
* scrape data from a website (pagination)
* grab the useful data from a webpage
* store this data in a dataframe
* save your dataframe

### Outline


```python
import requests # grabbing the 'html-info'  from the url

import pandas as pd
import numpy as np

from bs4 import BeautifulSoup # a tool to traverse your html page or xml page or ... 


import matplotlib.pyplot as plt
```


```python
search = 'floor lamps'
url = "https://www.ebay.com/sch/i.html?_nkw={}".format(search.replace(" ", "+"))
url
page_url = "https://www.ebay.com/sch/i.html?_nkw={}&_pgn={}".format(search.replace(" ", "+"), 2)
```




    'https://www.ebay.com/sch/i.html?_nkw=floor+lamps'




```python
res_page = requests.get(url)
res_page.content[:50]
```




    b'<!DOCTYPE html><!--[if IE 9]><html class="ie9" lan'




```python
soup = BeautifulSoup(res_page.content, 'html.parser')
```


```python
# Let's build a function to make a soup object from a search
```


```python
def get_ebay_soup(search='floor lamps', page_number=1, parser='html.parser'):
    url = "https://www.ebay.com/sch/i.html?_nkw={}&_pgn={}".format(search.replace(" ", "+"), page_number)
    res_page = requests.get(url)
    soup = BeautifulSoup(res_page.content, 'html.parser')
    return soup
```


```python
def get_item_boxes(search, page, parser='html.parser'):
    soup = get_ebay_soup(search, page, parser)
    item_boxes = soup.find_all("div", attrs={"class":"s-item__wrapper"})
    return item_boxes
```


```python
def make_item_box_dict_list(item_boxes, dlist=[]):
    for index, item_box in enumerate(item_boxes):
        d = dict()

        try:
            d["price"] = float(item_box.find("span", class_="s-item__price").text.strip("$"))
        except:
            # maybe include average price
            continue

        d["condition"] = item_box.find("span", class_="SECONDARY_INFO").text

        name_text = item_box.find("h3").text.replace("SPONSORED", "SPONSORED$&~")
        if 'SPONSORED' in name_text:
            d["sponsored"] = name_text.replace("SPONSORED", "SPONSORED ").split("$&~")[0]
            d["name"] = name_text.replace("SPONSORED", "SPONSORED ").split("$&~")[1]
        else:
            d["name"] = name_text
            d["sponsored"] = None

        # shipping price
        d["shipping_price"] = item_box.find("span", class_="s-item__shipping s-item__logisticsCost").text

        dlist.append(d)
    return dlist
```


```python
def get_item_dict_list(search='floor lamps', page=1, parser='html.parser'):
    item_boxes = get_item_boxes(search=search, page=page, parser=parser)
    item_dict_list = make_item_box_dict_list(item_boxes)
    return item_dict_list
```


```python
item_dict_list = []
for page in range(1, 2):
    item_dict_list_sub = get_item_dict_list(search="avengers lunchboxes", page=page)
    item_dict_list.extend(item_dict_list_sub)
```


```python
df = pd.DataFrame(item_dict_list)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>condition</th>
      <th>name</th>
      <th>price</th>
      <th>shipping_price</th>
      <th>sponsored</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Brand New</td>
      <td>Marvel's Avengers: Infinity War Lunch Box  - O...</td>
      <td>13.99</td>
      <td>Free Shipping</td>
      <td>SPONSORED</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Brand New</td>
      <td>Marvel Kawaii Avengers Girls Boys Soft Insulat...</td>
      <td>18.99</td>
      <td>Free Shipping</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Pre-Owned</td>
      <td>Marvel Super Heroes Metal Lunchbox Thermos Set...</td>
      <td>59.50</td>
      <td>+$12.60 shipping</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Brand New</td>
      <td>Marvel Avengers Captain America Shield Shaped ...</td>
      <td>16.33</td>
      <td>Free Shipping</td>
      <td>SPONSORED</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Brand New</td>
      <td>Avengers Thermal Lunch Box New Kid Child's  ma...</td>
      <td>13.00</td>
      <td>Free Shipping</td>
      <td>SPONSORED</td>
    </tr>
  </tbody>
</table>
</div>




```python
soup = get_ebay_soup(search='floor lamps')
```


```python
soup.find_all("span", class_="s-item__price")[0] # querying the entire webpage
```




    <span class="s-item__price">$37.98</span>




```python
# How do I grab all of the item boxes?
# <li class="s-item   " "data-view"="..." "id"="...">
item_boxes = soup.find_all("div", attrs={"class":"s-item__wrapper"})
len(item_boxes)
```




    61




```python
# Pro Tip: Get everything ready for the first item you investigate

```


```python
item_box_1 = item_boxes[0]
# get price
price = float(item_box_1.find("span", class_="s-item__price").text.strip("$"))
condition = item_box_1.find("span", class_="SECONDARY_INFO").text
# get the name/sponsored boolean
name_text = item_box_1.find("h3").text.replace("SPONSORED", "SPONSORED$&~")
if 'SPONSORED' in name_text:
    sponsored = name_text.replace("SPONSORED", "SPONSORED ").split("$&~")[0]
    name = name_text.replace("SPONSORED", "SPONSORED ").split("$&~")[1]
else:
    name = name_text
    sponsored = None
    
# shipping price
shipping_price = item_box_1.find("span", class_="s-item__shipping s-item__logisticsCost").text
print(sponsored, name, price, shipping_price, condition)
```

    SPONSORED  Wood Shelf Floor Lamp Linen Shade Light Storage Organizer Living Room Black Home 37.98 Free Shipping Brand New



```python
# Now that we're able to get our information from 1 box, let's generalize and load into df
```


```python
make
```


```python
df = pd.DataFrame(dlist)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>condition</th>
      <th>name</th>
      <th>price</th>
      <th>shipping_price</th>
      <th>sponsored</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Brand New</td>
      <td>Wood Shelf Floor Lamp Linen Shade Light Storag...</td>
      <td>37.98</td>
      <td>Free Shipping</td>
      <td>SPONSORED</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Brand New</td>
      <td>Aged Brass Pharmacy Floor Lamp Adjustable Swin...</td>
      <td>99.99</td>
      <td>Free Shipping</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Brand New</td>
      <td>Floor Lamps Living Room Lamp Arc LED Modern Ov...</td>
      <td>54.02</td>
      <td>Free Shipping</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Brand New</td>
      <td>Floor Lamps Living Room Lamp Arc LED Modern Ov...</td>
      <td>49.99</td>
      <td>Free Shipping</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Brand New</td>
      <td>Modern LED Floor Lamp Reading Light Dimmable R...</td>
      <td>37.03</td>
      <td>Free Shipping</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python

```

### What did you learn?
* How strip works with strings
* Using sublime text
* how to maneuver through inpsection elements
* Use try/except
* The nature of web scraping is messy


```python
|
```
