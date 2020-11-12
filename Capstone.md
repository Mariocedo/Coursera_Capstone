# Capstone Project

## Week 1

In this file I will mostly develop the Capstone Project...


```python
import pandas as pd
import numpy as np
```


```python
print("Hello Capstone Project Course!")
```

    Hello Capstone Project Course!


## Week 3 - 1


```python
#!pip3 install beautifulsoup4 requests
import requests
from bs4 import BeautifulSoup
```


```python
# Get html content from url

headers = {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'GET',
    'Access-Control-Allow-Headers': 'Content-Type',
    'Access-Control-Max-Age': '3600',
    'User-Agent': 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0'
    }

url = "https://en.wikipedia.org/wiki/List_of_postal_codes_of_Canada:_M"
req = requests.get(url, headers)
soup = BeautifulSoup(req.content, 'html.parser')

```


```python
# Find table

table = soup.find_all("table")[0]

# html -> pandas dataframe

df = pd.read_html(str(table))[0]
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
      <th>Postal Code</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>M1A</td>
      <td>Not assigned</td>
      <td>Not assigned</td>
    </tr>
    <tr>
      <th>1</th>
      <td>M2A</td>
      <td>Not assigned</td>
      <td>Not assigned</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M3A</td>
      <td>North York</td>
      <td>Parkwoods</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M4A</td>
      <td>North York</td>
      <td>Victoria Village</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M5A</td>
      <td>Downtown Toronto</td>
      <td>Regent Park, Harbourfront</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Data clean

# Ignore cells with a borough that is Not assigned.
df = df[df['Borough'] != "Not assigned"]

# If a cell has a borough but a Not assigned neighborhood, then the neighborhood will be the same as the borough.
df[df['Neighbourhood'] == "Not assigned"]["Neighbourhood"] = df[df['Neighbourhood'] == "Not assigned"]["Borough"]

df
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
      <th>Postal Code</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>M3A</td>
      <td>North York</td>
      <td>Parkwoods</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M4A</td>
      <td>North York</td>
      <td>Victoria Village</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M5A</td>
      <td>Downtown Toronto</td>
      <td>Regent Park, Harbourfront</td>
    </tr>
    <tr>
      <th>5</th>
      <td>M6A</td>
      <td>North York</td>
      <td>Lawrence Manor, Lawrence Heights</td>
    </tr>
    <tr>
      <th>6</th>
      <td>M7A</td>
      <td>Downtown Toronto</td>
      <td>Queen's Park, Ontario Provincial Government</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>160</th>
      <td>M8X</td>
      <td>Etobicoke</td>
      <td>The Kingsway, Montgomery Road, Old Mill North</td>
    </tr>
    <tr>
      <th>165</th>
      <td>M4Y</td>
      <td>Downtown Toronto</td>
      <td>Church and Wellesley</td>
    </tr>
    <tr>
      <th>168</th>
      <td>M7Y</td>
      <td>East Toronto</td>
      <td>Business reply mail Processing Centre, South C...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>M8Y</td>
      <td>Etobicoke</td>
      <td>Old Mill South, King's Mill Park, Sunnylea, Hu...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>M8Z</td>
      <td>Etobicoke</td>
      <td>Mimico NW, The Queensway West, South of Bloor,...</td>
    </tr>
  </tbody>
</table>
<p>103 rows × 3 columns</p>
</div>




```python
# More than one neighborhood can exist in one postal code area. For example, in the table on the Wikipedia page, you will notice that M5A is listed twice and has two neighborhoods: Harbourfront and Regent Park. These two rows will be combined into one row with the neighborhoods separated with a comma as shown in row 11 in the above table.

def join_Neighbourhood(x):
    return [value + "," for value in x][0][:-1]

df = df.groupby(by=["Postal Code", "Borough"])['Neighbourhood'].agg([join_Neighbourhood])

df.reset_index(inplace=True)

```


```python
df.shape
```




    (103, 3)



# Week 3 - 2


```python
#!pip install geocoder
import geocoder

# fetch all latitudes/longitudes

latitude=[]
longitude=[]
for code in df['Postal Code']:
    g = geocoder.arcgis('{}, Toronto, Ontario'.format(code))
    while (g.latlng is None):
        g = geocoder.arcgis('{}, Toronto, Ontario'.format(code))
    latlng = g.latlng
    latitude.append(latlng[0])
    longitude.append(latlng[1])

```

    Collecting geocoder
      Downloading geocoder-1.38.1-py2.py3-none-any.whl (98 kB)
    [K     |████████████████████████████████| 98 kB 6.7 MB/s  eta 0:00:01
    [?25hRequirement already satisfied: future in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from geocoder) (0.18.2)
    Requirement already satisfied: six in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from geocoder) (1.15.0)
    Requirement already satisfied: click in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from geocoder) (7.1.2)
    Collecting ratelim
      Downloading ratelim-0.1.6-py2.py3-none-any.whl (4.0 kB)
    Requirement already satisfied: requests in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from geocoder) (2.24.0)
    Requirement already satisfied: decorator in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from ratelim->geocoder) (4.4.2)
    Requirement already satisfied: chardet<4,>=3.0.2 in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from requests->geocoder) (3.0.4)
    Requirement already satisfied: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from requests->geocoder) (1.25.9)
    Requirement already satisfied: certifi>=2017.4.17 in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from requests->geocoder) (2020.6.20)
    Requirement already satisfied: idna<3,>=2.5 in /opt/conda/envs/Python-3.7-main/lib/python3.7/site-packages (from requests->geocoder) (2.9)
    Installing collected packages: ratelim, geocoder
    Successfully installed geocoder-1.38.1 ratelim-0.1.6



```python
# Add columns to dataframe

df["Latitude"] = latitude
df["Longitude"] = longitude

```


```python
df.rename(columns={"join_Neighbourhood": "Neighbourhood"}, inplace=True)
```


```python
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
      <th>Postal Code</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>M1B</td>
      <td>Scarborough</td>
      <td>Malvern, Rouge</td>
      <td>43.81139</td>
      <td>-79.19662</td>
    </tr>
    <tr>
      <th>1</th>
      <td>M1C</td>
      <td>Scarborough</td>
      <td>Rouge Hill, Port Union, Highland Creek</td>
      <td>43.78574</td>
      <td>-79.15875</td>
    </tr>
    <tr>
      <th>2</th>
      <td>M1E</td>
      <td>Scarborough</td>
      <td>Guildwood, Morningside, West Hill</td>
      <td>43.76575</td>
      <td>-79.17470</td>
    </tr>
    <tr>
      <th>3</th>
      <td>M1G</td>
      <td>Scarborough</td>
      <td>Woburn</td>
      <td>43.76812</td>
      <td>-79.21761</td>
    </tr>
    <tr>
      <th>4</th>
      <td>M1H</td>
      <td>Scarborough</td>
      <td>Cedarbrae</td>
      <td>43.76944</td>
      <td>-79.23892</td>
    </tr>
  </tbody>
</table>
</div>



# Week 3 - 3


```python
pd.set_option('display.max_columns', None)
pd.set_option('display.max_rows', None)

import json # library to handle JSON files

#!conda install -c conda-forge geopy --yes # uncomment this line if you haven't completed the Foursquare API lab
from geopy.geocoders import Nominatim # convert an address into latitude and longitude values

import requests # library to handle requests
from pandas.io.json import json_normalize # tranform JSON file into a pandas dataframe

# Matplotlib and associated plotting modules
import matplotlib.cm as cm
import matplotlib.colors as colors

# import k-means from clustering stage
from sklearn.cluster import KMeans

#!pip install folium
import folium # map rendering library
```


```python
# Get only Boroughs with the word Toronto

df = df[df["Borough"].str.contains("Toronto")].reset_index()
```


```python
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
      <th>index</th>
      <th>Postal Code</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>37</td>
      <td>M4E</td>
      <td>East Toronto</td>
      <td>The Beaches</td>
      <td>43.67709</td>
      <td>-79.29547</td>
    </tr>
    <tr>
      <th>1</th>
      <td>41</td>
      <td>M4K</td>
      <td>East Toronto</td>
      <td>The Danforth West, Riverdale</td>
      <td>43.68375</td>
      <td>-79.35512</td>
    </tr>
    <tr>
      <th>2</th>
      <td>42</td>
      <td>M4L</td>
      <td>East Toronto</td>
      <td>India Bazaar, The Beaches West</td>
      <td>43.66797</td>
      <td>-79.31467</td>
    </tr>
    <tr>
      <th>3</th>
      <td>43</td>
      <td>M4M</td>
      <td>East Toronto</td>
      <td>Studio District</td>
      <td>43.66213</td>
      <td>-79.33497</td>
    </tr>
    <tr>
      <th>4</th>
      <td>44</td>
      <td>M4N</td>
      <td>Central Toronto</td>
      <td>Lawrence Park</td>
      <td>43.72843</td>
      <td>-79.38713</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Visualize points in map

address = 'Toronto'

geolocator = Nominatim(user_agent="TO_explorer")
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print('The geograpical coordinate of Toronto are {}, {}.'.format(latitude, longitude))

# create map of New York using latitude and longitude values
map_toronto = folium.Map(location=[latitude, longitude], zoom_start=12)

# add markers to map
for lat, lng, borough, neighborhood in zip(df['Latitude'], df['Longitude'], df['Borough'], df['Neighbourhood']):
    label = '{}, {}'.format(neighborhood, borough)
    label = folium.Popup(label, parse_html=True)
    folium.CircleMarker(
        [lat, lng],
        radius=5,
        popup=label,
        color='blue',
        fill=True,
        fill_color='#3186cc',
        fill_opacity=0.7,
        parse_html=False).add_to(map_toronto)  
    
map_toronto
```

    The geograpical coordinate of Toronto are 43.6534817, -79.3839347.





