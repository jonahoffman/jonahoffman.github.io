---
layout: post
title: Data Science - 3D Visualization in ANTz 
date: 2020-12-28 11:53:32 -0500
--- 

This internship was an awesome experience and helped me to develop my data cleaning and analysis skills in Python by creating advanced 3D data visualizations in the form of hierarchal glyphs using ANTz. 

In order to create these glyphs they first had to be designed by hand, then a sample glyph would be created in the ANTz software. Using the .csv file for the sample glyph, I used Python to create as many glyphs as needed that represented a specific data set using size, shape, and color.

My favorite of these was done using a dataset from the Vera Institue of Justice to map the demographics of a county's incarcerated population compared to the general county populations. Below is a screenshot from that visualization:
![](/assets/img/blog/matri-design-glyphs.png)

Though the code to create the loops cannot be shared, I have included a python script I wrote to assign latitudes and longitudes to counties using their zipcodes and chrome webdrivers: 

```python
import numpy as np
import pandas as pd
import jinja2
import requests
import os
import selenium
import time
from time import sleep
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.common.exceptions import TimeoutException
from selenium.webdriver.common.by import By
import selenium.webdriver.support.ui as ui
import selenium.webdriver.support.expected_conditions as EC

import sys
!{sys.executable} -m pip install folium
!{sys.executable} -m pip install geopy

import folium

df = pd.read_csv('incarceration_trends_2017_complete.csv')

locations = df[['latitude', 'longitude']]
locationlist = locations.values.tolist()
print(len(locationlist))

df = df[df['latitude'] != 0]
df = df[df['longitude'] != 0]
df = df[df['longitude'] < 0]

min_lat=df['latitude'].min()
max_lat=df['latitude'].max()
min_lon=df['longitude'].min()
max_lon=df['longitude'].max()
print(str('min_lat = ')+ str(min_lat))
print(str('max_lat = ')+ str(max_lat))

from geopy.distance import geodesic 
import geopy.distance

upperleft = (max_lat, min_lon) 
lowerleft= (min_lat, min_lon) 
height = geodesic(upperleft, lowerleft).km
print(height)

left = (max_lat, min_lon)
right = (max_lat, max_lon)
width = geodesic(left, right).km
print(width)

ratio = width/height
print(str('ratio=') + str(ratio))

buffer_lat = .1*height
buffer_lon = .1*width
print(buffer_lat)
print(buffer_lon)

height_pixel =(1000/ratio)
print(height_pixel)

# Upper Limit Latitude
startUL = geopy.Point(max_lat, min_lon)

d_latul = geopy.distance.distance(kilometers=buffer_lat)

# Use the `destination` method with a bearing of 0 degrees (which is north)
# in order to go from point `start` 1 km to north.
final_ul = d_latul.destination(point=startUL, bearing=0)
map_maxlat = final_ul.latitude
print(str('upper limit lat: ') + str(map_maxlat))

# Lower Limit Latitude
startLL = geopy.Point(min_lat, min_lon)

d_latll = geopy.distance.distance(kilometers=buffer_lat)

final_ll = d_latll.destination(point=startLL, bearing=180)
map_minlat = final_ll.latitude
print(str('lower limit lat: ') + str(map_minlat))

# Left Limit Longitude
startminlon = geopy.Point(max_lat, min_lon)

# Define a general distance object, initialized with a distance of 1 km.
d_lonleft = geopy.distance.distance(kilometers=buffer_lon)

# Use the `destination` method with a bearing of 0 degrees (which is north)
# in order to go from point `start` 1 km to north.
final_leftlon = d_lonleft.destination(point=startminlon, bearing=270)
map_minlon = final_leftlon.longitude

# Right Limit Longitude
startmaxlon = geopy.Point(max_lat, max_lon)

# Define a general distance object, initialized with a distance of 1 km.
d_lonright = geopy.distance.distance(kilometers=buffer_lon)

# Use the `destination` method with a bearing of 0 degrees (which is north)
# in order to go from point `start` 1 km to north.
final_rightlon = d_lonright.destination(point=startmaxlon, bearing=90)
map_maxlon = final_rightlon.longitude

m = folium.Map(location=[33, -117], width=1500, height=2000, tiles= 'https://server.arcgisonline.com/ArcGIS/rest/services/NatGeo_World_Map/MapServer/tile/{z}/{y}/{x}', attr= 'Tiles &copy; Esri &mdash; National Geographic, Esri, DeLorme, NAVTEQ, UNEP-WCMC, USGS, NASA, ESA, METI, NRCAN, GEBCO, NOAA, iPC', zoom_max= 16)


# for point in range(0, len(locationlist)):
 #   folium.Marker(locationlist[point], tooltip=locationlist[point]).add_to(m)

kw = {
    'radius': 3,

m.save('zipcode.html')
options = webdriver.ChromeOptions()
options.add_argument('--ignore-certificate-errors')
options.add_argument('--ignore-ssl-errors')
options.add_experimental_option("excludeSwitches", ['enable-automation'])
driver = webdriver.Chrome(options=options)
driver.set_window_size(2100, 2400)  # choose a resolution
#driver.get('file:///Users/offma/Documents/Matri%Design/Viz%Projects/zipcode.html')
driver.get('file:///C:/Users/offma/Documents/Matri%20Design/Viz%20Projects/zipcode.html')
time.sleep(12)
driv
[-]

```


[Back to Projects](/#projects)