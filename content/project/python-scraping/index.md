---
title:  Scrap Content to build a Hugo Site - Part 1
summary: An example of scraping sites using python and creating a json file for use in Hugo
tags:
- Python
- Static Site
- Hugo
- Beautiful Soup
date: "2019-10-31T00:00:00Z"

# Optional external URL for project (replaces project detail page).
external_link: ""

image:
  caption: Python, Json and Hugo
  focal_point: Smart

links:
- icon: twitter
  icon_pack: fab
  name: Follow
url_code: "https://github.com/boreendesign/public_projects/blob/master/Scrape%20Content%20and%20Build%20Hugo%20Site.ipynb"
url_pdf: ""
url_slides: ""
url_video: ""


---
# Scrap Content and Build a Hugo Site

### Steps
* Setup and Helper Functions
* Import the Site Data from a CSV
* For Each Site for teaser content, scrape the data (Title, Price and Url to the more information page)
* Combine all the site data into a data frame
* Export this data to a json and csv file for use on the hugo site and other projects (Drupal Import )
* Build the hugo site

## Setup and Helper Functions

### Import the neccesary python modules


```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
from datetime import datetime
import re
import hashlib
import subprocess
```

### Variable Setup


```python
timestamp_full = datetime.today().strftime('%Y-%m-%d-%H:%M:')
timestamp_day  = datetime.today().strftime('%Y-%m-%d')

# Site Import Data
site_data_file = 'importfiles/sitedata.csv'
site_data_df = pd.DataFrame(columns=["company","site", "collection"])

# Site Export Data
scraped_data_file = 'exportfiles/scraped_sites'

```

### Standard Helper Functions


```python
# Log processing commands to see where a site may have failed
def log_processing(url):
    print('  ---> processing ' + url)

# Pull out everything but numbers and letters from the imported string
def returnNumbersAndLettersOnly(oldString):
    newString = ''.join(e for e in oldString if e.isalnum())
    return newString

# Pull out everything but numbers from the imported string
def returnNumbersOnly(oldString):
    newString = ''.join(e for e in oldString if e.isdigit())
    return newString

# Set default headers for call
# Testing out calls
def getHeadersObject(url):
    headers = {
        'User-Agent': "PostmanRuntime/7.18.0",
        'Accept': "*/*",
        'Cache-Control': "no-cache",
        'Accept-Encoding': "gzip, deflate",
        'Referer': url,
        'Connection': "keep-alive",
        'cache-control': "no-cache"
    }
    return headers

# Get the index of an item and handle the exception where the index does not exist and set to empty
def pop(item,index):

    try:
        return item[index]
    except IndexError:
        return 'null'

    return breakout
```

### Custom Functions


```python
def scrapeTeaserDataFromCollection_type1(url,site):

    site_scraped_data_df_temp = pd.DataFrame(columns=["site","title", "url","price"])
    soup = BeautifulSoup(requests.get(url).text, 'html.parser')
    resultsRow = soup.find_all('article', {'class': 'box-info'})

    for resultRow in resultsRow:
        resultRowBreakdown = resultRow.find('p', {'class': 'text-info'}).text.split('\n')
        try:
            site_scraped_data_df_temp = site_scraped_data_df_temp.append(
                {
                    'site':site,
                    'title':pop(resultRowBreakdown,0),
                    'price':returnNumbersOnly(pop(resultRowBreakdown,2)),
                    'url':resultRow.find('a').get('href')
                }
            , ignore_index=True)
        except IndexError as e:
            gotdata = ''

    return site_scraped_data_df_temp

def scrapeTeaserDataFromCollection_type2(url,site):

    site_scraped_data_df_temp = pd.DataFrame(columns=["site","title", "url","price"])
    soup = BeautifulSoup(requests.get(url).text, 'html.parser')
    resultsRow = soup.find('div', {'class': 'ppb_tour_classic'}).find_all('div', {'class': 'element'})

    for resultRow in resultsRow:
        try:
            site_scraped_data_df_temp = site_scraped_data_df_temp.append(
                {
                    'site':site,
                    'title':resultRow.find('h4').text, #<h2><a -.find('a'),
                    'price':returnNumbersOnly(resultRow.find('div', {'class': 'tour_price'}).text.strip('$')),
                    'url':resultRow.find('a',{'class': 'tour_link'}).get('href')
                }
            , ignore_index=True)
        except IndexError as e:
            gotdata = ''

    return site_scraped_data_df_temp
```

### Import the site data


```python
# Import and Print out the Sites you want to Scrape
site_data_df = pd.read_csv(site_data_file)
site_data_df
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
      <th>company</th>
      <th>site</th>
      <th>collection</th>
      <th>scrape_type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Example Site 1</td>
      <td>https://www.examplesite.com</td>
      <td>https://www.example.com/bloglist/</td>
      <td>type1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Example Site 2</td>
      <td>https://www.examplesite.com</td>
      <td>https://www.example.com/bloglist/</td>
      <td>type2</td>
    </tr>
  </tbody>
</table>
</div>



## Scrape Pages


```python
site_scraped_data_df = pd.DataFrame(columns=["site","title", "url","price"])

#Go Through All Sites and Scrape the Appropriate Data

for index, row in site_data_df.iterrows():
    log_processing(row['company'])
    if row['scrape_type'] == 'type1':
        site_scraped_data_df = site_scraped_data_df.append(scrapeTeaserDataFromCollection(row['collection'],row['company']), ignore_index=True)
    elif row['scrape_type'] == 'type2':
        site_scraped_data_df = site_scraped_data_df.append(scrapeTeaserDataFromCollection_type2(row['collection'],row['company']), ignore_index=True)
    else:
        print("NOT SCRAPE TYPE FOUND")



```



## Output results to CSV & JSON files


```python
#Output to CSV
site_scraped_data_df.to_csv(scraped_data_file+'.csv')
#Output to JSON (Orient will allow you to input this into Datatables.net structure for display on a site)
site_scraped_data_df.to_json(scraped_data_file+'.json',orient='records')
print(site_scraped_data_df)
```


[Get the code](https://github.com/boreendesign/public_projects/blob/master/Scrape%20Content%20and%20Build%20Hugo%20Site.ipynb)

PART 2 - Building the Hugo Site to display the information on Netlify (Coming Soon)