<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5ID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5IiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFs0My42NTM0ODE3LCAtNzkuMzgzOTM0N10sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiAxMiwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyX2Q0MTcyNTI1NDQxMzQyMDg5N2I2ZDMwYjhkZTQ3ZTM3ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfY2YzZTkzYzA4NjRhNDBjZDkyYzA3ZWJlYjZiMGU2OGEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NzcwOTAwMDAwMDAwOCwgLTc5LjI5NTQ2OTk5OTk5OTk3XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzY2ZDljYjIwYzA5MjQ0ODA5NTViNDMzYjUzZTY2ZDRkID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF81MGVjOWRhZDQ1ZWI0MWRjOWEzN2Y1MDJjYTg4YzhhYiA9ICQoYDxkaXYgaWQ9Imh0bWxfNTBlYzlkYWQ0NWViNDFkYzlhMzdmNTAyY2E4OGM4YWIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRoZSBCZWFjaGVzLCBFYXN0IFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNjZkOWNiMjBjMDkyNDQ4MDk1NWI0MzNiNTNlNjZkNGQuc2V0Q29udGVudChodG1sXzUwZWM5ZGFkNDVlYjQxZGM5YTM3ZjUwMmNhODhjOGFiKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9jZjNlOTNjMDg2NGE0MGNkOTJjMDdlYmViNmIwZTY4YS5iaW5kUG9wdXAocG9wdXBfNjZkOWNiMjBjMDkyNDQ4MDk1NWI0MzNiNTNlNjZkNGQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzFkZWRkYTJjNDY2MzQxN2JiMDMyZDE0MDgwYjQwNGI5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjgzNzUwMDAwMDAwMDMsIC03OS4zNTUxMTk5OTk5OTk5NF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9lNjMzMjI3OTE1ZmY0NTkwOTU3NGExNGQ0YTFmOGFhZCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNTU0MTc3N2FkNTFhNGY0ZmEwZDc4YjMxNjFjY2I5YTQgPSAkKGA8ZGl2IGlkPSJodG1sXzU1NDE3NzdhZDUxYTRmNGZhMGQ3OGIzMTYxY2NiOWE0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5UaGUgRGFuZm9ydGggV2VzdCwgUml2ZXJkYWxlLCBFYXN0IFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZTYzMzIyNzkxNWZmNDU5MDk1NzRhMTRkNGExZjhhYWQuc2V0Q29udGVudChodG1sXzU1NDE3NzdhZDUxYTRmNGZhMGQ3OGIzMTYxY2NiOWE0KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8xZGVkZGEyYzQ2NjM0MTdiYjAzMmQxNDA4MGI0MDRiOS5iaW5kUG9wdXAocG9wdXBfZTYzMzIyNzkxNWZmNDU5MDk1NzRhMTRkNGExZjhhYWQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2U1MDBjZjRlOTEyMTQ4ZDg4MDYzMGMyNTI2NWFlZTU4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjY3OTcwMDAwMDAwMDI1LCAtNzkuMzE0NjY5OTk5OTk5OThdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfOTkwODgwM2U5NmJhNGIyOThjZTQ4OWMzNDRhOTg4ZDAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2M2NjJiNjcyNzc3ZTQ1YjE4N2M2OTg0M2JjNjAzNWFiID0gJChgPGRpdiBpZD0iaHRtbF9jNjYyYjY3Mjc3N2U0NWIxODdjNjk4NDNiYzYwMzVhYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SW5kaWEgQmF6YWFyLCBUaGUgQmVhY2hlcyBXZXN0LCBFYXN0IFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOTkwODgwM2U5NmJhNGIyOThjZTQ4OWMzNDRhOTg4ZDAuc2V0Q29udGVudChodG1sX2M2NjJiNjcyNzc3ZTQ1YjE4N2M2OTg0M2JjNjAzNWFiKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9lNTAwY2Y0ZTkxMjE0OGQ4ODA2MzBjMjUyNjVhZWU1OC5iaW5kUG9wdXAocG9wdXBfOTkwODgwM2U5NmJhNGIyOThjZTQ4OWMzNDRhOTg4ZDApCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzMzODUyNzEyODFkMjRmMzY5NTUyZTI4YjNmODc2NzU5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjYyMTMwMDAwMDAwMDUsIC03OS4zMzQ5Njk5OTk5OTk5NF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8wNWFhNmYyZDU5NGY0MjI0Yjg5NzIxZWVjOWU5NWMyYyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNGJhYWFkNThkYTQwNDVhNmI4OTQ4OGJkMDA2MWU2NDIgPSAkKGA8ZGl2IGlkPSJodG1sXzRiYWFhZDU4ZGE0MDQ1YTZiODk0ODhiZDAwNjFlNjQyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdHVkaW8gRGlzdHJpY3QsIEVhc3QgVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wNWFhNmYyZDU5NGY0MjI0Yjg5NzIxZWVjOWU5NWMyYy5zZXRDb250ZW50KGh0bWxfNGJhYWFkNThkYTQwNDVhNmI4OTQ4OGJkMDA2MWU2NDIpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzMzODUyNzEyODFkMjRmMzY5NTUyZTI4YjNmODc2NzU5LmJpbmRQb3B1cChwb3B1cF8wNWFhNmYyZDU5NGY0MjI0Yjg5NzIxZWVjOWU5NWMyYykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDdjOWE5YmJmOGU2NDVkNGFhODNkMTgwZWZlYTNlOWUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43Mjg0MzAwMDAwMDAwNiwgLTc5LjM4NzEyOTk5OTk5OTk2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzhmMjQ4OGQ2MjZiOTRkYThiZjk3YWNlYmY0NWNmNzQ0ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9iZDkzZGIzZjQ5NWQ0MWY4ODVkMzE2NDNkZWYwZTYzOSA9ICQoYDxkaXYgaWQ9Imh0bWxfYmQ5M2RiM2Y0OTVkNDFmODg1ZDMxNjQzZGVmMGU2MzkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkxhd3JlbmNlIFBhcmssIENlbnRyYWwgVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF84ZjI0ODhkNjI2Yjk0ZGE4YmY5N2FjZWJmNDVjZjc0NC5zZXRDb250ZW50KGh0bWxfYmQ5M2RiM2Y0OTVkNDFmODg1ZDMxNjQzZGVmMGU2MzkpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzQ3YzlhOWJiZjhlNjQ1ZDRhYTgzZDE4MGVmZWEzZTllLmJpbmRQb3B1cChwb3B1cF84ZjI0ODhkNjI2Yjk0ZGE4YmY5N2FjZWJmNDVjZjc0NCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYWU1OWE1ODdjODYwNGFiZTg0ZTRkY2M3YTZkNmIzYTQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTI3NjAwMDAwMDAwNiwgLTc5LjM4ODUwOTk5OTk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzFhZGFlMzIzYTc5ZjQxZjk5Y2I5MWVmYzBiMWVhN2ZiID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF83NmNiMmE3Njk5MTI0YjllOTM3YzZlZDQ2ODcxYTBhOCA9ICQoYDxkaXYgaWQ9Imh0bWxfNzZjYjJhNzY5OTEyNGI5ZTkzN2M2ZWQ0Njg3MWEwYTgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkRhdmlzdmlsbGUgTm9ydGgsIENlbnRyYWwgVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8xYWRhZTMyM2E3OWY0MWY5OWNiOTFlZmMwYjFlYTdmYi5zZXRDb250ZW50KGh0bWxfNzZjYjJhNzY5OTEyNGI5ZTkzN2M2ZWQ0Njg3MWEwYTgpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2FlNTlhNTg3Yzg2MDRhYmU4NGU0ZGNjN2E2ZDZiM2E0LmJpbmRQb3B1cChwb3B1cF8xYWRhZTMyM2E3OWY0MWY5OWNiOTFlZmMwYjFlYTdmYikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYWVmYjVmZjY2OTM0NDgyZmIzMmFiOGQxY2VjNDI0OGYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTQ1ODAwMDAwMDAwNywgLTc5LjQwNjY3OTk5OTk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzM4NjVkYTc4ZTI4NTQxODM5NDZkZjZiZWJkMjEyZGI1ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8wODIyODRiOGU4YWM0NDAwYmQwMTI5MjE4NTZhMGRiMSA9ICQoYDxkaXYgaWQ9Imh0bWxfMDgyMjg0YjhlOGFjNDQwMGJkMDEyOTIxODU2YTBkYjEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5vcnRoIFRvcm9udG8gV2VzdCwgTGF3cmVuY2UgUGFyaywgQ2VudHJhbCBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzM4NjVkYTc4ZTI4NTQxODM5NDZkZjZiZWJkMjEyZGI1LnNldENvbnRlbnQoaHRtbF8wODIyODRiOGU4YWM0NDAwYmQwMTI5MjE4NTZhMGRiMSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfYWVmYjVmZjY2OTM0NDgyZmIzMmFiOGQxY2VjNDI0OGYuYmluZFBvcHVwKHBvcHVwXzM4NjVkYTc4ZTI4NTQxODM5NDZkZjZiZWJkMjEyZGI1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80MzBhZWRmMzAwYTk0MTE5ODAxYzk5MGM3NDAzZTU1YiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjcwMzQwMDAwMDAwMDA0NSwgLTc5LjM4NjU4OTk5OTk5OTk2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzJjZWMxMWM5Y2MyMzQwMDJhZGMyYzZlZjdhZjAxNjBiID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9kMTdhYTdmMTc1ZDA0NmYwOGJhODI0OWE2YjhkZDlhNiA9ICQoYDxkaXYgaWQ9Imh0bWxfZDE3YWE3ZjE3NWQwNDZmMDhiYTgyNDlhNmI4ZGQ5YTYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkRhdmlzdmlsbGUsIENlbnRyYWwgVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8yY2VjMTFjOWNjMjM0MDAyYWRjMmM2ZWY3YWYwMTYwYi5zZXRDb250ZW50KGh0bWxfZDE3YWE3ZjE3NWQwNDZmMDhiYTgyNDlhNmI4ZGQ5YTYpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzQzMGFlZGYzMDBhOTQxMTk4MDFjOTkwYzc0MDNlNTViLmJpbmRQb3B1cChwb3B1cF8yY2VjMTFjOWNjMjM0MDAyYWRjMmM2ZWY3YWYwMTYwYikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZTRkMGYwNDVhYjBiNDNlNzg2NGM1NzM2YmYwZTlmZDQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42OTA0ODAwMDAwMDAwMzYsIC03OS4zODMxNzk5OTk5OTk5OF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81MTZjYzE1NmNjY2U0YmJlYmZlM2JhMTMyN2MwMzg2NSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfODM0MWFhYWM1ZjZhNDU5ZDhjMTc3Nzc2OTAzNzI4MjggPSAkKGA8ZGl2IGlkPSJodG1sXzgzNDFhYWFjNWY2YTQ1OWQ4YzE3Nzc3NjkwMzcyODI4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Nb29yZSBQYXJrLCBTdW1tZXJoaWxsIEVhc3QsIENlbnRyYWwgVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF81MTZjYzE1NmNjY2U0YmJlYmZlM2JhMTMyN2MwMzg2NS5zZXRDb250ZW50KGh0bWxfODM0MWFhYWM1ZjZhNDU5ZDhjMTc3Nzc2OTAzNzI4MjgpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2U0ZDBmMDQ1YWIwYjQzZTc4NjRjNTczNmJmMGU5ZmQ0LmJpbmRQb3B1cChwb3B1cF81MTZjYzE1NmNjY2U0YmJlYmZlM2JhMTMyN2MwMzg2NSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNTNmOTg0N2Q0NzkzNGVkNWE4NjFkZGQ4OWE5NjM2ZWYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42ODU2ODAwMDAwMDAwNSwgLTc5LjQwMjM2OTk5OTk5OTk2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzIxMjkxOGUzNjc3NDQ3MWZiZmNkODQ2Y2M0MjBkMDZiID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF85NmJmM2JkNTQxNjI0MWNmOWNjMTI0NWNjY2Y2NjZjMiA9ICQoYDxkaXYgaWQ9Imh0bWxfOTZiZjNiZDU0MTYyNDFjZjljYzEyNDVjY2NmNjY2YzIiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN1bW1lcmhpbGwgV2VzdCwgUmF0aG5lbGx5LCBTb3V0aCBIaWxsLCBGb3Jlc3QgSGlsbCBTRSwgRGVlciBQYXJrLCBDZW50cmFsIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMjEyOTE4ZTM2Nzc0NDcxZmJmY2Q4NDZjYzQyMGQwNmIuc2V0Q29udGVudChodG1sXzk2YmYzYmQ1NDE2MjQxY2Y5Y2MxMjQ1Y2NjZjY2NmMyKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl81M2Y5ODQ3ZDQ3OTM0ZWQ1YTg2MWRkZDg5YTk2MzZlZi5iaW5kUG9wdXAocG9wdXBfMjEyOTE4ZTM2Nzc0NDcxZmJmY2Q4NDZjYzQyMGQwNmIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzQwNThlMzEyYjc1NzQ5MGM5NDhjNDk4OGE0NjZlOGE0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjgxOTAwMDAwMDAwMDQsIC03OS4zNzgyODk5OTk5OTk5NF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF82NGIzZTcxNDFkNzQ0OWRkYmFiNzI0YWYxMzA3ZmVhMiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYzAzNzFkYzZlODVjNGVmMmE1OTMxNDdkZDZmM2Q1MWUgPSAkKGA8ZGl2IGlkPSJodG1sX2MwMzcxZGM2ZTg1YzRlZjJhNTkzMTQ3ZGQ2ZjNkNTFlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Sb3NlZGFsZSwgRG93bnRvd24gVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF82NGIzZTcxNDFkNzQ0OWRkYmFiNzI0YWYxMzA3ZmVhMi5zZXRDb250ZW50KGh0bWxfYzAzNzFkYzZlODVjNGVmMmE1OTMxNDdkZDZmM2Q1MWUpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzQwNThlMzEyYjc1NzQ5MGM5NDhjNDk4OGE0NjZlOGE0LmJpbmRQb3B1cChwb3B1cF82NGIzZTcxNDFkNzQ0OWRkYmFiNzI0YWYxMzA3ZmVhMikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZTRmYmFmNGM4ZjUzNGZjNWIxYzUyNjI4Nzg5YmQ4ZDUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42Njc4ODAwMDAwMDAwMjUsIC03OS4zNjY0ODk5OTk5OTk5NF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9kNGFiZGQ0NzlkZGE0OTNhYjRlMTBiNTJjMGU3YmZhNSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfY2I5ZTFjMDA0ZTA5NDQxNTk1YWQyNzZjNmExMWQzYTIgPSAkKGA8ZGl2IGlkPSJodG1sX2NiOWUxYzAwNGUwOTQ0MTU5NWFkMjc2YzZhMTFkM2EyIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdC4gSmFtZXMgVG93biwgQ2FiYmFnZXRvd24sIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZDRhYmRkNDc5ZGRhNDkzYWI0ZTEwYjUyYzBlN2JmYTUuc2V0Q29udGVudChodG1sX2NiOWUxYzAwNGUwOTQ0MTU5NWFkMjc2YzZhMTFkM2EyKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9lNGZiYWY0YzhmNTM0ZmM1YjFjNTI2Mjg3ODliZDhkNS5iaW5kUG9wdXAocG9wdXBfZDRhYmRkNDc5ZGRhNDkzYWI0ZTEwYjUyYzBlN2JmYTUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2ZjNWI5NDg2M2E5MDQ0ZjdiZWE3MTcwNDVhZGU4NzhlID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjY2NTkwMDAwMDAwMDQsIC03OS4zODEzMjk5OTk5OTk5M10sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9lMmY3Yjc4MWYzYWU0ZWQ5YWQ0ZDcxZDM2NWNjZjA3YSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjQ2OGQyNjg1MTYyNDI5MmE0YzdkOGJmZjE1MDZmM2MgPSAkKGA8ZGl2IGlkPSJodG1sX2I0NjhkMjY4NTE2MjQyOTJhNGM3ZDhiZmYxNTA2ZjNjIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaHVyY2ggYW5kIFdlbGxlc2xleSwgRG93bnRvd24gVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9lMmY3Yjc4MWYzYWU0ZWQ5YWQ0ZDcxZDM2NWNjZjA3YS5zZXRDb250ZW50KGh0bWxfYjQ2OGQyNjg1MTYyNDI5MmE0YzdkOGJmZjE1MDZmM2MpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2ZjNWI5NDg2M2E5MDQ0ZjdiZWE3MTcwNDVhZGU4NzhlLmJpbmRQb3B1cChwb3B1cF9lMmY3Yjc4MWYzYWU0ZWQ5YWQ0ZDcxZDM2NWNjZjA3YSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZWRiNzE1NzBlMjIwNGU5ZGE2MWIwMGVjOTc4ODEzMjggPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTUxMjAwMDAwMDAwNywgLTc5LjM2MjYzOTk5OTk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2YxNjk2MTlhN2JmYzQzY2I5NzQxYTg3YjgzMWUwMWRjID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF80YWZhNWRmNDc5ZTM0YTA3OWEyMDdlN2M5NTVhZTMyMCA9ICQoYDxkaXYgaWQ9Imh0bWxfNGFmYTVkZjQ3OWUzNGEwNzlhMjA3ZTdjOTU1YWUzMjAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlJlZ2VudCBQYXJrLCBIYXJib3VyZnJvbnQsIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZjE2OTYxOWE3YmZjNDNjYjk3NDFhODdiODMxZTAxZGMuc2V0Q29udGVudChodG1sXzRhZmE1ZGY0NzllMzRhMDc5YTIwN2U3Yzk1NWFlMzIwKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9lZGI3MTU3MGUyMjA0ZTlkYTYxYjAwZWM5Nzg4MTMyOC5iaW5kUG9wdXAocG9wdXBfZjE2OTYxOWE3YmZjNDNjYjk3NDFhODdiODMxZTAxZGMpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzMyNzAyMGNhNmM4NTQxN2JiMmFhOGY1OGViZDAzN2YxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjU3MzkwMDAwMDAwMDgsIC03OS4zNzgwMzk5OTk5OTk5NF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81NWQ2YzFjNGI1OWQ0ZWFlODJkNzA1M2UwM2FmYzAyYiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOTllZGY0NDk3M2NkNGIwMTg0MGFkM2ZkZGZmMjIyOTkgPSAkKGA8ZGl2IGlkPSJodG1sXzk5ZWRmNDQ5NzNjZDRiMDE4NDBhZDNmZGRmZjIyMjk5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5HYXJkZW4gRGlzdHJpY3QsIFJ5ZXJzb24sIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNTVkNmMxYzRiNTlkNGVhZTgyZDcwNTNlMDNhZmMwMmIuc2V0Q29udGVudChodG1sXzk5ZWRmNDQ5NzNjZDRiMDE4NDBhZDNmZGRmZjIyMjk5KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8zMjcwMjBjYTZjODU0MTdiYjJhYThmNThlYmQwMzdmMS5iaW5kUG9wdXAocG9wdXBfNTVkNmMxYzRiNTlkNGVhZTgyZDcwNTNlMDNhZmMwMmIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2EwYzM2M2RhNjQ5NjQyMzFiZjBkZDUyNTc1OTk1ZmIyID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjUyMTUwMDAwMDAwMDYsIC03OS4zNzU4Njk5OTk5OTk5Nl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9lYWUxMzllNzkwMDQ0YjE1YWI3N2U0YWY4ZTM1NTE3ZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfN2FlMmI0YjY4Njg5NDhmNWEzNDdmMDgwZmMxZmMyZTQgPSAkKGA8ZGl2IGlkPSJodG1sXzdhZTJiNGI2ODY4OTQ4ZjVhMzQ3ZjA4MGZjMWZjMmU0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdC4gSmFtZXMgVG93biwgRG93bnRvd24gVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9lYWUxMzllNzkwMDQ0YjE1YWI3N2U0YWY4ZTM1NTE3ZS5zZXRDb250ZW50KGh0bWxfN2FlMmI0YjY4Njg5NDhmNWEzNDdmMDgwZmMxZmMyZTQpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2EwYzM2M2RhNjQ5NjQyMzFiZjBkZDUyNTc1OTk1ZmIyLmJpbmRQb3B1cChwb3B1cF9lYWUxMzllNzkwMDQ0YjE1YWI3N2U0YWY4ZTM1NTE3ZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTQ4ODMxN2Q1Y2Q4NDdmYTlmYmQ0ZmU5ZmI2NjQ2NTAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDUzNjAwMDAwMDAwNCwgLTc5LjM3MzA1OTk5OTk5OTk1XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzBiM2Y1MTczOTYyMTQyOTZhNzdmZWJhZmRjODQyNjBmID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF82MzljYmQwMDgzNjI0NDIzOTkwNzMxNjhlZDQ5ZjM0OCA9ICQoYDxkaXYgaWQ9Imh0bWxfNjM5Y2JkMDA4MzYyNDQyMzk5MDczMTY4ZWQ0OWYzNDgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJlcmN6eSBQYXJrLCBEb3dudG93biBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzBiM2Y1MTczOTYyMTQyOTZhNzdmZWJhZmRjODQyNjBmLnNldENvbnRlbnQoaHRtbF82MzljYmQwMDgzNjI0NDIzOTkwNzMxNjhlZDQ5ZjM0OCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfMTQ4ODMxN2Q1Y2Q4NDdmYTlmYmQ0ZmU5ZmI2NjQ2NTAuYmluZFBvcHVwKHBvcHVwXzBiM2Y1MTczOTYyMTQyOTZhNzdmZWJhZmRjODQyNjBmKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83MzhhMzZkODY4MjA0MWVjYjZhMjc3NjNjYTdmYzE5ZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1NjA5MDAwMDAwMDA2LCAtNzkuMzg0OTI5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMzM1ZjczYjNhOTFjNGE1MjgwYWU2NTJiNGVmMWM3YmEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzk3MDNmYjIyNzYyMjQ5ZjY5NzkzYWQ2MjgyOWZhMzM3ID0gJChgPGRpdiBpZD0iaHRtbF85NzAzZmIyMjc2MjI0OWY2OTc5M2FkNjI4MjlmYTMzNyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2VudHJhbCBCYXkgU3RyZWV0LCBEb3dudG93biBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzMzNWY3M2IzYTkxYzRhNTI4MGFlNjUyYjRlZjFjN2JhLnNldENvbnRlbnQoaHRtbF85NzAzZmIyMjc2MjI0OWY2OTc5M2FkNjI4MjlmYTMzNyk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNzM4YTM2ZDg2ODIwNDFlY2I2YTI3NzYzY2E3ZmMxOWUuYmluZFBvcHVwKHBvcHVwXzMzNWY3M2IzYTkxYzRhNTI4MGFlNjUyYjRlZjFjN2JhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85ZTk4M2ZlMDM1Zjk0NjljYThlZGRhMjI5YWZiY2NkYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0OTcwMDAwMDAwMDA1LCAtNzkuMzgyNTc5OTk5OTk5OTZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZDFmOTQ3NGY5ODIyNGY3Y2E2YmY2YTBiZWVkZTdmZjcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzg1ODdjYjhlMGE5YzQ2ZTA5MGM0YWJiYjFmZDJkNjFlID0gJChgPGRpdiBpZD0iaHRtbF84NTg3Y2I4ZTBhOWM0NmUwOTBjNGFiYmIxZmQyZDYxZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UmljaG1vbmQsIEFkZWxhaWRlLCBLaW5nLCBEb3dudG93biBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2QxZjk0NzRmOTgyMjRmN2NhNmJmNmEwYmVlZGU3ZmY3LnNldENvbnRlbnQoaHRtbF84NTg3Y2I4ZTBhOWM0NmUwOTBjNGFiYmIxZmQyZDYxZSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfOWU5ODNmZTAzNWY5NDY5Y2E4ZWRkYTIyOWFmYmNjZGEuYmluZFBvcHVwKHBvcHVwX2QxZjk0NzRmOTgyMjRmN2NhNmJmNmEwYmVlZGU3ZmY3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8xZWFlOWFmMTdiM2Y0MmVmYjNhOTZmNTk4NDRhZDdkYiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0Mjg1MDAwMDAwMDA3LCAtNzkuMzgwNzU5OTk5OTk5OTVdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNjVlNDE4OTBmMWJiNGRiZDgxOTA4YjY1ODljZmEzNWQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzNkOTM5YTBkYTYyYjRmNTQ5Y2RlOGQxMWExNWM5OGQ0ID0gJChgPGRpdiBpZD0iaHRtbF8zZDkzOWEwZGE2MmI0ZjU0OWNkZThkMTFhMTVjOThkNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+SGFyYm91cmZyb250IEVhc3QsIFVuaW9uIFN0YXRpb24sIFRvcm9udG8gSXNsYW5kcywgRG93bnRvd24gVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF82NWU0MTg5MGYxYmI0ZGJkODE5MDhiNjU4OWNmYTM1ZC5zZXRDb250ZW50KGh0bWxfM2Q5MzlhMGRhNjJiNGY1NDljZGU4ZDExYTE1Yzk4ZDQpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzFlYWU5YWYxN2IzZjQyZWZiM2E5NmY1OTg0NGFkN2RiLmJpbmRQb3B1cChwb3B1cF82NWU0MTg5MGYxYmI0ZGJkODE5MDhiNjU4OWNmYTM1ZCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTJmMjFjZmY2Mzk5NDMwMmJlNjJlZTBjYzdmY2RhNGIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDcxMDAwMDAwMDAwOCwgLTc5LjM4MTUyOTk5OTk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzE3MzdiMGM5YTdjMzRiNWY4MjVkYTBkM2I3NzdjMDM1ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9hNjZmNTY1ZjQyYjg0N2U4Yjg5MDJiZTgzOTg2NjJiNSA9ICQoYDxkaXYgaWQ9Imh0bWxfYTY2ZjU2NWY0MmI4NDdlOGI4OTAyYmU4Mzk4NjYyYjUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRvcm9udG8gRG9taW5pb24gQ2VudHJlLCBEZXNpZ24gRXhjaGFuZ2UsIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMTczN2IwYzlhN2MzNGI1ZjgyNWRhMGQzYjc3N2MwMzUuc2V0Q29udGVudChodG1sX2E2NmY1NjVmNDJiODQ3ZThiODkwMmJlODM5ODY2MmI1KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl85MmYyMWNmZjYzOTk0MzAyYmU2MmVlMGNjN2ZjZGE0Yi5iaW5kUG9wdXAocG9wdXBfMTczN2IwYzlhN2MzNGI1ZjgyNWRhMGQzYjc3N2MwMzUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2RjN2JhMGU1MzVlMjQ0Njg4MTEyMDg2YzI5ZDE3Y2ZkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4NDAwMDAwMDAwMDQsIC03OS4zNzkxMzk5OTk5OTk5NV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF83MjBmZmU2MzZkMzM0NjM1OTFlZjJiOGRhNDMyNWRiMyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMjBkYjM1YTEyMjk4NGY3MDhmMjUxZjE3NDUwODE0ODggPSAkKGA8ZGl2IGlkPSJodG1sXzIwZGIzNWExMjI5ODRmNzA4ZjI1MWYxNzQ1MDgxNDg4IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Db21tZXJjZSBDb3VydCwgVmljdG9yaWEgSG90ZWwsIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNzIwZmZlNjM2ZDMzNDYzNTkxZWYyYjhkYTQzMjVkYjMuc2V0Q29udGVudChodG1sXzIwZGIzNWExMjI5ODRmNzA4ZjI1MWYxNzQ1MDgxNDg4KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9kYzdiYTBlNTM1ZTI0NDY4ODExMjA4NmMyOWQxN2NmZC5iaW5kUG9wdXAocG9wdXBfNzIwZmZlNjM2ZDMzNDYzNTkxZWYyYjhkYTQzMjVkYjMpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzcxNDQ1NTQ5NTcwNzQ5OGY5NWQ5ZmY3ZTkzM2VmYTNmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzEyMDgwMDAwMDAwMDcsIC03OS40MTg0Nzk5OTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9hMjU4YjdmOWFjMjk0ZjVjOTg0ZTczYWRjZmY4MTEzYyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOTgyMTk5MzUxNjRiNDlhNmEzNzg5MjEzOTNlZmUxMDUgPSAkKGA8ZGl2IGlkPSJodG1sXzk4MjE5OTM1MTY0YjQ5YTZhMzc4OTIxMzkzZWZlMTA1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Sb3NlbGF3biwgQ2VudHJhbCBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2EyNThiN2Y5YWMyOTRmNWM5ODRlNzNhZGNmZjgxMTNjLnNldENvbnRlbnQoaHRtbF85ODIxOTkzNTE2NGI0OWE2YTM3ODkyMTM5M2VmZTEwNSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNzE0NDU1NDk1NzA3NDk4Zjk1ZDlmZjdlOTMzZWZhM2YuYmluZFBvcHVwKHBvcHVwX2EyNThiN2Y5YWMyOTRmNWM5ODRlNzNhZGNmZjgxMTNjKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kYTk4ZTYyNGI4NTI0ODRjYTE0YzBhMGQwNzYwNjg1YiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY5NDc5MDAwMDAwMDA3LCAtNzkuNDE0Mzk5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZjQ3NGM5NzQ5Y2FlNGJiODlmZGFkYWZjMjFlMWY5MzAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzBlOWE2OTBiNzJlMzRlNzFiODRhYzllNmI3MWYxN2ViID0gJChgPGRpdiBpZD0iaHRtbF8wZTlhNjkwYjcyZTM0ZTcxYjg0YWM5ZTZiNzFmMTdlYiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Rm9yZXN0IEhpbGwgTm9ydGggJmFtcDsgV2VzdCwgRm9yZXN0IEhpbGwgUm9hZCBQYXJrLCBDZW50cmFsIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZjQ3NGM5NzQ5Y2FlNGJiODlmZGFkYWZjMjFlMWY5MzAuc2V0Q29udGVudChodG1sXzBlOWE2OTBiNzJlMzRlNzFiODRhYzllNmI3MWYxN2ViKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9kYTk4ZTYyNGI4NTI0ODRjYTE0YzBhMGQwNzYwNjg1Yi5iaW5kUG9wdXAocG9wdXBfZjQ3NGM5NzQ5Y2FlNGJiODlmZGFkYWZjMjFlMWY5MzApCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzk5MzM4YjEzMDA2NzQ4YmJiYzgxYWI5MDRmMDRiMjM2ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjc0ODQwMDAwMDAwMDc0LCAtNzkuNDA0NTE5OTk5OTk5OTNdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNDNhMzU2NWE0ZmEwNDlhMWJjMjAxOTZiMjY3NDM2YmQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzY1NTk0MzEyNGNhNzQ4M2FhYWI4MjQyZGU1MDI2Yzk0ID0gJChgPGRpdiBpZD0iaHRtbF82NTU5NDMxMjRjYTc0ODNhYWFiODI0MmRlNTAyNmM5NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGhlIEFubmV4LCBOb3J0aCBNaWR0b3duLCBZb3JrdmlsbGUsIENlbnRyYWwgVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80M2EzNTY1YTRmYTA0OWExYmMyMDE5NmIyNjc0MzZiZC5zZXRDb250ZW50KGh0bWxfNjU1OTQzMTI0Y2E3NDgzYWFhYjgyNDJkZTUwMjZjOTQpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzk5MzM4YjEzMDA2NzQ4YmJiYzgxYWI5MDRmMDRiMjM2LmJpbmRQb3B1cChwb3B1cF80M2EzNTY1YTRmYTA0OWExYmMyMDE5NmIyNjc0MzZiZCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZmRkYjNkZTZiZDc1NDRjOWEzMzM4MGRlMDY3YjJhMWYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjMxMTAwMDAwMDAwNzQsIC03OS40MDE3OTk5OTk5OTk5OF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF84YWNhZTRhMGYzNzg0ZTBkOTdlN2I4ZGE1OWNmZmJlMyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZTdkM2U2NjcyNDAyNGVjMmI2ZmUyODZlMWY1MjI0NTEgPSAkKGA8ZGl2IGlkPSJodG1sX2U3ZDNlNjY3MjQwMjRlYzJiNmZlMjg2ZTFmNTIyNDUxIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Vbml2ZXJzaXR5IG9mIFRvcm9udG8sIEhhcmJvcmQsIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfOGFjYWU0YTBmMzc4NGUwZDk3ZTdiOGRhNTljZmZiZTMuc2V0Q29udGVudChodG1sX2U3ZDNlNjY3MjQwMjRlYzJiNmZlMjg2ZTFmNTIyNDUxKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9mZGRiM2RlNmJkNzU0NGM5YTMzMzgwZGUwNjdiMmExZi5iaW5kUG9wdXAocG9wdXBfOGFjYWU0YTBmMzc4NGUwZDk3ZTdiOGRhNTljZmZiZTMpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzZjMWUxMDQzZGNmYTQyZTJhZmU0OWRjYjFkYmQzNWM5ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjUzNTEwMDAwMDAwMDQsIC03OS4zOTcyMTk5OTk5OTk5NV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF81MDQ1ZjIyYmVhOGU0NGUxYWJkYzdkMWJjNGFhM2Y2YSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfZGYxNTA3MTc2MDgxNDAyZDg1ZjMxYjMxN2Y2YzJkZWUgPSAkKGA8ZGl2IGlkPSJodG1sX2RmMTUwNzE3NjA4MTQwMmQ4NWYzMWIzMTdmNmMyZGVlIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5LZW5zaW5ndG9uIE1hcmtldCwgQ2hpbmF0b3duLCBHcmFuZ2UgUGFyaywgRG93bnRvd24gVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF81MDQ1ZjIyYmVhOGU0NGUxYWJkYzdkMWJjNGFhM2Y2YS5zZXRDb250ZW50KGh0bWxfZGYxNTA3MTc2MDgxNDAyZDg1ZjMxYjMxN2Y2YzJkZWUpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzZjMWUxMDQzZGNmYTQyZTJhZmU0OWRjYjFkYmQzNWM5LmJpbmRQb3B1cChwb3B1cF81MDQ1ZjIyYmVhOGU0NGUxYWJkYzdkMWJjNGFhM2Y2YSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTM2YTI3Mzg2ZGU4NDU2ZGFkMjQyMzAyNzA0NTgzMjUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDA4MjAwMDAwMDAwNzYsIC03OS4zOTgxNzk5OTk5OTk5N10sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jZDc3Y2RhYjhkNWU0ZTM5ODRhZjA0MGFmM2E3Y2M0ZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMDNmYzg3MTg4ZjBiNDQzYzgxMmMzMGMwNTY2NjEyYTYgPSAkKGA8ZGl2IGlkPSJodG1sXzAzZmM4NzE4OGYwYjQ0M2M4MTJjMzBjMDU2NjYxMmE2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DTiBUb3dlciwgS2luZyBhbmQgU3BhZGluYSwgUmFpbHdheSBMYW5kcywgSGFyYm91cmZyb250IFdlc3QsIEJhdGh1cnN0IFF1YXksIFNvdXRoIE5pYWdhcmEsIElzbGFuZCBhaXJwb3J0LCBEb3dudG93biBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2NkNzdjZGFiOGQ1ZTRlMzk4NGFmMDQwYWYzYTdjYzRlLnNldENvbnRlbnQoaHRtbF8wM2ZjODcxODhmMGI0NDNjODEyYzMwYzA1NjY2MTJhNik7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfOTM2YTI3Mzg2ZGU4NDU2ZGFkMjQyMzAyNzA0NTgzMjUuYmluZFBvcHVwKHBvcHVwX2NkNzdjZGFiOGQ1ZTRlMzk4NGFmMDQwYWYzYTdjYzRlKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mNzE0NTFiODA0YmM0OWQ2OTliOGMwY2NkMGJhNjc0NSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0ODY5MDAwMDAwMDA0NSwgLTc5LjM4NTQzOTk5OTk5OTk2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzg4MmQxZmQwZDNiYjRkOTA4MjY1NGQxMmQ3N2ExNjAwID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF83ZTZkZDkyNWU3NDU0MjdjOTQwOTcyMmI1NDg3YmU5MyA9ICQoYDxkaXYgaWQ9Imh0bWxfN2U2ZGQ5MjVlNzQ1NDI3Yzk0MDk3MjJiNTQ4N2JlOTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN0biBBIFBPIEJveGVzLCBEb3dudG93biBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzg4MmQxZmQwZDNiYjRkOTA4MjY1NGQxMmQ3N2ExNjAwLnNldENvbnRlbnQoaHRtbF83ZTZkZDkyNWU3NDU0MjdjOTQwOTcyMmI1NDg3YmU5Myk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfZjcxNDUxYjgwNGJjNDlkNjk5YjhjMGNjZDBiYTY3NDUuYmluZFBvcHVwKHBvcHVwXzg4MmQxZmQwZDNiYjRkOTA4MjY1NGQxMmQ3N2ExNjAwKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zN2YwNjRlMWIzNGI0MTQ4YWRjZTZiMjY2ZGYzNDVhOCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0ODI4MDAwMDAwMDA2LCAtNzkuMzgxNDU5OTk5OTk5OTVdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYWMxY2JkMjUxN2U4NDdjZDgyOGM0YjQ4ZmQ1NThiMjkgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2RkZTg3ZTBmOGJkZTRlNzU5YjM5MDc1YjBmOGVmOTRiID0gJChgPGRpdiBpZD0iaHRtbF9kZGU4N2UwZjhiZGU0ZTc1OWIzOTA3NWIwZjhlZjk0YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Rmlyc3QgQ2FuYWRpYW4gUGxhY2UsIFVuZGVyZ3JvdW5kIGNpdHksIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYWMxY2JkMjUxN2U4NDdjZDgyOGM0YjQ4ZmQ1NThiMjkuc2V0Q29udGVudChodG1sX2RkZTg3ZTBmOGJkZTRlNzU5YjM5MDc1YjBmOGVmOTRiKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8zN2YwNjRlMWIzNGI0MTQ4YWRjZTZiMjY2ZGYzNDVhOC5iaW5kUG9wdXAocG9wdXBfYWMxY2JkMjUxN2U4NDdjZDgyOGM0YjQ4ZmQ1NThiMjkpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzNjZjZjYzBiNjA0ZjQ5MTdiZjJjYTJhMWZkYWJhOWI3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjY4NjkwMDAwMDAwMDI2LCAtNzkuNDIwNzA5OTk5OTk5OTldLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZTZiMjE4NzcyYzRkNGNkNGJjZGNhYmI5NGJlMTY3MDEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzY5YTczMzE3OTZkNDRhMDliZjQ4ZTk0MzQ5N2IxNWQzID0gJChgPGRpdiBpZD0iaHRtbF82OWE3MzMxNzk2ZDQ0YTA5YmY0OGU5NDM0OTdiMTVkMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2hyaXN0aWUsIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZTZiMjE4NzcyYzRkNGNkNGJjZGNhYmI5NGJlMTY3MDEuc2V0Q29udGVudChodG1sXzY5YTczMzE3OTZkNDRhMDliZjQ4ZTk0MzQ5N2IxNWQzKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8zY2Y2Y2MwYjYwNGY0OTE3YmYyY2EyYTFmZGFiYTliNy5iaW5kUG9wdXAocG9wdXBfZTZiMjE4NzcyYzRkNGNkNGJjZGNhYmI5NGJlMTY3MDEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2RiOWEzZjBlMDFhYTQyOTE5ZDI5YTY4NTMwMGM0M2YxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjY1MDUwMDAwMDAwMDY1LCAtNzkuNDM4OTA5OTk5OTk5OTZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZTg2Mzk5ZjBhODBhNGJlZmI4MzAzNTJjODc5YTE3MTAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2YyYjVlZDRlYjI1ZjRiNGQ4MDg2Y2E2MzI3ZjFjMjFlID0gJChgPGRpdiBpZD0iaHRtbF9mMmI1ZWQ0ZWIyNWY0YjRkODA4NmNhNjMyN2YxYzIxZSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RHVmZmVyaW4sIERvdmVyY291cnQgVmlsbGFnZSwgV2VzdCBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2U4NjM5OWYwYTgwYTRiZWZiODMwMzUyYzg3OWExNzEwLnNldENvbnRlbnQoaHRtbF9mMmI1ZWQ0ZWIyNWY0YjRkODA4NmNhNjMyN2YxYzIxZSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfZGI5YTNmMGUwMWFhNDI5MTlkMjlhNjg1MzAwYzQzZjEuYmluZFBvcHVwKHBvcHVwX2U4NjM5OWYwYTgwYTRiZWZiODMwMzUyYzg3OWExNzEwKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yMDA4NDUxY2FmZGY0MmQwODI2YjQzZDQxZmQ3MzkyZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0ODQ4MDAwMDAwMDA2LCAtNzkuNDE3NzM5OTk5OTk5OThdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZWQ5NmEzMWExZDllNDEzNmJjNDVhMTQ2NjhiMGE4YjQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2JmYjQ5YzM3NDkwNjQzMjY5ZThlNmY5ZjNmODI1OTQxID0gJChgPGRpdiBpZD0iaHRtbF9iZmI0OWMzNzQ5MDY0MzI2OWU4ZTZmOWYzZjgyNTk0MSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGl0dGxlIFBvcnR1Z2FsLCBUcmluaXR5LCBXZXN0IFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZWQ5NmEzMWExZDllNDEzNmJjNDVhMTQ2NjhiMGE4YjQuc2V0Q29udGVudChodG1sX2JmYjQ5YzM3NDkwNjQzMjY5ZThlNmY5ZjNmODI1OTQxKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8yMDA4NDUxY2FmZGY0MmQwODI2YjQzZDQxZmQ3MzkyZS5iaW5kUG9wdXAocG9wdXBfZWQ5NmEzMWExZDllNDEzNmJjNDVhMTQ2NjhiMGE4YjQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2QwNDgxMjk4Mzg0MjQxODc5NDdiN2UyMjk1MTRmN2NhID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjM5NDEwMDAwMDAwMDU1LCAtNzkuNDI2NzU5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZjJkNGM0MzVkYjIyNGM3Y2E4ZWU5ZjhlMDgyY2Y3ZDggPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2FjZTMyMmRiNmQwNjQzN2U5NDgyNWNiNzg0MmI4OTJjID0gJChgPGRpdiBpZD0iaHRtbF9hY2UzMjJkYjZkMDY0MzdlOTQ4MjVjYjc4NDJiODkyYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QnJvY2t0b24sIFBhcmtkYWxlIFZpbGxhZ2UsIEV4aGliaXRpb24gUGxhY2UsIFdlc3QgVG9yb250bzwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9mMmQ0YzQzNWRiMjI0YzdjYThlZTlmOGUwODJjZjdkOC5zZXRDb250ZW50KGh0bWxfYWNlMzIyZGI2ZDA2NDM3ZTk0ODI1Y2I3ODQyYjg5MmMpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2QwNDgxMjk4Mzg0MjQxODc5NDdiN2UyMjk1MTRmN2NhLmJpbmRQb3B1cChwb3B1cF9mMmQ0YzQzNWRiMjI0YzdjYThlZTlmOGUwODJjZjdkOCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYjUxMDYyMTM2N2Q3NDI3MjhmNDlhMDU0OGY0MmRhYjAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTk3MzAwMDAwMDAwMjUsIC03OS40NjI4MDk5OTk5OTk5M10sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzM5MDAwNDBkNzlmODQxY2E4OWEzNGU5YThlYmNmMmU5KTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9hZWMxYjBmYTY1MGY0NzMyYWQ2MWMxZDA2Y2RhZjI4NyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNTJmMTE3YmExYzU4NGI1NjljOTY1NTQxYTQ4OTI3NTcgPSAkKGA8ZGl2IGlkPSJodG1sXzUyZjExN2JhMWM1ODRiNTY5Yzk2NTU0MWE0ODkyNzU3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5IaWdoIFBhcmssIFRoZSBKdW5jdGlvbiBTb3V0aCwgV2VzdCBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2FlYzFiMGZhNjUwZjQ3MzJhZDYxYzFkMDZjZGFmMjg3LnNldENvbnRlbnQoaHRtbF81MmYxMTdiYTFjNTg0YjU2OWM5NjU1NDFhNDg5Mjc1Nyk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfYjUxMDYyMTM2N2Q3NDI3MjhmNDlhMDU0OGY0MmRhYjAuYmluZFBvcHVwKHBvcHVwX2FlYzFiMGZhNjUwZjQ3MzJhZDYxYzFkMDZjZGFmMjg3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mNzllOTczYzBhNDE0MDljOGMyYjFlNzMyN2Q3NGM0OSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0Nzc3MDAwMDAwMDA0LCAtNzkuNDQ5ODg5OTk5OTk5OThdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZjMzNDhlMmY3NzA3NGEzZTkwNzI1NmNjMDIxNThiOTIgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzE4YThhNjFjMDZlZTQ1MDA4NGViODQyZWE1NTc0MjQyID0gJChgPGRpdiBpZD0iaHRtbF8xOGE4YTYxYzA2ZWU0NTAwODRlYjg0MmVhNTU3NDI0MiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGFya2RhbGUsIFJvbmNlc3ZhbGxlcywgV2VzdCBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2YzMzQ4ZTJmNzcwNzRhM2U5MDcyNTZjYzAyMTU4YjkyLnNldENvbnRlbnQoaHRtbF8xOGE4YTYxYzA2ZWU0NTAwODRlYjg0MmVhNTU3NDI0Mik7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfZjc5ZTk3M2MwYTQxNDA5YzhjMmIxZTczMjdkNzRjNDkuYmluZFBvcHVwKHBvcHVwX2YzMzQ4ZTJmNzcwNzRhM2U5MDcyNTZjYzAyMTU4YjkyKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84OWI4NjMzMzk2ZjA0NjgyYmI5YjBhYzdiNzBkNzg2MiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0OTgyMDAwMDAwMDAzNCwgLTc5LjQ3NTQ3OTk5OTk5OTk1XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMzkwMDA0MGQ3OWY4NDFjYTg5YTM0ZTlhOGViY2YyZTkpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2ZjZDk1MTQ2YjkxNzRlOTRhYjRmYmI5NjQwZjEzNzA0ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF80ZmQ4NmJiNzg1MzU0YzNkYmZjNWMwNTAzY2JjNDM0ZCA9ICQoYDxkaXYgaWQ9Imh0bWxfNGZkODZiYjc4NTM1NGMzZGJmYzVjMDUwM2NiYzQzNGQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlJ1bm55bWVkZSwgU3dhbnNlYSwgV2VzdCBUb3JvbnRvPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2ZjZDk1MTQ2YjkxNzRlOTRhYjRmYmI5NjQwZjEzNzA0LnNldENvbnRlbnQoaHRtbF80ZmQ4NmJiNzg1MzU0YzNkYmZjNWMwNTAzY2JjNDM0ZCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfODliODYzMzM5NmYwNDY4MmJiOWIwYWM3YjcwZDc4NjIuYmluZFBvcHVwKHBvcHVwX2ZjZDk1MTQ2YjkxNzRlOTRhYjRmYmI5NjQwZjEzNzA0KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zMmVlYjY5NWVkNGI0ZWI0Yjg1ZDAxNTdhN2RjYzAyZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2MjUzMDAwMDAwMDA2LCAtNzkuMzkxODc5OTk5OTk5OTZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNzU3NDM4MDc2YWEwNDYzZjk1MTU5YjJjMzViZTI1MTcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzZjYjhlMTFlOWI4ZTQ2N2Y4YmM1NDFkMDcyYTAzNmU2ID0gJChgPGRpdiBpZD0iaHRtbF82Y2I4ZTExZTliOGU0NjdmOGJjNTQxZDA3MmEwMzZlNiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UXVlZW4mIzM5O3MgUGFyaywgT250YXJpbyBQcm92aW5jaWFsIEdvdmVybm1lbnQsIERvd250b3duIFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNzU3NDM4MDc2YWEwNDYzZjk1MTU5YjJjMzViZTI1MTcuc2V0Q29udGVudChodG1sXzZjYjhlMTFlOWI4ZTQ2N2Y4YmM1NDFkMDcyYTAzNmU2KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8zMmVlYjY5NWVkNGI0ZWI0Yjg1ZDAxNTdhN2RjYzAyZC5iaW5kUG9wdXAocG9wdXBfNzU3NDM4MDc2YWEwNDYzZjk1MTU5YjJjMzViZTI1MTcpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzdiZDYzYmUyNjViNTQ4YjU5NzYyYzA0YThjYjQ3OTZjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4NjkwMDAwMDAwMDQ1LCAtNzkuMzg1NDM5OTk5OTk5OTZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF8zOTAwMDQwZDc5Zjg0MWNhODlhMzRlOWE4ZWJjZjJlOSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMmQyMTA2MDkzMWRmNGIwYmExNWI0YmM3OGRjYjc4MDcgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2NkODEwMzk3Y2JmZDQ4YmNiYjJkZWIzYzYxM2RkODRkID0gJChgPGRpdiBpZD0iaHRtbF9jZDgxMDM5N2NiZmQ0OGJjYmIyZGViM2M2MTNkZDg0ZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+QnVzaW5lc3MgcmVwbHkgbWFpbCBQcm9jZXNzaW5nIENlbnRyZSwgU291dGggQ2VudHJhbCBMZXR0ZXIgUHJvY2Vzc2luZyBQbGFudCBUb3JvbnRvLCBFYXN0IFRvcm9udG88L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMmQyMTA2MDkzMWRmNGIwYmExNWI0YmM3OGRjYjc4MDcuc2V0Q29udGVudChodG1sX2NkODEwMzk3Y2JmZDQ4YmNiYjJkZWIzYzYxM2RkODRkKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl83YmQ2M2JlMjY1YjU0OGI1OTc2MmMwNGE4Y2I0Nzk2Yy5iaW5kUG9wdXAocG9wdXBfMmQyMTA2MDkzMWRmNGIwYmExNWI0YmM3OGRjYjc4MDcpCiAgICAgICAgOwoKICAgICAgICAKICAgIAo8L3NjcmlwdD4= onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python

