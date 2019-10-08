
### Questions


### Objectives
YWBAT
* scrape data from a website (pagination?)
* grab the useful data from a webpage
* store this data in a dataframe
* save your dataframe as a csv file

### Outline


```python
import requests # grabbing the 'html-info'  from the url

import pandas as pd
import numpy as np

from bs4 import BeautifulSoup # a tool to traverse your html page or xml page or ... 


import matplotlib.pyplot as plt # maybe? 
```

### What should we scrape? 
- Ebay
- Sports - ESPN
- Steam
- Reddit - API (doesn't need to be scraped, PRAW)
- Medium

# let's scrape Medium


```python
url = "https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn={}"
urls = [url.format(i) for i in range(1, 21)]
urls
```




    ['https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=1',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=2',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=3',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=4',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=5',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=6',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=7',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=8',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=9',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=10',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=11',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=12',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=13',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=14',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=15',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=16',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=17',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=18',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=19',
     'https://www.ebay.com/sch/i.html?_nkw=usb+microphone+professional&_pgn=20']




```python
req = requests.get(url)
```


```python
soup = BeautifulSoup(req.content, 'html.parser')
```


```python
dictionary_list = []


for index, url in enumerate(urls):
    print(f"scraping page {index+1}")
    req = requests.get(url)
    soup = BeautifulSoup(req.content, 'html.parser')
    containers = soup.find_all("div", class_="s-item__info clearfix")

    for container in containers:
        d = {}
        d["name"] = container.find("h3", class_="s-item__title").text
        d["condition"] = container.find("span", class_="SECONDARY_INFO").text

        try:
            trending_price_text = container.find("span", class_="s-item__trending-price").text
            d["trending_price_type"], rest_of_trending = trending_price_text.split(":")
            d["trending_price_amount"] = rest_of_trending.split("$")[1]
        except:
            pass

        d["price"] = container.find("span", class_="s-item__price").text

        bold_list = container.find_all("span", class_="BOLD")
        for item in bold_list:
            if "Watching" in item.text:
                d["watching"] = item.text
                continue
            if "left" in item.text:
                d["items_left"] = item.text
                continue
            if "shipping" in item.text.lower():
                d["shipping_cost"] = item.text
                continue


        dictionary_list.append(d)
```

    scraping page 0
    scraping page 1
    scraping page 2
    scraping page 3
    scraping page 4
    scraping page 5
    scraping page 6
    scraping page 7
    scraping page 8
    scraping page 9
    scraping page 10
    scraping page 11
    scraping page 12
    scraping page 13
    scraping page 14
    scraping page 15
    scraping page 16
    scraping page 17
    scraping page 18
    scraping page 19



```python
df = pd.DataFrame(dictionary_list)
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
      <th>name</th>
      <th>condition</th>
      <th>trending_price_type</th>
      <th>trending_price_amount</th>
      <th>price</th>
      <th>shipping_cost</th>
      <th>items_left</th>
      <th>watching</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>USB Streaming Podcast PC Microphone, SUDOTACK ...</td>
      <td>Brand New</td>
      <td>Was</td>
      <td>67.49</td>
      <td>$53.99</td>
      <td>Free Shipping</td>
      <td>Only 1 left!</td>
      <td>5 Watching</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Blue Microphones Yeti Professional USB Condens...</td>
      <td>Pre-Owned</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>$78.00</td>
      <td>Free Shipping</td>
      <td>NaN</td>
      <td>17 Watching</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Marantz Professional Pod Pack 1 USB Microphone...</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>$59.95</td>
      <td>Free Shipping</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Professional USB Condenser Microphone Studio S...</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>$7.99</td>
      <td>Free Shipping</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Blue Yeti Blackout Professional Omnidirectiona...</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>$119.36</td>
      <td>Free Shipping</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.shape
```




    (985, 8)




```python
df.to_csv("ebay_microphone_information.csv", index=False)
```


```python

```


```python
my_new_df = pd.read_csv("ebay_microphone_information.csv")
my_new_df.head()
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
      <th>name</th>
      <th>condition</th>
      <th>trending_price_type</th>
      <th>trending_price_amount</th>
      <th>price</th>
      <th>shipping_cost</th>
      <th>items_left</th>
      <th>watching</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>0</td>
      <td>USB Streaming Podcast PC Microphone, SUDOTACK ...</td>
      <td>Brand New</td>
      <td>Was</td>
      <td>67.49</td>
      <td>$53.99</td>
      <td>Free Shipping</td>
      <td>Only 1 left!</td>
      <td>5 Watching</td>
    </tr>
    <tr>
      <td>1</td>
      <td>Blue Microphones Yeti Professional USB Condens...</td>
      <td>Pre-Owned</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>$78.00</td>
      <td>Free Shipping</td>
      <td>NaN</td>
      <td>17 Watching</td>
    </tr>
    <tr>
      <td>2</td>
      <td>Marantz Professional Pod Pack 1 USB Microphone...</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>$59.95</td>
      <td>Free Shipping</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>3</td>
      <td>Professional USB Condenser Microphone Studio S...</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>$7.99</td>
      <td>Free Shipping</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <td>4</td>
      <td>Blue Yeti Blackout Professional Omnidirectiona...</td>
      <td>Brand New</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>$119.36</td>
      <td>Free Shipping</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



### Assessments

- How to use BS4 library to scrape data from a website
- How to inspect the elements and find what to scrape for
- Good to get containers that contain the most information you want
- How to turn scraped data into a dataframe using a list of dictionaries


```python

```
