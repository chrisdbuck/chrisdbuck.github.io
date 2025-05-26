---
layout: post
title: Web Scraping Exchange Rates
date: 2025-04-06 00:00:00 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: hmrc3.png # Add image post (optional)
tags: [Web Scraping, Beautiful Soup, Python] # add tag
---
When I report US income for UK taxes, I need to convert the dollar amounts to pounds using the official monthly exchange rates published by HMRC. Each exchange rate is on a different webpage, so retrieving a year’s worth of exchange rates can be time consuming. To solve this problem I use these Python functions that harness the power of Beautiful Soup to scrape the relevant exchange rates from the webpages for a given time period. The results are saved in a csv file so they can be used to calculate the pound of equivalent of dollar-based income in a spreadsheet.

## Libraries


```python
# Import the required libraries
import requests
import bs4
from bs4 import BeautifulSoup
import pandas as pd
import time
from datetime import datetime
```

## Functions


```python
def get_usdgbp_rate(year, month):
    """
    Scrape exchange USD/GBP rate for a specific year and month from HMRC website

    Args:
        year (int): Year of exchange rate
        month (int): Month of exchange rate
        
    Returns:
        dict: Dictionary containing USA exchange rate data for the specified month
    """

    url = f"https://www.trade-tariff.service.gov.uk/exchange_rates/view/{year}-{month}"

    # Headers to mimic a real browser request (helps avoid bot detection)
    HEADERS = {
    'User-Agent': ('Mozilla/5.0 (X11; Linux x86_64)' # Identifies the request as coming from a Linux browser
                    'AppleWebKit/537.36 (KHTML, like Gecko)' # Mimics a Chrome browser
                    'Chrome/44.0.2403.157 Safari/537.36'), # Specifies language preference to English
    'Accept-Language': 'en-US, en;q=0.5'
    }

    response = requests.get(url,  headers=HEADERS)
    if response.status_code != 200:
        print(f"Failed to retrieve the page. Status code: {response.status_code}")
    else:
        print(f"All Good. Status code: {response.status_code}")

    # Now pipe it into BS4
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # Get the table containing exchange rates
    table = soup.find('table', class_='govuk-table govuk-!-margin-top-6')

    # Extract table headers
    headers = []
    for th in table.find('thead').find_all('th'):
        headers.append(th.text.strip())

    # Find the USA row directly using text search
    usa_row = None
    for tr in table.find('tbody').find_all('tr'):
        first_td = tr.find('td')
        if first_td and ('USA' in first_td.text):
            usa_row = tr
            break

    # Extract the row data
    row_data = [td.text.strip() for td in usa_row.find_all('td')]

    # Create a row for the DataFrame
    data_row = {
        'month': datetime(year, month, 1).strftime('%B'),
        'year': year,
        'rate': row_data[3] if len(row_data) > 2 else None,
    }
    
    return data_row
```


```python
def get_all_usdgbp_rates(start_year, start_month, end_year, end_month):
    """
    Scrape USD/GBP exchange rates for a range of months and save as csv

    Args:
        start_year (int): Starting year
        start_month (int): Starting month
        end_year (int): Ending year
        end_month (int): Ending month
        
    Returns:
        pandas.DataFrame: DataFrame containing all USA exchange rates
        'usdgpb_rates.csv': a csv file of the DataFrame
    """
    all_data = []
    
    current_date = datetime(start_year, start_month, 1)
    end_date = datetime(end_year, end_month, 1)
    
    while current_date <= end_date:
        year = current_date.year
        month = current_date.month
        
        print(f"Scraping USA exchange rate for {year}-{month}...")
        
        usa_data = get_usdgbp_rate(year, month)
        if usa_data is not None:
            all_data.append(usa_data)
        
        # Add a delay to avoid overwhelming the server
        time.sleep(1)
        
        # Move to next month
        if current_date.month == 12:
            current_date = datetime(current_date.year + 1, 1, 1)
        else:
            current_date = datetime(current_date.year, current_date.month + 1, 1)
    
    # Create DataFrame from all collected data
    if all_data:
        df = pd.DataFrame(all_data)
        df.to_csv('usdgpb_rates.csv', index=False)
        return df
    else:
        return None
```

## Retrieve the exchange rates


```python
get_all_usdgbp_rates(2024, 4, 2025, 4)
```

    Scraping USA exchange rate for 2024-4...
    All Good. Status code: 200
    Scraping USA exchange rate for 2024-5...
    All Good. Status code: 200
    Scraping USA exchange rate for 2024-6...
    All Good. Status code: 200
    Scraping USA exchange rate for 2024-7...
    All Good. Status code: 200
    Scraping USA exchange rate for 2024-8...
    All Good. Status code: 200
    Scraping USA exchange rate for 2024-9...
    All Good. Status code: 200
    Scraping USA exchange rate for 2024-10...
    All Good. Status code: 200
    Scraping USA exchange rate for 2024-11...
    All Good. Status code: 200
    Scraping USA exchange rate for 2024-12...
    All Good. Status code: 200
    Scraping USA exchange rate for 2025-1...
    All Good. Status code: 200
    Scraping USA exchange rate for 2025-2...
    All Good. Status code: 200
    Scraping USA exchange rate for 2025-3...
    All Good. Status code: 200
    Scraping USA exchange rate for 2025-4...
    All Good. Status code: 200





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
      <th>month</th>
      <th>year</th>
      <th>rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>April</td>
      <td>2024</td>
      <td>1.2693</td>
    </tr>
    <tr>
      <th>1</th>
      <td>May</td>
      <td>2024</td>
      <td>1.2457</td>
    </tr>
    <tr>
      <th>2</th>
      <td>June</td>
      <td>2024</td>
      <td>1.2709</td>
    </tr>
    <tr>
      <th>3</th>
      <td>July</td>
      <td>2024</td>
      <td>1.2732</td>
    </tr>
    <tr>
      <th>4</th>
      <td>August</td>
      <td>2024</td>
      <td>1.3033</td>
    </tr>
    <tr>
      <th>5</th>
      <td>September</td>
      <td>2024</td>
      <td>1.3032</td>
    </tr>
    <tr>
      <th>6</th>
      <td>October</td>
      <td>2024</td>
      <td>1.3211</td>
    </tr>
    <tr>
      <th>7</th>
      <td>November</td>
      <td>2024</td>
      <td>1.2952</td>
    </tr>
    <tr>
      <th>8</th>
      <td>December</td>
      <td>2024</td>
      <td>1.2662</td>
    </tr>
    <tr>
      <th>9</th>
      <td>January</td>
      <td>2025</td>
      <td>1.2707</td>
    </tr>
    <tr>
      <th>10</th>
      <td>February</td>
      <td>2025</td>
      <td>1.2357</td>
    </tr>
    <tr>
      <th>11</th>
      <td>March</td>
      <td>2025</td>
      <td>1.2585</td>
    </tr>
    <tr>
      <th>12</th>
      <td>April</td>
      <td>2025</td>
      <td>1.2978</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