```


```python
# Function to fetch venues data from foursquare

def getNearbyVenues(names, latitudes, longitudes, radius=500, limit = 100):
    
    venues_list=[]
    for name, lat, lng in zip(names, latitudes, longitudes):
            
        # create the API request URL
        url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&v={}&ll={},{}&radius={}&limit={}'.format(
            CLIENT_ID, 
            CLIENT_SECRET, 
            VERSION, 
            lat, 
            lng, 
            radius, 
            limit)
            
        # make the GET request
        results = requests.get(url).json()["response"]['groups'][0]['items']
        
        # return only relevant information for each nearby venue
        venues_list.append([(
            name, 
            lat, 
            lng, 
            v['venue']['name'], 
            v['venue']['location']['lat'], 
            v['venue']['location']['lng'],  
            v['venue']['categories'][0]['name']) for v in results])

    nearby_venues = pd.DataFrame([item for venue_list in venues_list for item in venue_list])
    nearby_venues.columns = ['Neighbourhood', 
                  'Neighbourhood Latitude', 
                  'Neighbourhood Longitude', 
                  'Venue', 
                  'Venue Latitude', 
                  'Venue Longitude', 
                  'Venue Category']
    
    return(nearby_venues)
```


```python
# Get venues associated to each postal code

df_venues = getNearbyVenues(names=df['Neighbourhood'],
                            latitudes=df['Latitude'],
                            longitudes=df['Longitude']
                           )
df_venues.head()
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
      <th>Neighbourhood</th>
      <th>Neighbourhood Latitude</th>
      <th>Neighbourhood Longitude</th>
      <th>Venue</th>
      <th>Venue Latitude</th>
      <th>Venue Longitude</th>
      <th>Venue Category</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>The Beaches</td>
      <td>43.67709</td>
      <td>-79.29547</td>
      <td>Glen Manor Ravine</td>
      <td>43.676821</td>
      <td>-79.293942</td>
      <td>Trail</td>
    </tr>
    <tr>
      <th>1</th>
      <td>The Beaches</td>
      <td>43.67709</td>
      <td>-79.29547</td>
      <td>The Big Carrot Natural Food Market</td>
      <td>43.678879</td>
      <td>-79.297734</td>
      <td>Health Food Store</td>
    </tr>
    <tr>
      <th>2</th>
      <td>The Beaches</td>
      <td>43.67709</td>
      <td>-79.29547</td>
      <td>Grover Pub and Grub</td>
      <td>43.679181</td>
      <td>-79.297215</td>
      <td>Pub</td>
    </tr>
    <tr>
      <th>3</th>
      <td>The Beaches</td>
      <td>43.67709</td>
      <td>-79.29547</td>
      <td>Upper Beaches</td>
      <td>43.680563</td>
      <td>-79.292869</td>
      <td>Neighborhood</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Danforth West, Riverdale</td>
      <td>43.68375</td>
      <td>-79.35512</td>
      <td>Dairy Queen</td>
      <td>43.684223</td>
      <td>-79.357062</td>
      <td>Ice Cream Shop</td>
    </tr>
  </tbody>
</table>
</div>




```python
# One hot encoding
df_onehot = pd.get_dummies(df_venues[['Venue Category']], prefix="", prefix_sep="")

# add neighborhood column back to dataframe
df_onehot['Neighbourhood'] = df_venues['Neighbourhood'] 

# move neighborhood column to the first column
fixed_columns = [df_onehot.columns[-1]] + list(df_onehot.columns[:-1])
df_onehot = df_onehot[fixed_columns]

df_onehot.head()
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
      <th>Neighbourhood</th>
      <th>Accessories Store</th>
      <th>Afghan Restaurant</th>
      <th>American Restaurant</th>
      <th>Antique Shop</th>
      <th>Aquarium</th>
      <th>Art Gallery</th>
      <th>Art Museum</th>
      <th>Arts &amp; Crafts Store</th>
      <th>Asian Restaurant</th>
      <th>Athletics &amp; Sports</th>
      <th>BBQ Joint</th>
      <th>Baby Store</th>
      <th>Bagel Shop</th>
      <th>Bakery</th>
      <th>Bank</th>
      <th>Bar</th>
      <th>Basketball Stadium</th>
      <th>Beer Bar</th>
      <th>Beer Store</th>
      <th>Belgian Restaurant</th>
      <th>Bike Trail</th>
      <th>Bistro</th>
      <th>Boat or Ferry</th>
      <th>Bookstore</th>
      <th>Boutique</th>
      <th>Brazilian Restaurant</th>
      <th>Breakfast Spot</th>
      <th>Brewery</th>
      <th>Bubble Tea Shop</th>
      <th>Building</th>
      <th>Burger Joint</th>
      <th>Burrito Place</th>
      <th>Bus Line</th>
      <th>Business Service</th>
      <th>Butcher</th>
      <th>Café</th>
      <th>Camera Store</th>
      <th>Candy Store</th>
      <th>Caribbean Restaurant</th>
      <th>Cheese Shop</th>
      <th>Chinese Restaurant</th>
      <th>Chiropractor</th>
      <th>Climbing Gym</th>
      <th>Clothing Store</th>
      <th>Cocktail Bar</th>
      <th>Coffee Shop</th>
      <th>College Arts Building</th>
      <th>College Gym</th>
      <th>College Rec Center</th>
      <th>Colombian Restaurant</th>
      <th>Comfort Food Restaurant</th>
      <th>Comic Shop</th>
      <th>Concert Hall</th>
      <th>Convenience Store</th>
      <th>Cosmetics Shop</th>
      <th>Costume Shop</th>
      <th>Creperie</th>
      <th>Cuban Restaurant</th>
      <th>Cupcake Shop</th>
      <th>Dance Studio</th>
      <th>Deli / Bodega</th>
      <th>Department Store</th>
      <th>Dessert Shop</th>
      <th>Dim Sum Restaurant</th>
      <th>Diner</th>
      <th>Discount Store</th>
      <th>Distribution Center</th>
      <th>Dog Run</th>
      <th>Donut Shop</th>
      <th>Dumpling Restaurant</th>
      <th>Eastern European Restaurant</th>
      <th>Electronics Store</th>
      <th>Elementary School</th>
      <th>Escape Room</th>
      <th>Ethiopian Restaurant</th>
      <th>Event Space</th>
      <th>Falafel Restaurant</th>
      <th>Farm</th>
      <th>Farmers Market</th>
      <th>Fast Food Restaurant</th>
      <th>Fish &amp; Chips Shop</th>
      <th>Fish Market</th>
      <th>Flea Market</th>
      <th>Flower Shop</th>
      <th>Food</th>
      <th>Food &amp; Drink Shop</th>
      <th>Food Court</th>
      <th>Food Truck</th>
      <th>Fountain</th>
      <th>French Restaurant</th>
      <th>Fried Chicken Joint</th>
      <th>Frozen Yogurt Shop</th>
      <th>Furniture / Home Store</th>
      <th>Gaming Cafe</th>
      <th>Garden</th>
      <th>Gas Station</th>
      <th>Gastropub</th>
      <th>Gay Bar</th>
      <th>General Entertainment</th>
      <th>General Travel</th>
      <th>German Restaurant</th>
      <th>Gift Shop</th>
      <th>Gluten-free Restaurant</th>
      <th>Gourmet Shop</th>
      <th>Greek Restaurant</th>
      <th>Grocery Store</th>
      <th>Gym</th>
      <th>Gym / Fitness Center</th>
      <th>Gym Pool</th>
      <th>Hawaiian Restaurant</th>
      <th>Health Food Store</th>
      <th>Historic Site</th>
      <th>History Museum</th>
      <th>Hobby Shop</th>
      <th>Hookah Bar</th>
      <th>Hotel</th>
      <th>Hotel Bar</th>
      <th>IT Services</th>
      <th>Ice Cream Shop</th>
      <th>Indian Restaurant</th>
      <th>Indoor Play Area</th>
      <th>Intersection</th>
      <th>Irish Pub</th>
      <th>Italian Restaurant</th>
      <th>Japanese Restaurant</th>
      <th>Jazz Club</th>
      <th>Jewelry Store</th>
      <th>Juice Bar</th>
      <th>Kitchen Supply Store</th>
      <th>Korean Restaurant</th>
      <th>Lake</th>
      <th>Latin American Restaurant</th>
      <th>Light Rail Station</th>
      <th>Lingerie Store</th>
      <th>Liquor Store</th>
      <th>Lounge</th>
      <th>Malay Restaurant</th>
      <th>Market</th>
      <th>Martial Arts School</th>
      <th>Massage Studio</th>
      <th>Mediterranean Restaurant</th>
      <th>Men's Store</th>
      <th>Mexican Restaurant</th>
      <th>Middle Eastern Restaurant</th>
      <th>Miscellaneous Shop</th>
      <th>Modern European Restaurant</th>
      <th>Molecular Gastronomy Restaurant</th>
      <th>Monument / Landmark</th>
      <th>Moroccan Restaurant</th>
      <th>Movie Theater</th>
      <th>Moving Target</th>
      <th>Museum</th>
      <th>Music Venue</th>
      <th>Neighborhood</th>
      <th>New American Restaurant</th>
      <th>Nightclub</th>
      <th>Noodle House</th>
      <th>Office</th>
      <th>Opera House</th>
      <th>Optical Shop</th>
      <th>Organic Grocery</th>
      <th>Other Great Outdoors</th>
      <th>Other Nightlife</th>
      <th>Park</th>
      <th>Performing Arts Venue</th>
      <th>Peruvian Restaurant</th>
      <th>Pet Store</th>
      <th>Pharmacy</th>
      <th>Pilates Studio</th>
      <th>Pizza Place</th>
      <th>Playground</th>
      <th>Plaza</th>
      <th>Poke Place</th>
      <th>Portuguese Restaurant</th>
      <th>Poutine Place</th>
      <th>Pub</th>
      <th>Ramen Restaurant</th>
      <th>Record Shop</th>
      <th>Residential Building (Apartment / Condo)</th>
      <th>Restaurant</th>
      <th>Roof Deck</th>
      <th>Sake Bar</th>
      <th>Salad Place</th>
      <th>Salon / Barbershop</th>
      <th>Sandwich Place</th>
      <th>Sculpture Garden</th>
      <th>Seafood Restaurant</th>
      <th>Shoe Store</th>
      <th>Shop &amp; Service</th>
      <th>Shopping Mall</th>
      <th>Smoke Shop</th>
      <th>Smoothie Shop</th>
      <th>Snack Place</th>
      <th>Soccer Field</th>
      <th>Soup Place</th>
      <th>Southern / Soul Food Restaurant</th>
      <th>Souvlaki Shop</th>
      <th>Spa</th>
      <th>Speakeasy</th>
      <th>Sporting Goods Shop</th>
      <th>Sports Bar</th>
      <th>Stadium</th>
      <th>Steakhouse</th>
      <th>Strip Club</th>
      <th>Supermarket</th>
      <th>Sushi Restaurant</th>
      <th>Swim School</th>
      <th>Taco Place</th>
      <th>Tailor Shop</th>
      <th>Taiwanese Restaurant</th>
      <th>Tanning Salon</th>
      <th>Tea Room</th>
      <th>Tennis Court</th>
      <th>Thai Restaurant</th>
      <th>Theater</th>
      <th>Theme Restaurant</th>
      <th>Thrift / Vintage Store</th>
      <th>Toy / Game Store</th>
      <th>Trail</th>
      <th>Train Station</th>
      <th>Vegetarian / Vegan Restaurant</th>
      <th>Veterinarian</th>
      <th>Video Game Store</th>
      <th>Vietnamese Restaurant</th>
      <th>Wine Bar</th>
      <th>Wine Shop</th>
      <th>Wings Joint</th>
      <th>Women's Store</th>
      <th>Yoga Studio</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>The Beaches</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>The Beaches</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>The Beaches</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>The Beaches</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>The Danforth West, Riverdale</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_grouped = df_onehot.groupby('Neighbourhood').mean().reset_index()
```


```python

```


```python
# Auxiliar Stuff

# Get neighborhood most common venues

def return_most_common_venues(row, num_top_venues):
    row_categories = row.iloc[1:]
    row_categories_sorted = row_categories.sort_values(ascending=False)
    
    return row_categories_sorted.index.values[0:num_top_venues]
num_top_venues = 10

indicators = ['st', 'nd', 'rd']

# create columns according to number of top venues
columns = ['Neighbourhood']
for ind in np.arange(num_top_venues):
    try:
        columns.append('{}{} Most Common Venue'.format(ind+1, indicators[ind]))
    except:
        columns.append('{}th Most Common Venue'.format(ind+1))

# create a new dataframe
neighborhoods_venues_sorted = pd.DataFrame(columns=columns)
neighborhoods_venues_sorted['Neighbourhood'] = df_grouped['Neighbourhood']

for ind in np.arange(df_grouped.shape[0]):
    neighborhoods_venues_sorted.iloc[ind, 1:] = return_most_common_venues(df_grouped.iloc[ind, :], num_top_venues)

neighborhoods_venues_sorted["Cluster Labels"] = 0
    
neighborhoods_venues_sorted.head()
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
      <th>Neighbourhood</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
      <th>Cluster Labels</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Berczy Park</td>
      <td>Coffee Shop</td>
      <td>Farmers Market</td>
      <td>Beer Bar</td>
      <td>Breakfast Spot</td>
      <td>Cocktail Bar</td>
      <td>Restaurant</td>
      <td>Cheese Shop</td>
      <td>Bakery</td>
      <td>Seafood Restaurant</td>
      <td>Lounge</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Brockton, Parkdale Village, Exhibition Place</td>
      <td>Coffee Shop</td>
      <td>Bar</td>
      <td>Café</td>
      <td>Restaurant</td>
      <td>Gift Shop</td>
      <td>Sandwich Place</td>
      <td>Nightclub</td>
      <td>Japanese Restaurant</td>
      <td>Supermarket</td>
      <td>Furniture / Home Store</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Business reply mail Processing Centre, South C...</td>
      <td>Coffee Shop</td>
      <td>Hotel</td>
      <td>Restaurant</td>
      <td>Café</td>
      <td>Italian Restaurant</td>
      <td>Bar</td>
      <td>Asian Restaurant</td>
      <td>Salon / Barbershop</td>
      <td>Thai Restaurant</td>
      <td>Pub</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>CN Tower, King and Spadina, Railway Lands, Har...</td>
      <td>Italian Restaurant</td>
      <td>Café</td>
      <td>Coffee Shop</td>
      <td>Bar</td>
      <td>Park</td>
      <td>French Restaurant</td>
      <td>Lounge</td>
      <td>Sandwich Place</td>
      <td>Restaurant</td>
      <td>Gym / Fitness Center</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Central Bay Street</td>
      <td>Coffee Shop</td>
      <td>Clothing Store</td>
      <td>Restaurant</td>
      <td>Sandwich Place</td>
      <td>Sushi Restaurant</td>
      <td>Café</td>
      <td>Plaza</td>
      <td>Bubble Tea Shop</td>
      <td>Cosmetics Shop</td>
      <td>Bookstore</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>




```python

```


```python
# Apply k-means to new grouped dataframe 

# set number of clusters
kclusters = 5

df_grouped_clustering = df_grouped.drop('Neighbourhood', 1)

# run k-means clustering
kmeans = KMeans(n_clusters=kclusters, random_state=0).fit(df_grouped_clustering)

# check cluster labels generated for each row in the dataframe
kmeans.labels_[0:10] 

# add clustering labels
neighborhoods_venues_sorted['Cluster Labels'] = kmeans.labels_
```


```python

df_merged = df

# merge manhattan_grouped with manhattan_data to add latitude/longitude for each neighborhood
df_merged = df_merged.join(neighborhoods_venues_sorted.set_index('Neighbourhood'), on='Neighbourhood')

df_merged = df_merged[~df_merged["Cluster Labels"].isna()]

df_merged.head() # check the last columns!
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
      <th>index</th>
      <th>Postal Code</th>
      <th>Borough</th>
      <th>Neighbourhood</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
      <th>Cluster Labels</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>37</td>
      <td>M4E</td>
      <td>East Toronto</td>
      <td>The Beaches</td>
      <td>43.67709</td>
      <td>-79.29547</td>
      <td>Health Food Store</td>
      <td>Pub</td>
      <td>Trail</td>
      <td>Neighborhood</td>
      <td>Yoga Studio</td>
      <td>Eastern European Restaurant</td>
      <td>Fish &amp; Chips Shop</td>
      <td>Fast Food Restaurant</td>
      <td>Farmers Market</td>
      <td>Farm</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>41</td>
      <td>M4K</td>
      <td>East Toronto</td>
      <td>The Danforth West, Riverdale</td>
      <td>43.68375</td>
      <td>-79.35512</td>
      <td>Intersection</td>
      <td>Bus Line</td>
      <td>Business Service</td>
      <td>Coffee Shop</td>
      <td>Park</td>
      <td>Ice Cream Shop</td>
      <td>Grocery Store</td>
      <td>Discount Store</td>
      <td>Farmers Market</td>
      <td>Event Space</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>42</td>
      <td>M4L</td>
      <td>East Toronto</td>
      <td>India Bazaar, The Beaches West</td>
      <td>43.66797</td>
      <td>-79.31467</td>
      <td>Park</td>
      <td>Gym</td>
      <td>Italian Restaurant</td>
      <td>Pet Store</td>
      <td>Pizza Place</td>
      <td>Pub</td>
      <td>Coffee Shop</td>
      <td>Restaurant</td>
      <td>Movie Theater</td>
      <td>Sandwich Place</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>43</td>
      <td>M4M</td>
      <td>East Toronto</td>
      <td>Studio District</td>
      <td>43.66213</td>
      <td>-79.33497</td>
      <td>Pizza Place</td>
      <td>Diner</td>
      <td>Brewery</td>
      <td>Italian Restaurant</td>
      <td>Arts &amp; Crafts Store</td>
      <td>Bar</td>
      <td>Bakery</td>
      <td>Gastropub</td>
      <td>Coffee Shop</td>
      <td>Sushi Restaurant</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>44</td>
      <td>M4N</td>
      <td>Central Toronto</td>
      <td>Lawrence Park</td>
      <td>43.72843</td>
      <td>-79.38713</td>
      <td>Bus Line</td>
      <td>Swim School</td>
      <td>Yoga Studio</td>
      <td>Elementary School</td>
      <td>Flea Market</td>
      <td>Fish Market</td>
      <td>Fish &amp; Chips Shop</td>
      <td>Fast Food Restaurant</td>
      <td>Farmers Market</td>
      <td>Farm</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# create map
map_clusters = folium.Map(location=[latitude, longitude], zoom_start=11)

# set color scheme for the clusters
x = np.arange(kclusters)
ys = [i + x + (i*x)**2 for i in range(kclusters)]
colors_array = cm.rainbow(np.linspace(0, 1, len(ys)))
rainbow = [colors.rgb2hex(i) for i in colors_array]

# add markers to the map
markers_colors = []
for lat, lon, poi, cluster in zip(df_merged['Latitude'], df_merged['Longitude'], df_merged['Neighbourhood'], df_merged['Cluster Labels']):
    label = folium.Popup(str(poi) + ' Cluster ' + str(cluster), parse_html=True)
    folium.CircleMarker(
        [lat, lon],
        radius=5,
        popup=label,
        color=rainbow[int(cluster-1)],
        fill=True,
        fill_color=rainbow[int(cluster-1)],
        fill_opacity=0.7).add_to(map_clusters)
       
map_clusters
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe src="about:blank" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" data-html=PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjYuMC9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxIiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFs0My42NTM0ODE3LCAtNzkuMzgzOTM0N10sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiAxMSwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyX2QzYWFjM2I1M2Q4MDQ5NzBhNWU4ZTk5YTMzYmQzODg0ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNGUxOThhZGIwOGQxNDMzZDg5YzZmOTY4MDQ3ZDE0MGQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NzcwOTAwMDAwMDAwOCwgLTc5LjI5NTQ2OTk5OTk5OTk3XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjODBmZmI0IiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiM4MGZmYjQiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzkyYjcxZWMxMWNiNzQyODU4ZDkyYTVmZTY2NGE1MzYwID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF82Y2U1ZmZmNmRjNTE0OWM3YTgyODFlN2Y2OTBiNjMyOCA9ICQoYDxkaXYgaWQ9Imh0bWxfNmNlNWZmZjZkYzUxNDljN2E4MjgxZTdmNjkwYjYzMjgiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRoZSBCZWFjaGVzIENsdXN0ZXIgMy4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzkyYjcxZWMxMWNiNzQyODU4ZDkyYTVmZTY2NGE1MzYwLnNldENvbnRlbnQoaHRtbF82Y2U1ZmZmNmRjNTE0OWM3YTgyODFlN2Y2OTBiNjMyOCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNGUxOThhZGIwOGQxNDMzZDg5YzZmOTY4MDQ3ZDE0MGQuYmluZFBvcHVwKHBvcHVwXzkyYjcxZWMxMWNiNzQyODU4ZDkyYTVmZTY2NGE1MzYwKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mODg3ZjA2ZTk0M2I0NTBiYmQ3NjVjNzRmY2Q4OGM0MCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY4Mzc1MDAwMDAwMDAzLCAtNzkuMzU1MTE5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMGRhZTQxYjUzZjg3NGQ2MDkwNmZjZmNmYTQxYjg3YzggPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzJkMWEwZDBkZDc0NzQzYTJiZGY5NjFmNjQyMTU1OTYwID0gJChgPGRpdiBpZD0iaHRtbF8yZDFhMGQwZGQ3NDc0M2EyYmRmOTYxZjY0MjE1NTk2MCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+VGhlIERhbmZvcnRoIFdlc3QsIFJpdmVyZGFsZSBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wZGFlNDFiNTNmODc0ZDYwOTA2ZmNmY2ZhNDFiODdjOC5zZXRDb250ZW50KGh0bWxfMmQxYTBkMGRkNzQ3NDNhMmJkZjk2MWY2NDIxNTU5NjApOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2Y4ODdmMDZlOTQzYjQ1MGJiZDc2NWM3NGZjZDg4YzQwLmJpbmRQb3B1cChwb3B1cF8wZGFlNDFiNTNmODc0ZDYwOTA2ZmNmY2ZhNDFiODdjOCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfYTM1MjUyMjA4ZTgyNDk0Mjg2MDc5NDQ0YTRkODBkNmMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42Njc5NzAwMDAwMDAwMjUsIC03OS4zMTQ2Njk5OTk5OTk5OF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8yYjBkYWIwYWRhYWI0YTFjOWEyMzI5NmMzZjJhZWQ5MiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjA0ZTM0ZjM4YzY1NGZmYjlhYTIyNTg0MDZkZjk4YTAgPSAkKGA8ZGl2IGlkPSJodG1sXzYwNGUzNGYzOGM2NTRmZmI5YWEyMjU4NDA2ZGY5OGEwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5JbmRpYSBCYXphYXIsIFRoZSBCZWFjaGVzIFdlc3QgQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMmIwZGFiMGFkYWFiNGExYzlhMjMyOTZjM2YyYWVkOTIuc2V0Q29udGVudChodG1sXzYwNGUzNGYzOGM2NTRmZmI5YWEyMjU4NDA2ZGY5OGEwKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9hMzUyNTIyMDhlODI0OTQyODYwNzk0NDRhNGQ4MGQ2Yy5iaW5kUG9wdXAocG9wdXBfMmIwZGFiMGFkYWFiNGExYzlhMjMyOTZjM2YyYWVkOTIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzM1MDQ5YTkyYjgwNzQ1ZDc4OTEwOTJlNWU4MGNjNjJjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjYyMTMwMDAwMDAwMDUsIC03OS4zMzQ5Njk5OTk5OTk5NF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jZTBmNjA5NmUwZDY0NDNlOGVmMjExMmRjY2ZkNjQ3ZCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjI2MGVlMTcyZWQ0NDM4NGFlMmFkZmQ4MWM3MDJlYTkgPSAkKGA8ZGl2IGlkPSJodG1sX2IyNjBlZTE3MmVkNDQzODRhZTJhZGZkODFjNzAyZWE5IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5TdHVkaW8gRGlzdHJpY3QgQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfY2UwZjYwOTZlMGQ2NDQzZThlZjIxMTJkY2NmZDY0N2Quc2V0Q29udGVudChodG1sX2IyNjBlZTE3MmVkNDQzODRhZTJhZGZkODFjNzAyZWE5KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8zNTA0OWE5MmI4MDc0NWQ3ODkxMDkyZTVlODBjYzYyYy5iaW5kUG9wdXAocG9wdXBfY2UwZjYwOTZlMGQ2NDQzZThlZjIxMTJkY2NmZDY0N2QpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzQ5ZDE1ZDUxYjdiNjQ1ZDdiMzM0ZWVjODVmNTIzNzAxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNzI4NDMwMDAwMDAwMDYsIC03OS4zODcxMjk5OTk5OTk5Nl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiIzgwMDBmZiIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjODAwMGZmIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8zNjkzM2Y3ODMwZjM0ZTliYjFhYzQwMmE2NjZkZDY1MCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfOGJkYjdlYzE3YTljNGJjYTk3YmMyOWI2ZTYzNjQ3Y2EgPSAkKGA8ZGl2IGlkPSJodG1sXzhiZGI3ZWMxN2E5YzRiY2E5N2JjMjliNmU2MzY0N2NhIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5MYXdyZW5jZSBQYXJrIENsdXN0ZXIgMS4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzM2OTMzZjc4MzBmMzRlOWJiMWFjNDAyYTY2NmRkNjUwLnNldENvbnRlbnQoaHRtbF84YmRiN2VjMTdhOWM0YmNhOTdiYzI5YjZlNjM2NDdjYSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNDlkMTVkNTFiN2I2NDVkN2IzMzRlZWM4NWY1MjM3MDEuYmluZFBvcHVwKHBvcHVwXzM2OTMzZjc4MzBmMzRlOWJiMWFjNDAyYTY2NmRkNjUwKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wNjJkNGM5OTM1NTY0ZjAxYTYwYmI5ZmU1MmE0YjdlMCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjcxMjc2MDAwMDAwMDA2LCAtNzkuMzg4NTA5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMzViNWFkZjYwNzBmNDE2NTlmYjAwNDg0ZjUxYWIyNWEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzQyOTM1MTc4Y2M4NzQxMTc5NzM5MWZkMTNiNjVmNzEwID0gJChgPGRpdiBpZD0iaHRtbF80MjkzNTE3OGNjODc0MTE3OTczOTFmZDEzYjY1ZjcxMCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+RGF2aXN2aWxsZSBOb3J0aCBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8zNWI1YWRmNjA3MGY0MTY1OWZiMDA0ODRmNTFhYjI1YS5zZXRDb250ZW50KGh0bWxfNDI5MzUxNzhjYzg3NDExNzk3MzkxZmQxM2I2NWY3MTApOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzA2MmQ0Yzk5MzU1NjRmMDFhNjBiYjlmZTUyYTRiN2UwLmJpbmRQb3B1cChwb3B1cF8zNWI1YWRmNjA3MGY0MTY1OWZiMDA0ODRmNTFhYjI1YSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZTIzOGNkOGNjYTQ5NDAxNWJhYzhhZTg0MDlhZDU1ZWEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MTQ1ODAwMDAwMDAwNywgLTc5LjQwNjY3OTk5OTk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmZiMzYwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZmIzNjAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2ZjY2RlMDQ5NWFiZTQyODFiNDczMDdkNDc5MTBhNTEzID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF81ZTM0MzczNGY1YjM0MzM0ODY3NjA2NmRjOGI2M2RjZSA9ICQoYDxkaXYgaWQ9Imh0bWxfNWUzNDM3MzRmNWIzNDMzNDg2NzYwNjZkYzhiNjNkY2UiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk5vcnRoIFRvcm9udG8gV2VzdCwgTGF3cmVuY2UgUGFyayBDbHVzdGVyIDQuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9mY2NkZTA0OTVhYmU0MjgxYjQ3MzA3ZDQ3OTEwYTUxMy5zZXRDb250ZW50KGh0bWxfNWUzNDM3MzRmNWIzNDMzNDg2NzYwNjZkYzhiNjNkY2UpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2UyMzhjZDhjY2E0OTQwMTViYWM4YWU4NDA5YWQ1NWVhLmJpbmRQb3B1cChwb3B1cF9mY2NkZTA0OTVhYmU0MjgxYjQ3MzA3ZDQ3OTEwYTUxMykKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZDQxYzBiMzRhMGYzNDczMWE1ODIwNTk0ZWQxOTRkZGMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My43MDM0MDAwMDAwMDAwNDUsIC03OS4zODY1ODk5OTk5OTk5Nl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80MDZkZTk3NDIzYjI0YjY0OGJlNjM4NDhlNTkzYzk4ZSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYTkwZmM4ZDljMWM3NGYyYTkzMWJhYjRhMTFlMjdhNGQgPSAkKGA8ZGl2IGlkPSJodG1sX2E5MGZjOGQ5YzFjNzRmMmE5MzFiYWI0YTExZTI3YTRkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5EYXZpc3ZpbGxlIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzQwNmRlOTc0MjNiMjRiNjQ4YmU2Mzg0OGU1OTNjOThlLnNldENvbnRlbnQoaHRtbF9hOTBmYzhkOWMxYzc0ZjJhOTMxYmFiNGExMWUyN2E0ZCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfZDQxYzBiMzRhMGYzNDczMWE1ODIwNTk0ZWQxOTRkZGMuYmluZFBvcHVwKHBvcHVwXzQwNmRlOTc0MjNiMjRiNjQ4YmU2Mzg0OGU1OTNjOThlKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lZTU1YTdjYTQ4ZmY0MzAzODE4YzczNWM4NmYzNjljMCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY5MDQ4MDAwMDAwMDAzNiwgLTc5LjM4MzE3OTk5OTk5OTk4XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2E3OGRjYjNjYTAyYjRjYWNhMjE3Y2E3MTQwMjVjZmM4ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9jYjM1MGNiNTljZGU0ZDY2ODA1MWRiNjUzYzdlYjdmMyA9ICQoYDxkaXYgaWQ9Imh0bWxfY2IzNTBjYjU5Y2RlNGQ2NjgwNTFkYjY1M2M3ZWI3ZjMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPk1vb3JlIFBhcmssIFN1bW1lcmhpbGwgRWFzdCBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9hNzhkY2IzY2EwMmI0Y2FjYTIxN2NhNzE0MDI1Y2ZjOC5zZXRDb250ZW50KGh0bWxfY2IzNTBjYjU5Y2RlNGQ2NjgwNTFkYjY1M2M3ZWI3ZjMpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2VlNTVhN2NhNDhmZjQzMDM4MThjNzM1Yzg2ZjM2OWMwLmJpbmRQb3B1cChwb3B1cF9hNzhkY2IzY2EwMmI0Y2FjYTIxN2NhNzE0MDI1Y2ZjOCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNjBlZTQ0ZmQyNmI4NDgwODhkODZmNTc3NzBmYTQ4NzAgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42ODU2ODAwMDAwMDAwNSwgLTc5LjQwMjM2OTk5OTk5OTk2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzA0NmFkMWM2MTM1YjRmMGE4YWU5NDdhZDM4MTkxMTRiID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF81ODJmYjI5Y2Y1MjA0ODlmODc2M2RjYTVhZjgwYjc5MyA9ICQoYDxkaXYgaWQ9Imh0bWxfNTgyZmIyOWNmNTIwNDg5Zjg3NjNkY2E1YWY4MGI3OTMiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlN1bW1lcmhpbGwgV2VzdCwgUmF0aG5lbGx5LCBTb3V0aCBIaWxsLCBGb3Jlc3QgSGlsbCBTRSwgRGVlciBQYXJrIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzA0NmFkMWM2MTM1YjRmMGE4YWU5NDdhZDM4MTkxMTRiLnNldENvbnRlbnQoaHRtbF81ODJmYjI5Y2Y1MjA0ODlmODc2M2RjYTVhZjgwYjc5Myk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfNjBlZTQ0ZmQyNmI4NDgwODhkODZmNTc3NzBmYTQ4NzAuYmluZFBvcHVwKHBvcHVwXzA0NmFkMWM2MTM1YjRmMGE4YWU5NDdhZDM4MTkxMTRiKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lZjM3ZDRmOGE1ZWQ0MjI3OGFkNzk5YmJiMDA4YzAxMSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY4MTkwMDAwMDAwMDA0LCAtNzkuMzc4Mjg5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZmIzNjAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmYjM2MCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfYzI0MWM3MWNlOTk3NDRjYzgyNzQwNGNiODU2ZGM2ZTUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2IzMDI1ODliYzQ5MTQyMmJiNTliNzBiN2U1ODg1MWUzID0gJChgPGRpdiBpZD0iaHRtbF9iMzAyNTg5YmM0OTE0MjJiYjU5YjcwYjdlNTg4NTFlMyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Um9zZWRhbGUgQ2x1c3RlciA0LjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYzI0MWM3MWNlOTk3NDRjYzgyNzQwNGNiODU2ZGM2ZTUuc2V0Q29udGVudChodG1sX2IzMDI1ODliYzQ5MTQyMmJiNTliNzBiN2U1ODg1MWUzKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9lZjM3ZDRmOGE1ZWQ0MjI3OGFkNzk5YmJiMDA4YzAxMS5iaW5kUG9wdXAocG9wdXBfYzI0MWM3MWNlOTk3NDRjYzgyNzQwNGNiODU2ZGM2ZTUpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzI2NmY2NmRkM2QyNTRjZDg5ZTllMjE1NjhhODJlYjhkID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjY3ODgwMDAwMDAwMDI1LCAtNzkuMzY2NDg5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZmE1NTg4NmZjZDIyNDRmN2JjNWUyZmVlZjU4ZDA0MTQgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzgyYTVhNWI2NjVkMzRjNmI4MTIwZmM2NTQyZDEyOTA4ID0gJChgPGRpdiBpZD0iaHRtbF84MmE1YTViNjY1ZDM0YzZiODEyMGZjNjU0MmQxMjkwOCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U3QuIEphbWVzIFRvd24sIENhYmJhZ2V0b3duIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2ZhNTU4ODZmY2QyMjQ0ZjdiYzVlMmZlZWY1OGQwNDE0LnNldENvbnRlbnQoaHRtbF84MmE1YTViNjY1ZDM0YzZiODEyMGZjNjU0MmQxMjkwOCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfMjY2ZjY2ZGQzZDI1NGNkODllOWUyMTU2OGE4MmViOGQuYmluZFBvcHVwKHBvcHVwX2ZhNTU4ODZmY2QyMjQ0ZjdiYzVlMmZlZWY1OGQwNDE0KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl81MWFlMzhhNGRiMDY0ZTc0OTIzY2Q2NTllYjE5MTM0MSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2NjU5MDAwMDAwMDA0LCAtNzkuMzgxMzI5OTk5OTk5OTNdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMzkwNWU0YjQ1MmU3NDVmNGExYTBlMGIwYThjY2ZkYjEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2YzMjVlODYxNzE2ZjRlZjE5MWQ5ZTMxOTY4ZDA5YWQ0ID0gJChgPGRpdiBpZD0iaHRtbF9mMzI1ZTg2MTcxNmY0ZWYxOTFkOWUzMTk2OGQwOWFkNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q2h1cmNoIGFuZCBXZWxsZXNsZXkgQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMzkwNWU0YjQ1MmU3NDVmNGExYTBlMGIwYThjY2ZkYjEuc2V0Q29udGVudChodG1sX2YzMjVlODYxNzE2ZjRlZjE5MWQ5ZTMxOTY4ZDA5YWQ0KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl81MWFlMzhhNGRiMDY0ZTc0OTIzY2Q2NTllYjE5MTM0MS5iaW5kUG9wdXAocG9wdXBfMzkwNWU0YjQ1MmU3NDVmNGExYTBlMGIwYThjY2ZkYjEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzBkZWY0NTVjMzE1MTQ4NDA5NmRlMDNjMGRjY2MxOWM4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjU1MTIwMDAwMDAwMDcsIC03OS4zNjI2Mzk5OTk5OTk5NF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xZGNkYzE0ZWQxYmY0ODYxOThkNTYzZjEwNjQ3NmMxNSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMzk4NWVhNjk3MjMzNGEwZTgyZDRkYTU1MTFhYjUwODcgPSAkKGA8ZGl2IGlkPSJodG1sXzM5ODVlYTY5NzIzMzRhMGU4MmQ0ZGE1NTExYWI1MDg3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5SZWdlbnQgUGFyaywgSGFyYm91cmZyb250IENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzFkY2RjMTRlZDFiZjQ4NjE5OGQ1NjNmMTA2NDc2YzE1LnNldENvbnRlbnQoaHRtbF8zOTg1ZWE2OTcyMzM0YTBlODJkNGRhNTUxMWFiNTA4Nyk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfMGRlZjQ1NWMzMTUxNDg0MDk2ZGUwM2MwZGNjYzE5YzguYmluZFBvcHVwKHBvcHVwXzFkY2RjMTRlZDFiZjQ4NjE5OGQ1NjNmMTA2NDc2YzE1KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9jOTcyNmUzY2JlNzI0ZjZlYWRhNzY3OGY0OGJmYTViZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1NzM5MDAwMDAwMDA4LCAtNzkuMzc4MDM5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMDg5NmFjYmFlZDQyNGQ2Y2FmY2Q2MzY5MGQ0NjM0YjYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzUwOTEzOTkxMGExYTQyNGViMDdkZjhjNGYwZTFlYmFjID0gJChgPGRpdiBpZD0iaHRtbF81MDkxMzk5MTBhMWE0MjRlYjA3ZGY4YzRmMGUxZWJhYyIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+R2FyZGVuIERpc3RyaWN0LCBSeWVyc29uIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzA4OTZhY2JhZWQ0MjRkNmNhZmNkNjM2OTBkNDYzNGI2LnNldENvbnRlbnQoaHRtbF81MDkxMzk5MTBhMWE0MjRlYjA3ZGY4YzRmMGUxZWJhYyk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfYzk3MjZlM2NiZTcyNGY2ZWFkYTc2NzhmNDhiZmE1YmUuYmluZFBvcHVwKHBvcHVwXzA4OTZhY2JhZWQ0MjRkNmNhZmNkNjM2OTBkNDYzNGI2KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83ZjA3NWEyYmE4MTc0YWIwOGZlOTFiZGVhMzg0NDlhYiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1MjE1MDAwMDAwMDA2LCAtNzkuMzc1ODY5OTk5OTk5OTZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfZDdlMTU0ZTc0NzE0NDdlYmI4ZDQ5YmJjMjU4ZjBmMjggPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzQyZDgxNDAwMDczZDRjZjA5NGJiNTFhYmU1Zjk3NzkwID0gJChgPGRpdiBpZD0iaHRtbF80MmQ4MTQwMDA3M2Q0Y2YwOTRiYjUxYWJlNWY5Nzc5MCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U3QuIEphbWVzIFRvd24gQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfZDdlMTU0ZTc0NzE0NDdlYmI4ZDQ5YmJjMjU4ZjBmMjguc2V0Q29udGVudChodG1sXzQyZDgxNDAwMDczZDRjZjA5NGJiNTFhYmU1Zjk3NzkwKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl83ZjA3NWEyYmE4MTc0YWIwOGZlOTFiZGVhMzg0NDlhYi5iaW5kUG9wdXAocG9wdXBfZDdlMTU0ZTc0NzE0NDdlYmI4ZDQ5YmJjMjU4ZjBmMjgpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2U5MTM3MDZhMWNmYjQxZTk5N2Q3NmQzYjlhYTViNjFmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ1MzYwMDAwMDAwMDQsIC03OS4zNzMwNTk5OTk5OTk5NV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jNmU5MGRkOWIwYjQ0NGMxODNiNDY0NGU2MGM5NDAxNiA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfMGY3MTM2YzhkZmYzNGJhZThkZjA0MWNiZDk0YjdjMzAgPSAkKGA8ZGl2IGlkPSJodG1sXzBmNzEzNmM4ZGZmMzRiYWU4ZGYwNDFjYmQ5NGI3YzMwIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CZXJjenkgUGFyayBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9jNmU5MGRkOWIwYjQ0NGMxODNiNDY0NGU2MGM5NDAxNi5zZXRDb250ZW50KGh0bWxfMGY3MTM2YzhkZmYzNGJhZThkZjA0MWNiZDk0YjdjMzApOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2U5MTM3MDZhMWNmYjQxZTk5N2Q3NmQzYjlhYTViNjFmLmJpbmRQb3B1cChwb3B1cF9jNmU5MGRkOWIwYjQ0NGMxODNiNDY0NGU2MGM5NDAxNikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOGUzY2VlYWY3NjEwNGZiMzk3OTVhMDhlMDU0MGJjYWEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTYwOTAwMDAwMDAwNiwgLTc5LjM4NDkyOTk5OTk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2E1NThiODhhN2FiMDRjZWViNGFiMmQ3ZDRlMDc1YmNlID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF83YzA3Yjg2ZDg2NzM0MmY2YWE4ZWZkMWJkY2NlMmZhZCA9ICQoYDxkaXYgaWQ9Imh0bWxfN2MwN2I4NmQ4NjczNDJmNmFhOGVmZDFiZGNjZTJmYWQiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNlbnRyYWwgQmF5IFN0cmVldCBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF9hNTU4Yjg4YTdhYjA0Y2VlYjRhYjJkN2Q0ZTA3NWJjZS5zZXRDb250ZW50KGh0bWxfN2MwN2I4NmQ4NjczNDJmNmFhOGVmZDFiZGNjZTJmYWQpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzhlM2NlZWFmNzYxMDRmYjM5Nzk1YTA4ZTA1NDBiY2FhLmJpbmRQb3B1cChwb3B1cF9hNTU4Yjg4YTdhYjA0Y2VlYjRhYjJkN2Q0ZTA3NWJjZSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNTIxMTY4YTJmZjllNDBiNjg0N2IzYmQ3ZjIwOWY4MmIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDk3MDAwMDAwMDAwNSwgLTc5LjM4MjU3OTk5OTk5OTk2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzkzMTNiNDVjMjYzOTQxNDdhYzZkZWE5ZGQ5NzE4ZWE2ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9iYjkxY2U2YzEyZTI0YjA4YTg4ZjEzMWY5MWI0MTI1ZiA9ICQoYDxkaXYgaWQ9Imh0bWxfYmI5MWNlNmMxMmUyNGIwOGE4OGYxMzFmOTFiNDEyNWYiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlJpY2htb25kLCBBZGVsYWlkZSwgS2luZyBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF85MzEzYjQ1YzI2Mzk0MTQ3YWM2ZGVhOWRkOTcxOGVhNi5zZXRDb250ZW50KGh0bWxfYmI5MWNlNmMxMmUyNGIwOGE4OGYxMzFmOTFiNDEyNWYpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzUyMTE2OGEyZmY5ZTQwYjY4NDdiM2JkN2YyMDlmODJiLmJpbmRQb3B1cChwb3B1cF85MzEzYjQ1YzI2Mzk0MTQ3YWM2ZGVhOWRkOTcxOGVhNikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNDE1MWE4NjgzOTc5NDk3ZjliZDQxNWI2ZGY3OGE4ZTUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDI4NTAwMDAwMDAwNywgLTc5LjM4MDc1OTk5OTk5OTk1XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzRhNDljMjY1YTYwMzQzMjZiNzQ4YzUzODc3Njk3MzFiID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8zZmZiNmEyZmYxYjY0NmE0ODI4NmYyZjk5NGM4ODJhNSA9ICQoYDxkaXYgaWQ9Imh0bWxfM2ZmYjZhMmZmMWI2NDZhNDgyODZmMmY5OTRjODgyYTUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhhcmJvdXJmcm9udCBFYXN0LCBVbmlvbiBTdGF0aW9uLCBUb3JvbnRvIElzbGFuZHMgQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNGE0OWMyNjVhNjAzNDMyNmI3NDhjNTM4Nzc2OTczMWIuc2V0Q29udGVudChodG1sXzNmZmI2YTJmZjFiNjQ2YTQ4Mjg2ZjJmOTk0Yzg4MmE1KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl80MTUxYTg2ODM5Nzk0OTdmOWJkNDE1YjZkZjc4YThlNS5iaW5kUG9wdXAocG9wdXBfNGE0OWMyNjVhNjAzNDMyNmI3NDhjNTM4Nzc2OTczMWIpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2IxMTM1MmM4Yzc5NjRiMWM5NjQ3ODhlZTNjNGFlMzJmID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ3MTAwMDAwMDAwMDgsIC03OS4zODE1Mjk5OTk5OTk5NF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8xN2MxMTQ0MDNiN2U0MDQyYjk2MTIwNDQ2MTI2MjhhYSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYmE2MWNmNTFhZjIyNGE3ZDk1M2JjZmJhYWJiMWRkN2IgPSAkKGA8ZGl2IGlkPSJodG1sX2JhNjFjZjUxYWYyMjRhN2Q5NTNiY2ZiYWFiYjFkZDdiIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5Ub3JvbnRvIERvbWluaW9uIENlbnRyZSwgRGVzaWduIEV4Y2hhbmdlIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzE3YzExNDQwM2I3ZTQwNDJiOTYxMjA0NDYxMjYyOGFhLnNldENvbnRlbnQoaHRtbF9iYTYxY2Y1MWFmMjI0YTdkOTUzYmNmYmFhYmIxZGQ3Yik7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfYjExMzUyYzhjNzk2NGIxYzk2NDc4OGVlM2M0YWUzMmYuYmluZFBvcHVwKHBvcHVwXzE3YzExNDQwM2I3ZTQwNDJiOTYxMjA0NDYxMjYyOGFhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mNWIyMDFjNzY0Y2I0NTAxOWYyZGQwZDYyNDQxZWE3NCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0ODQwMDAwMDAwMDA0LCAtNzkuMzc5MTM5OTk5OTk5OTVdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfMGVkMDEwY2U2MTZlNGE2MWJjMmFkYTI1MmU2YzdhZjAgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2I5OWRjMzA4Yzg1NzRmMzBiMzQzYWMxMDg2MjMxN2FkID0gJChgPGRpdiBpZD0iaHRtbF9iOTlkYzMwOGM4NTc0ZjMwYjM0M2FjMTA4NjIzMTdhZCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Q29tbWVyY2UgQ291cnQsIFZpY3RvcmlhIEhvdGVsIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzBlZDAxMGNlNjE2ZTRhNjFiYzJhZGEyNTJlNmM3YWYwLnNldENvbnRlbnQoaHRtbF9iOTlkYzMwOGM4NTc0ZjMwYjM0M2FjMTA4NjIzMTdhZCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfZjViMjAxYzc2NGNiNDUwMTlmMmRkMGQ2MjQ0MWVhNzQuYmluZFBvcHVwKHBvcHVwXzBlZDAxMGNlNjE2ZTRhNjFiYzJhZGEyNTJlNmM3YWYwKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lMWE3ZDIyYzhiYzY0NWM1ODkyZTE2OWJmN2U5OTBmZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY5NDc5MDAwMDAwMDA3LCAtNzkuNDE0Mzk5OTk5OTk5OTRdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiMwMGI1ZWIiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzAwYjVlYiIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNDUzODk1MDVlMjJhNGY4NmEzNmY0M2I5NDA2OWM0NmYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sX2YyMTE2ZDFlNWQyOTQwY2ViMTlhMDZhNzI1M2JkYjI0ID0gJChgPGRpdiBpZD0iaHRtbF9mMjExNmQxZTVkMjk0MGNlYjE5YTA2YTcyNTNiZGIyNCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+Rm9yZXN0IEhpbGwgTm9ydGggJmFtcDsgV2VzdCwgRm9yZXN0IEhpbGwgUm9hZCBQYXJrIENsdXN0ZXIgMi4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzQ1Mzg5NTA1ZTIyYTRmODZhMzZmNDNiOTQwNjljNDZmLnNldENvbnRlbnQoaHRtbF9mMjExNmQxZTVkMjk0MGNlYjE5YTA2YTcyNTNiZGIyNCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfZTFhN2QyMmM4YmM2NDVjNTg5MmUxNjliZjdlOTkwZmUuYmluZFBvcHVwKHBvcHVwXzQ1Mzg5NTA1ZTIyYTRmODZhMzZmNDNiOTQwNjljNDZmKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mODc2MjkzYWMyZjU0MzQzOWZiODgxZGQzMDRkN2JiMCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY3NDg0MDAwMDAwMDA3NCwgLTc5LjQwNDUxOTk5OTk5OTkzXSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2ZiMGM3ZjUxNDk1NzQwZTI5YzYzM2UxMGM4MDhhZTlhID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9iODMyY2Y0YjI1NjY0MzlhYmViZWJiNzQ1ZjU3OWI4YSA9ICQoYDxkaXYgaWQ9Imh0bWxfYjgzMmNmNGIyNTY2NDM5YWJlYmViYjc0NWY1NzliOGEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlRoZSBBbm5leCwgTm9ydGggTWlkdG93biwgWW9ya3ZpbGxlIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2ZiMGM3ZjUxNDk1NzQwZTI5YzYzM2UxMGM4MDhhZTlhLnNldENvbnRlbnQoaHRtbF9iODMyY2Y0YjI1NjY0MzlhYmViZWJiNzQ1ZjU3OWI4YSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfZjg3NjI5M2FjMmY1NDM0MzlmYjg4MWRkMzA0ZDdiYjAuYmluZFBvcHVwKHBvcHVwX2ZiMGM3ZjUxNDk1NzQwZTI5YzYzM2UxMGM4MDhhZTlhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mOGZiMjNiNjc3Mjc0NTY5ODEzMmQwZjgwZmE2ZWU1MSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY2MzExMDAwMDAwMDA3NCwgLTc5LjQwMTc5OTk5OTk5OTk4XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzAwYmVhZGFmZTc0MTQzYTdiYTVkOTQyYzJlNzBmYTcyID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9jNDAwMjY3MGU0ZjA0ZmY5OTBkMGRjMTgwYmY1ZDk3NSA9ICQoYDxkaXYgaWQ9Imh0bWxfYzQwMDI2NzBlNGYwNGZmOTkwZDBkYzE4MGJmNWQ5NzUiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPlVuaXZlcnNpdHkgb2YgVG9yb250bywgSGFyYm9yZCBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8wMGJlYWRhZmU3NDE0M2E3YmE1ZDk0MmMyZTcwZmE3Mi5zZXRDb250ZW50KGh0bWxfYzQwMDI2NzBlNGYwNGZmOTkwZDBkYzE4MGJmNWQ5NzUpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyX2Y4ZmIyM2I2NzcyNzQ1Njk4MTMyZDBmODBmYTZlZTUxLmJpbmRQb3B1cChwb3B1cF8wMGJlYWRhZmU3NDE0M2E3YmE1ZDk0MmMyZTcwZmE3MikKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDZhOWNlZDVmMDg0NGMwNjg1MDliMjZhY2NmMDU1YmQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NTM1MTAwMDAwMDAwNCwgLTc5LjM5NzIxOTk5OTk5OTk1XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwX2Y1ZmI0ZDRkYzk0MTRhYTA5ZGUzM2IxNGJjNzNhNjA3ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF82Y2ViOGJlYTg5YTk0MmM5ODcwMWQ1ZGRkYmI0MWY1MCA9ICQoYDxkaXYgaWQ9Imh0bWxfNmNlYjhiZWE4OWE5NDJjOTg3MDFkNWRkZGJiNDFmNTAiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPktlbnNpbmd0b24gTWFya2V0LCBDaGluYXRvd24sIEdyYW5nZSBQYXJrIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2Y1ZmI0ZDRkYzk0MTRhYTA5ZGUzM2IxNGJjNzNhNjA3LnNldENvbnRlbnQoaHRtbF82Y2ViOGJlYTg5YTk0MmM5ODcwMWQ1ZGRkYmI0MWY1MCk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfMDZhOWNlZDVmMDg0NGMwNjg1MDliMjZhY2NmMDU1YmQuYmluZFBvcHVwKHBvcHVwX2Y1ZmI0ZDRkYzk0MTRhYTA5ZGUzM2IxNGJjNzNhNjA3KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mOTBlNTIxNTY4MjY0Y2FkODQ2NTQ5NWI3NGJjOGE0ZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0MDgyMDAwMDAwMDA3NiwgLTc5LjM5ODE3OTk5OTk5OTk3XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzQ2YmY2YWI1MWE1ZTQyNjNhYzc5ZmFlZTkzMGJmMTI0ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF8yMjkyMWNjZTA3Mjk0MGMzOTE4NTE3MjNmYWNkZTNhOSA9ICQoYDxkaXYgaWQ9Imh0bWxfMjI5MjFjY2UwNzI5NDBjMzkxODUxNzIzZmFjZGUzYTkiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkNOIFRvd2VyLCBLaW5nIGFuZCBTcGFkaW5hLCBSYWlsd2F5IExhbmRzLCBIYXJib3VyZnJvbnQgV2VzdCwgQmF0aHVyc3QgUXVheSwgU291dGggTmlhZ2FyYSwgSXNsYW5kIGFpcnBvcnQgQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfNDZiZjZhYjUxYTVlNDI2M2FjNzlmYWVlOTMwYmYxMjQuc2V0Q29udGVudChodG1sXzIyOTIxY2NlMDcyOTQwYzM5MTg1MTcyM2ZhY2RlM2E5KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9mOTBlNTIxNTY4MjY0Y2FkODQ2NTQ5NWI3NGJjOGE0ZS5iaW5kUG9wdXAocG9wdXBfNDZiZjZhYjUxYTVlNDI2M2FjNzlmYWVlOTMwYmYxMjQpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2UwMjcwYWNlYjdmYTQ4NTVhOGViZDI0OTA2MGFhOTU0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4NjkwMDAwMDAwMDQ1LCAtNzkuMzg1NDM5OTk5OTk5OTZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfM2RjMmJhOWIxZDVmNDIwNDgxNWIxMGQ2ZDkzZDJjOTYgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzc0Y2VhNTQ2NTQ0MTRjOWRhMGYzODY2ZTViZGJjOTZiID0gJChgPGRpdiBpZD0iaHRtbF83NGNlYTU0NjU0NDE0YzlkYTBmMzg2NmU1YmRiYzk2YiIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+U3RuIEEgUE8gQm94ZXMgQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfM2RjMmJhOWIxZDVmNDIwNDgxNWIxMGQ2ZDkzZDJjOTYuc2V0Q29udGVudChodG1sXzc0Y2VhNTQ2NTQ0MTRjOWRhMGYzODY2ZTViZGJjOTZiKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl9lMDI3MGFjZWI3ZmE0ODU1YThlYmQyNDkwNjBhYTk1NC5iaW5kUG9wdXAocG9wdXBfM2RjMmJhOWIxZDVmNDIwNDgxNWIxMGQ2ZDkzZDJjOTYpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzczODQwYTg5ZmExZDQwZjY5MDc4MzlmZjBkMzhjOTczID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjQ4MjgwMDAwMDAwMDYsIC03OS4zODE0NTk5OTk5OTk5NV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF83NWNkZjdmZGVkZjA0ZGFhYTg3OTBlY2RlZDEyY2Y5OCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfN2UxNTMyYWNjNzk0NGI1OWJjZTYyOTUzNjIxODc4NDYgPSAkKGA8ZGl2IGlkPSJodG1sXzdlMTUzMmFjYzc5NDRiNTliY2U2Mjk1MzYyMTg3ODQ2IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5GaXJzdCBDYW5hZGlhbiBQbGFjZSwgVW5kZXJncm91bmQgY2l0eSBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF83NWNkZjdmZGVkZjA0ZGFhYTg3OTBlY2RlZDEyY2Y5OC5zZXRDb250ZW50KGh0bWxfN2UxNTMyYWNjNzk0NGI1OWJjZTYyOTUzNjIxODc4NDYpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzczODQwYTg5ZmExZDQwZjY5MDc4MzlmZjBkMzhjOTczLmJpbmRQb3B1cChwb3B1cF83NWNkZjdmZGVkZjA0ZGFhYTg3OTBlY2RlZDEyY2Y5OCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDU0ZjdiZjhlYzBlNDNhYTk0NDdmYWQ2NTc2M2M0ODUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42Njg2OTAwMDAwMDAwMjYsIC03OS40MjA3MDk5OTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8yZTRmMTUzZjllOGU0MDUzOTMyMTJmNDllZGVlOGI4NCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNTQ3ZjFjMjc5YzA2NDYxYmJjN2U0NmYwYjIyMmNmMjcgPSAkKGA8ZGl2IGlkPSJodG1sXzU0N2YxYzI3OWMwNjQ2MWJiYzdlNDZmMGIyMjJjZjI3IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5DaHJpc3RpZSBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF8yZTRmMTUzZjllOGU0MDUzOTMyMTJmNDllZGVlOGI4NC5zZXRDb250ZW50KGh0bWxfNTQ3ZjFjMjc5YzA2NDYxYmJjN2U0NmYwYjIyMmNmMjcpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzA1NGY3YmY4ZWMwZTQzYWE5NDQ3ZmFkNjU3NjNjNDg1LmJpbmRQb3B1cChwb3B1cF8yZTRmMTUzZjllOGU0MDUzOTMyMTJmNDllZGVlOGI4NCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOTY0ODQ1MGZlNDE0NGQ0OThiZmZkY2Y4MWVhY2EzZjcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NjUwNTAwMDAwMDAwNjUsIC03OS40Mzg5MDk5OTk5OTk5Nl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF85NmUwZTM0ZDA5M2M0ZDRhOTMzYWYwOTM4NzNjMTIxYyA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjJkNjJjMmYwZDRkNDJlMGIyY2FkMDg1ZDY0OWQwYTUgPSAkKGA8ZGl2IGlkPSJodG1sXzYyZDYyYzJmMGQ0ZDQyZTBiMmNhZDA4NWQ2NDlkMGE1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5EdWZmZXJpbiwgRG92ZXJjb3VydCBWaWxsYWdlIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzk2ZTBlMzRkMDkzYzRkNGE5MzNhZjA5Mzg3M2MxMjFjLnNldENvbnRlbnQoaHRtbF82MmQ2MmMyZjBkNGQ0MmUwYjJjYWQwODVkNjQ5ZDBhNSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfOTY0ODQ1MGZlNDE0NGQ0OThiZmZkY2Y4MWVhY2EzZjcuYmluZFBvcHVwKHBvcHVwXzk2ZTBlMzRkMDkzYzRkNGE5MzNhZjA5Mzg3M2MxMjFjKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84ZmY1NjNkMzUwZWY0NjljODljY2U3M2QwNmFlNTFjMCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0ODQ4MDAwMDAwMDA2LCAtNzkuNDE3NzM5OTk5OTk5OThdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfY2Y0NjFhOTc5MzgxNGExNTk3YTg4MDcxZDI4ZWQyOGEgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzhlMWY5ZTIyNDkxYTRjMzhiNzk4ZTFmMDQ3NmU2ZDI1ID0gJChgPGRpdiBpZD0iaHRtbF84ZTFmOWUyMjQ5MWE0YzM4Yjc5OGUxZjA0NzZlNmQyNSIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+TGl0dGxlIFBvcnR1Z2FsLCBUcmluaXR5IENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwX2NmNDYxYTk3OTM4MTRhMTU5N2E4ODA3MWQyOGVkMjhhLnNldENvbnRlbnQoaHRtbF84ZTFmOWUyMjQ5MWE0YzM4Yjc5OGUxZjA0NzZlNmQyNSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfOGZmNTYzZDM1MGVmNDY5Yzg5Y2NlNzNkMDZhZTUxYzAuYmluZFBvcHVwKHBvcHVwX2NmNDYxYTk3OTM4MTRhMTU5N2E4ODA3MWQyOGVkMjhhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mYjY4ZjA5MjU5ZTQ0ZmE5YjdiN2ZmYWU1M2E1MDIyMCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjYzOTQxMDAwMDAwMDA1NSwgLTc5LjQyNjc1OTk5OTk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzA4ZjVlOWM1OTAxODQ5NmQ5N2VkZGQ1YzZlYjZlZDlhID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9mNjdkYTYzODFlYzI0NmVjYjE0OTJlNDZmZDhkM2I2NyA9ICQoYDxkaXYgaWQ9Imh0bWxfZjY3ZGE2MzgxZWMyNDZlY2IxNDkyZTQ2ZmQ4ZDNiNjciIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkJyb2NrdG9uLCBQYXJrZGFsZSBWaWxsYWdlLCBFeGhpYml0aW9uIFBsYWNlIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzA4ZjVlOWM1OTAxODQ5NmQ5N2VkZGQ1YzZlYjZlZDlhLnNldENvbnRlbnQoaHRtbF9mNjdkYTYzODFlYzI0NmVjYjE0OTJlNDZmZDhkM2I2Nyk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfZmI2OGYwOTI1OWU0NGZhOWI3YjdmZmFlNTNhNTAyMjAuYmluZFBvcHVwKHBvcHVwXzA4ZjVlOWM1OTAxODQ5NmQ5N2VkZGQ1YzZlYjZlZDlhKQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85ODI3ZWFkMjM3YWY0YjYxOTQxNTI3MWYyODRiZjFmZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY1OTczMDAwMDAwMDAyNSwgLTc5LjQ2MjgwOTk5OTk5OTkzXSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICIjZmYwMDAwIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiNmZjAwMDAiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfOWIwMTQ3ZTdjMzAyNDE2ZWI3MjdlODAzOTdjOWMzNTEpOwogICAgICAgIAogICAgCiAgICAgICAgdmFyIHBvcHVwXzUyNmE4MDQ1OTg5NjQ0Njc4MTJhYTU4ZWY3YWFhOTQ2ID0gTC5wb3B1cCh7Im1heFdpZHRoIjogIjEwMCUifSk7CgogICAgICAgIAogICAgICAgICAgICB2YXIgaHRtbF9mZTAzM2E2YTE0YWQ0ODg5YWY1ZTgyZjk3MTIzNzJiYSA9ICQoYDxkaXYgaWQ9Imh0bWxfZmUwMzNhNmExNGFkNDg4OWFmNWU4MmY5NzEyMzcyYmEiIHN0eWxlPSJ3aWR0aDogMTAwLjAlOyBoZWlnaHQ6IDEwMC4wJTsiPkhpZ2ggUGFyaywgVGhlIEp1bmN0aW9uIFNvdXRoIENsdXN0ZXIgMC4wPC9kaXY+YClbMF07CiAgICAgICAgICAgIHBvcHVwXzUyNmE4MDQ1OTg5NjQ0Njc4MTJhYTU4ZWY3YWFhOTQ2LnNldENvbnRlbnQoaHRtbF9mZTAzM2E2YTE0YWQ0ODg5YWY1ZTgyZjk3MTIzNzJiYSk7CiAgICAgICAgCgogICAgICAgIGNpcmNsZV9tYXJrZXJfOTgyN2VhZDIzN2FmNGI2MTk0MTUyNzFmMjg0YmYxZmUuYmluZFBvcHVwKHBvcHVwXzUyNmE4MDQ1OTg5NjQ0Njc4MTJhYTU4ZWY3YWFhOTQ2KQogICAgICAgIDsKCiAgICAgICAgCiAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80ZDUxNzNlNmJhZTk0ODAxYTdhZGNiZjg0OTA4YThiNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWzQzLjY0Nzc3MDAwMDAwMDA0LCAtNzkuNDQ5ODg5OTk5OTk5OThdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogIiNmZjAwMDAiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiI2ZmMDAwMCIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF85YjAxNDdlN2MzMDI0MTZlYjcyN2U4MDM5N2M5YzM1MSk7CiAgICAgICAgCiAgICAKICAgICAgICB2YXIgcG9wdXBfNGI1MTZhMzdmNDA2NGUyMTljNjU4MjY3YzM0ZTY4OTUgPSBMLnBvcHVwKHsibWF4V2lkdGgiOiAiMTAwJSJ9KTsKCiAgICAgICAgCiAgICAgICAgICAgIHZhciBodG1sXzhhMTViNDM5Yzg5YTQxYjU4Y2VjNjRkNWMzYTg5ODk0ID0gJChgPGRpdiBpZD0iaHRtbF84YTE1YjQzOWM4OWE0MWI1OGNlYzY0ZDVjM2E4OTg5NCIgc3R5bGU9IndpZHRoOiAxMDAuMCU7IGhlaWdodDogMTAwLjAlOyI+UGFya2RhbGUsIFJvbmNlc3ZhbGxlcyBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80YjUxNmEzN2Y0MDY0ZTIxOWM2NTgyNjdjMzRlNjg5NS5zZXRDb250ZW50KGh0bWxfOGExNWI0MzljODlhNDFiNThjZWM2NGQ1YzNhODk4OTQpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzRkNTE3M2U2YmFlOTQ4MDFhN2FkY2JmODQ5MDhhOGI1LmJpbmRQb3B1cChwb3B1cF80YjUxNmEzN2Y0MDY0ZTIxOWM2NTgyNjdjMzRlNjg5NSkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMWE1YTM1ZjY4MDBmNGQwZjk0YThiYmNhMDhiNTEwYTMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDk4MjAwMDAwMDAwMzQsIC03OS40NzU0Nzk5OTk5OTk5NV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF8wMjMyMWU2NzBjODE0MWI5YjNlMWVhODdiNWQwYjA2YSA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfNjg5ZDc0MGQyZGYxNGViYWJjZGVmZTY3ZTVmZDU1OWQgPSAkKGA8ZGl2IGlkPSJodG1sXzY4OWQ3NDBkMmRmMTRlYmFiY2RlZmU2N2U1ZmQ1NTlkIiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5SdW5ueW1lZGUsIFN3YW5zZWEgQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfMDIzMjFlNjcwYzgxNDFiOWIzZTFlYTg3YjVkMGIwNmEuc2V0Q29udGVudChodG1sXzY4OWQ3NDBkMmRmMTRlYmFiY2RlZmU2N2U1ZmQ1NTlkKTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8xYTVhMzVmNjgwMGY0ZDBmOTRhOGJiY2EwOGI1MTBhMy5iaW5kUG9wdXAocG9wdXBfMDIzMjFlNjcwYzgxNDFiOWIzZTFlYTg3YjVkMGIwNmEpCiAgICAgICAgOwoKICAgICAgICAKICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzY0YjJiMmYwZDJhZTRjYmU5NGNhOGU0MjcxNTM1ZmE3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbNDMuNjYyNTMwMDAwMDAwMDYsIC03OS4zOTE4Nzk5OTk5OTk5Nl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF80NGFiMDBiZGM3MWM0NjkwOWY0NmIxMWRjZDA4YzlkNCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjUwOGU3ZTYxZmQ1NGM5ZWFhZTY2ZmZmODU3NWNjMTQgPSAkKGA8ZGl2IGlkPSJodG1sX2I1MDhlN2U2MWZkNTRjOWVhYWU2NmZmZjg1NzVjYzE0IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5RdWVlbiYjMzk7cyBQYXJrLCBPbnRhcmlvIFByb3ZpbmNpYWwgR292ZXJubWVudCBDbHVzdGVyIDAuMDwvZGl2PmApWzBdOwogICAgICAgICAgICBwb3B1cF80NGFiMDBiZGM3MWM0NjkwOWY0NmIxMWRjZDA4YzlkNC5zZXRDb250ZW50KGh0bWxfYjUwOGU3ZTYxZmQ1NGM5ZWFhZTY2ZmZmODU3NWNjMTQpOwogICAgICAgIAoKICAgICAgICBjaXJjbGVfbWFya2VyXzY0YjJiMmYwZDJhZTRjYmU5NGNhOGU0MjcxNTM1ZmE3LmJpbmRQb3B1cChwb3B1cF80NGFiMDBiZGM3MWM0NjkwOWY0NmIxMWRjZDA4YzlkNCkKICAgICAgICA7CgogICAgICAgIAogICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDdiNmUzMTA5Y2U5NGFiZGEyMmU2Y2Y2YTUyNmFmMzMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFs0My42NDg2OTAwMDAwMDAwNDUsIC03OS4zODU0Mzk5OTk5OTk5Nl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiI2ZmMDAwMCIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjZmYwMDAwIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzliMDE0N2U3YzMwMjQxNmViNzI3ZTgwMzk3YzljMzUxKTsKICAgICAgICAKICAgIAogICAgICAgIHZhciBwb3B1cF9jOWYwM2M1Zjc4Zjc0OGZhOTZmOWJmZTllYmM3Njk5OCA9IEwucG9wdXAoeyJtYXhXaWR0aCI6ICIxMDAlIn0pOwoKICAgICAgICAKICAgICAgICAgICAgdmFyIGh0bWxfYjA2NmM1YTAzNWYwNGI5YTg2MGVjYjJhY2I3MDM4NTUgPSAkKGA8ZGl2IGlkPSJodG1sX2IwNjZjNWEwMzVmMDRiOWE4NjBlY2IyYWNiNzAzODU1IiBzdHlsZT0id2lkdGg6IDEwMC4wJTsgaGVpZ2h0OiAxMDAuMCU7Ij5CdXNpbmVzcyByZXBseSBtYWlsIFByb2Nlc3NpbmcgQ2VudHJlLCBTb3V0aCBDZW50cmFsIExldHRlciBQcm9jZXNzaW5nIFBsYW50IFRvcm9udG8gQ2x1c3RlciAwLjA8L2Rpdj5gKVswXTsKICAgICAgICAgICAgcG9wdXBfYzlmMDNjNWY3OGY3NDhmYTk2ZjliZmU5ZWJjNzY5OTguc2V0Q29udGVudChodG1sX2IwNjZjNWEwMzVmMDRiOWE4NjBlY2IyYWNiNzAzODU1KTsKICAgICAgICAKCiAgICAgICAgY2lyY2xlX21hcmtlcl8wN2I2ZTMxMDljZTk0YWJkYTIyZTZjZjZhNTI2YWYzMy5iaW5kUG9wdXAocG9wdXBfYzlmMDNjNWY3OGY3NDhmYTk2ZjliZmU5ZWJjNzY5OTgpCiAgICAgICAgOwoKICAgICAgICAKICAgIAo8L3NjcmlwdD4= onload="this.contentDocument.open();this.contentDocument.write(atob(this.getAttribute('data-html')));this.contentDocument.close();" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python

```
