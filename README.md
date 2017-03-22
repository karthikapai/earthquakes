# Earthquakes
STAT 141B Exploratory Data Analysis Project


## Project Earthquake

### Group Members: Karthika Pai, Kathryn Chiang, An Qi Ma, Natalie Marcom

For our final STA 141B exploratory data science project, we have decided to focus on earthquakes. Though all four of us use significant datasets and analyze them in different ways, the crux of our datasets are from the [USGS Earthquake Database](https://earthquake.usgs.gov/earthquakes/feed/v1.0/csv.php). With the skills we have learned from this class - most specifically, csv file reading; using libraries such as Basemap, Pandas, Numpy, Matplotlib and basic statistics, we hope to answer several questions we have about earthquakes. Each section will be preceded by the question it tries to answer, in bold.

Each group member was in charge of one section:
1. Part 1 - Karthika Pai
2. Part 2 - Kathryn Chiang
3. Part 3 - An Qi Ma
4. Part 4 - Natalie Marcom


```python
import warnings
warnings.filterwarnings('ignore')

#import statements
import pandas as pd
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
matplotlib.style.use('ggplot')

import os

from mpl_toolkits.basemap import Basemap
%matplotlib inline
```

## Let's look at our dataset!

The dataset is a csv file that has been downloaded from the USGS Earthquake Database (shown above). This dataset represents significant earthquakes that have occured throughout the world from the years 1965 to 2016. A significant earthquake is one that has been determined by the USGS to meet the three following criteria:

1. mag_significance = magnitude * 100 * (magnitude / 6.5); 
2. pager_significance = (red) ? 2000 : (orange) ? 1000 : (yellow) ? 500 : 0; (PAGER is a USGS-internal measure)
3. dyfi_significance = min(num_responses, 1000) * max_cdi / 10; (Did you feel it - also known as dyfi - is a query that takes into account whether people perceived the earthquake or not. The higher the magnitude of the earthquake, the greater the dyfi significance)

significance = max(mag_significance, pager_significance) + dyfi_significance

Any event with a significance > 600 is considered a significant event and appears on the list.


```python
directory = os.path.join(".", "world_eq.csv") 
eq = pd.read_csv(directory)
```


```python
eq.head() #23412 total
total = len(eq)
```


```python
eq.dtypes
```




    Date                           object
    Time                           object
    Latitude                      float64
    Longitude                     float64
    Type                           object
    Depth                         float64
    Depth Error                   float64
    Depth Seismic Stations        float64
    Magnitude                     float64
    Magnitude Type                 object
    Magnitude Error               float64
    Magnitude Seismic Stations    float64
    Azimuthal Gap                 float64
    Horizontal Distance           float64
    Horizontal Error              float64
    Root Mean Square              float64
    ID                             object
    Source                         object
    Location Source                object
    Magnitude Source               object
    Status                         object
    dtype: object



This is certainly a large dataset! The file has records of over 23000 earthquakes (23000+ rows), the majority of whose magnitude is over 4.0 (precise statistics will be discussed later). It also has 21 features, such as latitude and longitude, the magnitude, depth and other features such as azimuthal gap and USGS-specific earthquake ID.

We don't need some features for our analysis, so let's include only the time, the data, the latitude and longitude, and the depth of the earthquake source to make our analysis simpler!


```python
simple = eq[["Date", "Time", "Latitude","Longitude","Magnitude", "Depth"]]
```


```python
simple.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/02/1965</td>
      <td>13:44:18</td>
      <td>19.246</td>
      <td>145.616</td>
      <td>6.0</td>
      <td>131.6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01/04/1965</td>
      <td>11:29:49</td>
      <td>1.863</td>
      <td>127.352</td>
      <td>5.8</td>
      <td>80.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01/05/1965</td>
      <td>18:05:58</td>
      <td>-20.579</td>
      <td>-173.972</td>
      <td>6.2</td>
      <td>20.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01/08/1965</td>
      <td>18:49:43</td>
      <td>-59.076</td>
      <td>-23.557</td>
      <td>5.8</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01/09/1965</td>
      <td>13:32:50</td>
      <td>11.938</td>
      <td>126.427</td>
      <td>5.8</td>
      <td>15.0</td>
    </tr>
  </tbody>
</table>
</div>



## What is a rough geographical distribution of our earthquake list? Are some areas more "cluttered" or concentrated than others?


```python
m = Basemap(projection="mill")
```


```python
x,y = m([longs for longs in simple["Longitude"]],
         [lats for lats in simple["Latitude"]])
fig = plt.figure(figsize=(20,20))
plt.title("Significant Earthquakes from 1965 - 2016")
m.scatter(x,y, s = 10, c = "maroon")
m.drawcoastlines()
m.drawmapboundary()
m.drawcountries()
m.fillcontinents(color='lightsteelblue',lake_color='skyblue')

plt.show()
```


![png](output_11_0.png)


It seems like earthquakes are distributed around naturally occuring fault lines in the earth's tectonic plates. Let's make the dots appear a bit larger in order to figure out which regions have the most concentration.


```python
fig = plt.figure(figsize=(20,20))
plt.title("Significant Earthquakes from 1965 - 2016")
m.scatter(x,y, s = 100, c = "maroon")
m.drawcoastlines()
m.drawmapboundary()
m.drawcountries()
m.fillcontinents(color='lightsteelblue',lake_color='skyblue')

plt.show()
```


![png](output_13_0.png)


It seems that majority of earthquakes are concentrated in the Indonesian, Sino Pacific and the Japanese area. Why is this so? Before we look at magnitude of earthquakes and how it relates to the geographical distribution of significant earthqauakes, let's try to answer this question. According to National Geographic, the Pacific Ring of Fire, technically called the Circum-Pacific belt, is the world's greatest earthquake belt, according to the U.S. Geological Survey (USGS), due to its series of fault lines stretching 25,000 miles (40,000 kilometers) from Chile in the Western Hemisphere through Japan and Southeast Asia. The magazine states that 
1. Roughly 90 percent of all the world's earthquakes, and 80 percent of the world's largest earthquakes, strike along the Ring of Fire
2. About 17 percent of the world's largest earthquakes and 5-6 percent of all quakes occur along the Alpide belt.

Are these statistics true? Let's find out!

I have the defined the Ring of Fire matrix to be the area of the world whose latitude is below 59.389 and above -45.783 and whose longitude is greater than -229.219 and below -65.391 degrees, (converted to about -70 to 120 on the Mercator projection). These values are obtained by drawing a rectangle that circumscribed the Ring of Fire area on the USGS interactive map.


```python
rof_lat = [-61.270, 56.632]
rof_long = [-70, 120]
```


```python
ringoffire = simple[((simple.Latitude < rof_lat[1]) & 
                    (simple.Latitude > rof_lat[0]) & 
                     ~((simple.Longitude < rof_long[1]) & 
                       (simple.Longitude > rof_long[0])))]
```


```python
x,y = m([longs for longs in ringoffire["Longitude"]],
         [lats for lats in ringoffire["Latitude"]])
fig2 = plt.figure(figsize=(20,20))
plt.title("Earthquakes in the Ring of Fire Area")
m.scatter(x,y, s = 15, c = "maroon")
m.drawcoastlines()
m.drawmapboundary()
m.drawcountries()
m.fillcontinents(color='lightsteelblue',lake_color='skyblue')

plt.show()
```


![png](output_18_0.png)



```python
ringoffire
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01/02/1965</td>
      <td>13:44:18</td>
      <td>19.2460</td>
      <td>145.6160</td>
      <td>6.0</td>
      <td>131.60</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01/04/1965</td>
      <td>11:29:49</td>
      <td>1.8630</td>
      <td>127.3520</td>
      <td>5.8</td>
      <td>80.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01/05/1965</td>
      <td>18:05:58</td>
      <td>-20.5790</td>
      <td>-173.9720</td>
      <td>6.2</td>
      <td>20.00</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01/09/1965</td>
      <td>13:32:50</td>
      <td>11.9380</td>
      <td>126.4270</td>
      <td>5.8</td>
      <td>15.00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>01/10/1965</td>
      <td>13:36:32</td>
      <td>-13.4050</td>
      <td>166.6290</td>
      <td>6.7</td>
      <td>35.00</td>
    </tr>
    <tr>
      <th>7</th>
      <td>01/15/1965</td>
      <td>23:17:42</td>
      <td>-13.3090</td>
      <td>166.2120</td>
      <td>6.0</td>
      <td>35.00</td>
    </tr>
    <tr>
      <th>9</th>
      <td>01/17/1965</td>
      <td>10:43:17</td>
      <td>-24.5630</td>
      <td>178.4870</td>
      <td>5.8</td>
      <td>565.00</td>
    </tr>
    <tr>
      <th>11</th>
      <td>01/24/1965</td>
      <td>00:11:17</td>
      <td>-2.6080</td>
      <td>125.9520</td>
      <td>8.2</td>
      <td>20.00</td>
    </tr>
    <tr>
      <th>12</th>
      <td>01/29/1965</td>
      <td>09:35:30</td>
      <td>54.6360</td>
      <td>161.7030</td>
      <td>5.5</td>
      <td>55.00</td>
    </tr>
    <tr>
      <th>13</th>
      <td>02/01/1965</td>
      <td>05:27:06</td>
      <td>-18.6970</td>
      <td>-177.8640</td>
      <td>5.6</td>
      <td>482.90</td>
    </tr>
    <tr>
      <th>15</th>
      <td>02/04/1965</td>
      <td>03:25:00</td>
      <td>-51.8400</td>
      <td>139.7410</td>
      <td>6.1</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>16</th>
      <td>02/04/1965</td>
      <td>05:01:22</td>
      <td>51.2510</td>
      <td>178.7150</td>
      <td>8.7</td>
      <td>30.30</td>
    </tr>
    <tr>
      <th>17</th>
      <td>02/04/1965</td>
      <td>06:04:59</td>
      <td>51.6390</td>
      <td>175.0550</td>
      <td>6.0</td>
      <td>30.00</td>
    </tr>
    <tr>
      <th>18</th>
      <td>02/04/1965</td>
      <td>06:37:06</td>
      <td>52.5280</td>
      <td>172.0070</td>
      <td>5.7</td>
      <td>25.00</td>
    </tr>
    <tr>
      <th>19</th>
      <td>02/04/1965</td>
      <td>06:39:32</td>
      <td>51.6260</td>
      <td>175.7460</td>
      <td>5.8</td>
      <td>25.00</td>
    </tr>
    <tr>
      <th>20</th>
      <td>02/04/1965</td>
      <td>07:11:23</td>
      <td>51.0370</td>
      <td>177.8480</td>
      <td>5.9</td>
      <td>25.00</td>
    </tr>
    <tr>
      <th>21</th>
      <td>02/04/1965</td>
      <td>07:14:59</td>
      <td>51.7300</td>
      <td>173.9750</td>
      <td>5.9</td>
      <td>20.00</td>
    </tr>
    <tr>
      <th>22</th>
      <td>02/04/1965</td>
      <td>07:23:12</td>
      <td>51.7750</td>
      <td>173.0580</td>
      <td>5.7</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>23</th>
      <td>02/04/1965</td>
      <td>07:43:43</td>
      <td>52.6110</td>
      <td>172.5880</td>
      <td>5.7</td>
      <td>24.00</td>
    </tr>
    <tr>
      <th>24</th>
      <td>02/04/1965</td>
      <td>08:06:17</td>
      <td>51.8310</td>
      <td>174.3680</td>
      <td>5.7</td>
      <td>31.80</td>
    </tr>
    <tr>
      <th>25</th>
      <td>02/04/1965</td>
      <td>08:33:41</td>
      <td>51.9480</td>
      <td>173.9690</td>
      <td>5.6</td>
      <td>20.00</td>
    </tr>
    <tr>
      <th>26</th>
      <td>02/04/1965</td>
      <td>08:40:44</td>
      <td>51.4430</td>
      <td>179.6050</td>
      <td>7.3</td>
      <td>30.00</td>
    </tr>
    <tr>
      <th>27</th>
      <td>02/04/1965</td>
      <td>12:06:08</td>
      <td>52.7730</td>
      <td>171.9740</td>
      <td>6.5</td>
      <td>30.00</td>
    </tr>
    <tr>
      <th>28</th>
      <td>02/04/1965</td>
      <td>12:50:59</td>
      <td>51.7720</td>
      <td>174.6960</td>
      <td>5.6</td>
      <td>20.00</td>
    </tr>
    <tr>
      <th>29</th>
      <td>02/04/1965</td>
      <td>14:18:29</td>
      <td>52.9750</td>
      <td>171.0910</td>
      <td>6.4</td>
      <td>25.00</td>
    </tr>
    <tr>
      <th>30</th>
      <td>02/04/1965</td>
      <td>15:51:25</td>
      <td>52.9900</td>
      <td>170.8740</td>
      <td>5.8</td>
      <td>25.00</td>
    </tr>
    <tr>
      <th>31</th>
      <td>02/04/1965</td>
      <td>18:34:12</td>
      <td>51.5360</td>
      <td>175.0450</td>
      <td>5.8</td>
      <td>25.00</td>
    </tr>
    <tr>
      <th>33</th>
      <td>02/04/1965</td>
      <td>22:30:03</td>
      <td>51.8120</td>
      <td>174.2060</td>
      <td>5.7</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>34</th>
      <td>02/05/1965</td>
      <td>06:39:50</td>
      <td>51.7620</td>
      <td>174.8410</td>
      <td>5.7</td>
      <td>25.00</td>
    </tr>
    <tr>
      <th>35</th>
      <td>02/05/1965</td>
      <td>09:32:11</td>
      <td>52.4380</td>
      <td>174.3210</td>
      <td>6.3</td>
      <td>39.50</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>23379</th>
      <td>12/10/2016</td>
      <td>02:45:40</td>
      <td>-10.8829</td>
      <td>161.2789</td>
      <td>5.8</td>
      <td>7.66</td>
    </tr>
    <tr>
      <th>23380</th>
      <td>12/10/2016</td>
      <td>16:24:35</td>
      <td>-5.6593</td>
      <td>154.4734</td>
      <td>6.0</td>
      <td>142.58</td>
    </tr>
    <tr>
      <th>23381</th>
      <td>12/11/2016</td>
      <td>14:33:13</td>
      <td>-9.1237</td>
      <td>-109.8492</td>
      <td>5.8</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>23382</th>
      <td>12/11/2016</td>
      <td>17:26:10</td>
      <td>-10.9640</td>
      <td>161.5723</td>
      <td>5.5</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>23383</th>
      <td>12/14/2016</td>
      <td>02:01:23</td>
      <td>21.2897</td>
      <td>144.4037</td>
      <td>6.0</td>
      <td>22.37</td>
    </tr>
    <tr>
      <th>23384</th>
      <td>12/14/2016</td>
      <td>21:14:56</td>
      <td>21.3697</td>
      <td>144.2175</td>
      <td>5.5</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>23385</th>
      <td>12/16/2016</td>
      <td>11:34:58</td>
      <td>14.0882</td>
      <td>-90.8691</td>
      <td>5.5</td>
      <td>71.26</td>
    </tr>
    <tr>
      <th>23386</th>
      <td>12/17/2016</td>
      <td>10:51:10</td>
      <td>-4.5049</td>
      <td>153.5216</td>
      <td>7.9</td>
      <td>94.54</td>
    </tr>
    <tr>
      <th>23387</th>
      <td>12/17/2016</td>
      <td>11:22:40</td>
      <td>-4.4244</td>
      <td>153.5419</td>
      <td>5.6</td>
      <td>83.36</td>
    </tr>
    <tr>
      <th>23388</th>
      <td>12/17/2016</td>
      <td>11:27:39</td>
      <td>-5.6497</td>
      <td>153.9975</td>
      <td>6.3</td>
      <td>26.50</td>
    </tr>
    <tr>
      <th>23389</th>
      <td>12/18/2016</td>
      <td>05:46:25</td>
      <td>-10.2137</td>
      <td>161.2177</td>
      <td>5.9</td>
      <td>37.39</td>
    </tr>
    <tr>
      <th>23390</th>
      <td>12/18/2016</td>
      <td>06:15:46</td>
      <td>-34.9886</td>
      <td>-107.8694</td>
      <td>5.5</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>23391</th>
      <td>12/18/2016</td>
      <td>06:39:42</td>
      <td>-6.3046</td>
      <td>154.3530</td>
      <td>5.9</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>23392</th>
      <td>12/18/2016</td>
      <td>09:47:05</td>
      <td>8.3489</td>
      <td>137.6672</td>
      <td>6.2</td>
      <td>12.43</td>
    </tr>
    <tr>
      <th>23393</th>
      <td>12/18/2016</td>
      <td>11:35:48</td>
      <td>-10.1904</td>
      <td>161.2187</td>
      <td>5.5</td>
      <td>57.52</td>
    </tr>
    <tr>
      <th>23394</th>
      <td>12/18/2016</td>
      <td>13:30:11</td>
      <td>-9.9640</td>
      <td>-70.9714</td>
      <td>6.4</td>
      <td>622.54</td>
    </tr>
    <tr>
      <th>23395</th>
      <td>12/20/2016</td>
      <td>04:21:29</td>
      <td>-10.1773</td>
      <td>161.2236</td>
      <td>6.4</td>
      <td>16.65</td>
    </tr>
    <tr>
      <th>23397</th>
      <td>12/20/2016</td>
      <td>12:33:14</td>
      <td>-10.1785</td>
      <td>160.9149</td>
      <td>6.0</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>23398</th>
      <td>12/20/2016</td>
      <td>20:07:53</td>
      <td>-10.1549</td>
      <td>160.7816</td>
      <td>5.5</td>
      <td>10.38</td>
    </tr>
    <tr>
      <th>23399</th>
      <td>12/21/2016</td>
      <td>00:17:15</td>
      <td>-7.5082</td>
      <td>127.9206</td>
      <td>6.7</td>
      <td>152.00</td>
    </tr>
    <tr>
      <th>23400</th>
      <td>12/21/2016</td>
      <td>16:43:57</td>
      <td>21.5036</td>
      <td>145.4172</td>
      <td>5.9</td>
      <td>12.05</td>
    </tr>
    <tr>
      <th>23401</th>
      <td>12/24/2016</td>
      <td>01:32:16</td>
      <td>-5.2453</td>
      <td>153.5754</td>
      <td>6.0</td>
      <td>35.00</td>
    </tr>
    <tr>
      <th>23402</th>
      <td>12/24/2016</td>
      <td>03:58:55</td>
      <td>-5.1460</td>
      <td>153.5166</td>
      <td>5.8</td>
      <td>30.00</td>
    </tr>
    <tr>
      <th>23403</th>
      <td>12/25/2016</td>
      <td>14:22:27</td>
      <td>-43.4029</td>
      <td>-73.9395</td>
      <td>7.6</td>
      <td>38.00</td>
    </tr>
    <tr>
      <th>23404</th>
      <td>12/25/2016</td>
      <td>14:32:13</td>
      <td>-43.4810</td>
      <td>-74.4771</td>
      <td>5.6</td>
      <td>14.93</td>
    </tr>
    <tr>
      <th>23406</th>
      <td>12/28/2016</td>
      <td>08:18:01</td>
      <td>38.3754</td>
      <td>-118.8977</td>
      <td>5.6</td>
      <td>10.80</td>
    </tr>
    <tr>
      <th>23407</th>
      <td>12/28/2016</td>
      <td>08:22:12</td>
      <td>38.3917</td>
      <td>-118.8941</td>
      <td>5.6</td>
      <td>12.30</td>
    </tr>
    <tr>
      <th>23408</th>
      <td>12/28/2016</td>
      <td>09:13:47</td>
      <td>38.3777</td>
      <td>-118.8957</td>
      <td>5.5</td>
      <td>8.80</td>
    </tr>
    <tr>
      <th>23409</th>
      <td>12/28/2016</td>
      <td>12:38:51</td>
      <td>36.9179</td>
      <td>140.4262</td>
      <td>5.9</td>
      <td>10.00</td>
    </tr>
    <tr>
      <th>23411</th>
      <td>12/30/2016</td>
      <td>20:08:28</td>
      <td>37.3973</td>
      <td>141.4103</td>
      <td>5.5</td>
      <td>11.94</td>
    </tr>
  </tbody>
</table>
<p>17596 rows × 6 columns</p>
</div>



There are 17596 earthquakes which are positioned solely in the ring of fire area. There were 23412 total large earthquakes in the entire dataset. So, frequency wise, about 75.1% of significant or largest earthquakes are in the Ring of Fire region. This is extremely close to the 80% figure cited in the National Geographic. 

## Magnitude Statistics

### What are some basic statistics (max, min, average etc) for the magnitudes of the entire dataset and the Ring of Fire earthquake subset?
### Which magnitudes occur the most frequently in both datasets?
### Is there some sort of pattern in the frequency of magnitudes?


```python
minimum = simple["Magnitude"].min()
maximum = simple["Magnitude"].max()
average = simple["Magnitude"].mean()

print("Minimum:", minimum)
print("Maximum:",maximum)
print("Mean",average)
```

    ('Minimum:', 5.5)
    ('Maximum:', 9.0999999999999996)
    ('Mean', 5.882530753460003)



```python
minimum = ringoffire["Magnitude"].min()
maximum = ringoffire["Magnitude"].max()
average = ringoffire["Magnitude"].mean()

print("Minimum:", minimum)
print("Maximum:",maximum)
print("Mean",average)
```

    ('Minimum:', 5.5)
    ('Maximum:', 9.0999999999999996)
    ('Mean', 5.887151057058525)


The minimum, maximum and average for both datasets are eerily close together! What does that mean? For one thing, the subset data (the Ring of Fire earthquakes) comprise almost 75% of the total data; this ensures that statistics for both datasets will be extremely similar. Secondly, and more importantly, the dataset contains only earthquakes that have more than 5.0 magnitude (significant ones). If the dataset included a list of all earthquakes, we would see that a concentration of the world's major earthquakes would be in the Ring of Fire area. We will do so later. 

In the meantime, let's continue to look at some simple statistics and correlations with magnitude.


```python
n, bins, patch = plt.hist(simple["Magnitude"], histtype = 'step', range=(5.5,9.5), bins = 10)
plt.xlabel("Earthquake Magnitudes")
plt.ylabel("Frequency")
plt.title("Frequency by Magnitude")
histo = pd.DataFrame()
for i in range(0, len(n)):
    mag = str(bins[i])+ "-"+str(bins[i+1])
    freq = n[i]
    percentage = round((n[i]/total) * 100, 4)
    histo = histo.append(pd.Series([mag, freq, percentage]), ignore_index=True)
    
histo.columns = ['Range of Magnitude', 'Frequency', 'Percentage']
histo
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Range of Magnitude</th>
      <th>Frequency</th>
      <th>Percentage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5.5-5.9</td>
      <td>14109.0</td>
      <td>60.2640</td>
    </tr>
    <tr>
      <th>1</th>
      <td>5.9-6.3</td>
      <td>5655.0</td>
      <td>24.1543</td>
    </tr>
    <tr>
      <th>2</th>
      <td>6.3-6.7</td>
      <td>2173.0</td>
      <td>9.2816</td>
    </tr>
    <tr>
      <th>3</th>
      <td>6.7-7.1</td>
      <td>905.0</td>
      <td>3.8655</td>
    </tr>
    <tr>
      <th>4</th>
      <td>7.1-7.5</td>
      <td>347.0</td>
      <td>1.4821</td>
    </tr>
    <tr>
      <th>5</th>
      <td>7.5-7.9</td>
      <td>162.0</td>
      <td>0.6920</td>
    </tr>
    <tr>
      <th>6</th>
      <td>7.9-8.3</td>
      <td>48.0</td>
      <td>0.2050</td>
    </tr>
    <tr>
      <th>7</th>
      <td>8.3-8.7</td>
      <td>9.0</td>
      <td>0.0384</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8.7-9.1</td>
      <td>2.0</td>
      <td>0.0085</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9.1-9.5</td>
      <td>2.0</td>
      <td>0.0085</td>
    </tr>
  </tbody>
</table>
</div>




![png](output_26_1.png)


It seems that 60% of significant earthquakes had a magnitude between 5.5 to 5.86, whereas less that 4% total scored between 7.0 and 9.1 on the Richter scale. 

An interesting patterns also occurs when we plot magnitudes vs frequency on a log scale.


```python
fig, ax = plt.subplots()
#ax.plot(histo.index, fit[0] * histo.index + fit[1], color='red')
ax.scatter(histo.index, histo['Frequency'])
plt.xticks(histo.index, bins, rotation='vertical')
plt.yscale('log', nonposy='clip')

plt.xlabel("Magnitude")
plt.ylabel("Frequency")
plt.title("Worldwide Earthquake Frequencies, Logarithmic Scale")
fig.show()
```


![png](output_28_0.png)


Now the earthquakes almost a straight line on the graph. This pattern is known as a power-law distribution: it turns out that for every increase of
one point in magnitude, an earthquake becomes about ten times less frequent. So, for example, magnitude 6 earthquakes occur ten times more frequently than magnitude 7's, and one hundred times more often than magnitude 8's. 

We can use this to relatively calculate the probability that an earthquake will hit a particular region, although it is impossible to know exactly when. For example, if we know that there were 15 earthquakes between 5.0 and 5.9 in a particular region in a period of 70 years, that works to about one earthquake in three years. Following this distribution above, we can "predict" that an earthquake measuring between 6.0 and 6.9 should
occur about once every thirty years in this region. 

# Is there any correlation between depth of the earthquake and magnitude of the earthquake? 

Earthquakes can occur anywhere between the Earth's surface and about 700 kilometers below the surface. For scientific purposes, an earthquake depth range of 0 - 700 km is divided into three zones: shallow, intermediate, and deep.


```python
shallow = len(simple[simple.Depth < 70]) #18660
intermediate = len(simple[(simple.Depth > 70) & (simple.Depth < 300)]) ##3390
deep = len(simple[simple.Depth > 300]) #1326

print str(round(shallow/float(total) * 100, 4)) + " percent of signficant earthquakes are shallow."
print str(round(intermediate/float(total) * 100, 4)) + " percent of signficant earthquakes are intermediate."
print str(round(deep/float(total) * 100, 4)) + " percent of signficant earthquakes are deep."
```

    79.7027 percent of signficant earthquakes are shallow.
    14.4798 percent of signficant earthquakes are intermediate.
    5.6638 percent of signficant earthquakes are deep.


This is very surprising! There was an assumption that deep earthquakes necessarily produce significant ones, but that is not true. 

What about the geographical distribution of deep earthquakes? I predict that deep earthquakes are primarily situated in the Ring of Fire. 


```python
deep_df = simple[simple.Depth > 300]
x,y = m([longs for longs in deep_df["Longitude"]],
         [lats for lats in deep_df["Latitude"]])
fig = plt.figure(figsize=(20,20))
plt.title("Geographical Distribution of Deep Earthquakes")
m.scatter(x,y, s = 60, c = "maroon")
m.drawcoastlines()
m.drawmapboundary()
m.drawcountries()
m.fillcontinents(color='lightsteelblue',lake_color='skyblue')

plt.show()
```


![png](output_34_0.png)


Deep earthquakes are primarily situated in the Ring of Fire area, with the exception of a few near the Italian Penninsula.


```python
plt.scatter(simple["Magnitude"],simple["Depth"])
plt.xlabel("Magnitude")
plt.ylabel("Depth (in meters)")
plt.title("Magnitude vs Depth")
plt.show()
```


![png](output_36_0.png)


This plot tells me that earthquakes with magnitudes 5.5 to roughly 6.5 can be found in a great range of depths, from 0 meters to 700 meters. However, the depth of larger earthquakes are bimodal - they originate from the surface or from deep underground.

Are they correlated at all? Doing a simple coefficient of correlations calculation says the answer is most likely no.


```python
np.corrcoef(simple["Magnitude"], simple["Depth"])
```




    array([[ 1.        ,  0.02345731],
           [ 0.02345731,  1.        ]])



## Time correlations

### Do some earthquakes occur more in some months than others?
### Do some years have more earthquakes than others?


```python
simple["Date"] = pd.to_datetime(simple["Date"])
simple["Month"] = simple['Date'].dt.month
simple["Year"] = simple['Date'].dt.year

freqbymonth = simple.groupby('Month').size()
freqbyyear = simple.groupby('Year').size()

fig, ax = plt.subplots(figsize = (20,10))
bar_positions = np.arange(12) + 0.5
months = ["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"]

k = plt.bar(np.arange(len(months)), freqbymonth)
plt.xticks(np.arange(len(months)), months)

plt.xlabel('Month')
plt.ylabel('Frequency')
plt.title('Earthquakes by Month')
 

def autolabel(rects):
    """
    Attach a text label above each bar displaying its height
    """
    for rect in rects:
        height = rect.get_height()
        ax.text(rect.get_x() + rect.get_width()/2., 1.05*height,
                '%d' % int(height),
                ha='center', va='bottom')
        
autolabel(k)
plt.show()
```


![png](output_40_0.png)


It seems that there is a uniform distribution of earthquake frequency along all 12 months.

Let's look at year.


```python
yearly_line = plt.plot([i for i in range(1965, 2017)], freqbyyear, color = 'steelblue')
plt.xlabel('Year')
plt.ylabel('Frequency')
plt.title('Frequencies of Signficant Earthquakes by Year 1965 - 2016')
```




    <matplotlib.text.Text at 0x11b1f04d0>




![png](output_42_1.png)


<h1>Earthquakes in the USA<h1/>
<h3>Part 2 By Kathryn Chiang</h3>


```python
import matplotlib.pyplot as plt
import matplotlib.cm
import numpy as np
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
from __future__ import division
from collections import Counter
from nltk.probability import FreqDist
from mpl_toolkits.basemap import Basemap
from matplotlib.patches import Polygon
```

<i>Those are some functions I use for this dataset. We will call the functions later in the project.</i>


```python
def title_time(df):
    """Input: dataframe
    Output: start time and end time of earthquake"""
    title = '%s through %s' % (str(df['time'][df['index']==min(df['index'])]).split()[1],str(df['time'][df['index']==max(df['index'])]).split()[1])
    return title

def get_marker_color(magnitude):
    """Input: magnitude
    Output: green for small earthquakes (<2), yellow for moderate
    earthquakes(<4), and red for significant earthquakes(>4)."""
    if magnitude < 2.0:
        return ('go')
    elif magnitude < 4.0:
        return ('yo')
    else:
        return ('ro')

def get_stat(statename):
    """Inpute: state name, ex: 'California'
    Output: earthquake information of that state"""
    ca = pd.DataFrame()
    for i in range(len(us)):
        if us['state'][i] == statename:
            ca = ca.append(us.loc[i])
    ca = ca.reset_index(drop=True)
    return ca

def map_mag(df,lllon,lllat,urlon,urlat, place):
    my_map = Basemap(projection='merc', lat_0=57, lon_0=-135,
                     resolution = 'h', area_thresh = 1000.0,
                     llcrnrlon=lllon, llcrnrlat=lllat,
                     urcrnrlon=urlon, urcrnrlat=urlat)

    my_map.drawcoastlines()
    my_map.drawcountries()
    my_map.fillcontinents(color='coral')
    my_map.drawmapboundary()
    my_map.drawstates()

    lats = df['latitude']
    lons = df['longitude']
    magnitudes = df['mag']
    min_marker_size = 2.5
    for lon, lat, mag in zip(lons, lats, magnitudes):
        x,y = my_map(lon, lat)
        msize = mag * min_marker_size
        marker_string = get_marker_color(mag)
        my_map.plot(x, y, marker_string, markersize=msize)
    title = 'Earthquake Magnitude in %s\n' % place
    title += title_time(df)
    plt.title(title)
    plt.show()

def get_co_data(df):
    """Input: dataframe;
    Output: freq of magnitude strength"""
    colors = []
    for i in range(len(df)):
        if get_marker_color(df['mag'][i]) == 'go':
            m = 'small'
        elif get_marker_color(df['mag'][i]) == 'yo':
            m = 'moderate'
        else:
            m = 'significant'
        colors.append(m)
    x = range(len(colors))
    f = Counter(colors)
    return f

def ratio(df):
    """Input: dataframe
    Output: ratio percentage for each magnitude strength"""
    co = get_co_data(df)
    ratio = list()
    for i in range(len(co)):
        r = co.values()[i]/sum(co.values())*100
        ratio.append(r)
    return ratio

states = ['Alabama','Alaska','Arizona','Arkansas','California','Colorado','Connecticut','Delaware','Florida','Georgia','Hawaii','Idaho', 'Illinois','Indiana','Iowa','Kansas','Kentucky','Louisiana','Maine' 'Maryland','Massachusetts','Michigan','Minnesota','Mississippi', 'Missouri','Montana','Nebraska','Nevada','New Hampshire','New Jersey','New Mexico','New York','North Carolina','North Dakota','Ohio','Oklahoma','Oregon','Pennsylvania','Rhode Island','South  Carolina','South Dakota','Tennessee','Texas','Utah','Vermont','Virginia','Washington','West Virginia','Wisconsin','Wyoming']
```

<h2>Let's look at our dataset!</h2>

<p>The dataset is a csv file that has been downloaded from the <a href = "https://earthquake.usgs.gov/earthquakes/feed/v1.0/csv.php">USGS Earthquake Database</a>. This dataset represents all the earthquakes that have occurred throughout the world from the month January to Febuary, 2017. This dataset includes 7660 earthquakes, but we will only focus on the data that located in the USA.</p>


```python
data_w = pd.read_csv('all_month.csv')
print len(data_w)
data_w.dtypes
```

    7660





    time                object
    latitude           float64
    longitude          float64
    depth              float64
    mag                float64
    magType             object
    nst                float64
    gap                float64
    dmin               float64
    rms                float64
    net                 object
    id                  object
    updated             object
    place               object
    type                object
    horizontalError    float64
    depthError         float64
    magError           float64
    magNst             float64
    status              object
    locationSource      object
    magSource           object
    dtype: object




```python
def create_us_file():
    state = ['Alabama','Alaska','Arizona','Arkansas','California','Colorado','Connecticut','Delaware','Florida','Georgia','Hawaii','Idaho', 'Illinois','Indiana','Iowa','Kansas','Kentucky','Louisiana','Maine' 'Maryland','Massachusetts','Michigan','Minnesota','Mississippi', 'Missouri','Montana','Nebraska','Nevada','New Hampshire','New Jersey','New Mexico','New York','North Carolina','North Dakota','Ohio','Oklahoma','Oregon','Pennsylvania','Rhode Island','South  Carolina','South Dakota','Tennessee','Texas','Utah','Vermont','Virginia','Washington','West Virginia','Wisconsin','Wyoming',"AL", "AK", "AZ", "AR", "CA", "CO", "CT", "DC", "DE", "FL", "GA", "HI", "ID", "IL", "IN", "IA", "KS", "KY", "LA", "ME", "MD", "MA", "MI", "MN", "MS", "MO", "MT", "NE", "NV", "NH", "NJ", "NM", "NY", "NC", "ND", "OH", "OK", "OR", "PA", "RI", "SC", "SD", "TN", "TX", "UT", "VT", "VA", "WA", "WV", "WI", "WY"]
    states_d = {'AK': 'Alaska','AL': 'Alabama','AR': 'Arkansas','AS': 'American Samoa','AZ': 'Arizona','CA': 'California','CO': 'Colorado','CT': 'Connecticut','DC': 'District of Columbia','DE': 'Delaware','FL': 'Florida','GA': 'Georgia','GU': 'Guam','HI': 'Hawaii','IA': 'Iowa','ID': 'Idaho','IL': 'Illinois','IN': 'Indiana','KS': 'Kansas','KY': 'Kentucky','LA': 'Louisiana','MA': 'Massachusetts','MD': 'Maryland','ME': 'Maine','MI': 'Michigan','MN': 'Minnesota','MO': 'Missouri','MP': 'Northern Mariana Islands','MS': 'Mississippi','MT': 'Montana','NA': 'National','NC': 'North Carolina','ND': 'North Dakota','NE': 'Nebraska','NH': 'New Hampshire','NJ': 'New Jersey','NM': 'New Mexico','NV': 'Nevada','NY': 'New York','OH': 'Ohio','OK': 'Oklahoma','OR': 'Oregon','PA': 'Pennsylvania','PR': 'Puerto Rico','RI': 'Rhode Island','SC': 'South Carolina','SD': 'South Dakota','TN': 'Tennessee','TX': 'Texas','UT': 'Utah','VA': 'Virginia','VI': 'Virgin Islands','VT': 'Vermont','WA': 'Washington','WI': 'Wisconsin','WV': 'West Virginia','WY': 'Wyoming'}
    usa = pd.DataFrame()
    for j in range(len(state)):
        for i in range(len(data_w)):
            if state[j] in data_w['place'][i].split(',')[-1]:
                data_w['state'] = state[j]
                usa = usa.append(data_w.loc[i])
    usa = usa.reset_index()
    for i in range(len(usa)):
        if len(usa['state'][i]) == 2:
            usa['state'][i] = states_d[usa['state'][i]]
    return usa
usa = create_us_file()
#usa.to_csv('us_earthquake.csv',mode = 'w',index = False)
```

<p>By making this dataset more accessible for the project, I extracted data that are located in the USA and added a new column "state" to show the states for each row. We will basicly use the columns (shown below) for this project.
</p>


```python
data = pd.read_csv('us_earthquake.csv')
us = data[["index","time", "latitude","longitude","mag","state","place","depth"]]
us.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>index</th>
      <th>time</th>
      <th>latitude</th>
      <th>longitude</th>
      <th>mag</th>
      <th>state</th>
      <th>place</th>
      <th>depth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>2017-02-16T19:41:22.795Z</td>
      <td>63.8717</td>
      <td>-150.3950</td>
      <td>1.3</td>
      <td>Alaska</td>
      <td>70km W of Healy, Alaska</td>
      <td>8.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>7</td>
      <td>2017-02-16T17:11:22.122Z</td>
      <td>62.6021</td>
      <td>-149.8518</td>
      <td>1.8</td>
      <td>Alaska</td>
      <td>33km NNE of Talkeetna, Alaska</td>
      <td>68.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>14</td>
      <td>2017-02-16T16:28:06.568Z</td>
      <td>61.4375</td>
      <td>-151.6854</td>
      <td>1.9</td>
      <td>Alaska</td>
      <td>85km NNW of Nikiski, Alaska</td>
      <td>83.8</td>
    </tr>
    <tr>
      <th>3</th>
      <td>19</td>
      <td>2017-02-16T15:27:17.594Z</td>
      <td>61.7097</td>
      <td>-149.6386</td>
      <td>1.6</td>
      <td>Alaska</td>
      <td>9km NNW of Meadow Lakes, Alaska</td>
      <td>30.5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20</td>
      <td>2017-02-16T15:23:09.053Z</td>
      <td>59.9683</td>
      <td>-147.0029</td>
      <td>2.0</td>
      <td>Alaska</td>
      <td>70km NNW of Middleton Island, Alaska</td>
      <td>16.5</td>
    </tr>
  </tbody>
</table>
</div>



<h2>What is a rough geographical distribution of our earthquake list? Are some areas more "cluttered" or concentrated than others?</h2>


```python
"""USA territories"""
map_mag(us, -172, 15, -65.25, 71,'USA')
```


![png](output_111_0.png)


<p> It seems that majority of earthquakes are concentrated in the West Coast of the United States, South of Alaska, and some in Nevada and Hawaii area. Why is this so? In the previous part "Earthquakse in the World", we discussed that roughly 90 percent of all earthquakes strike along the Ring of Fire, so does United States. The globel map above shows the magnitude of each earthquakes occured in the USA. There are 3 different catagories distingished by colors green, yellow, and red. Green color dots are for small earthquakes (magnitude less than 2), yellow color dots are for moderate earthquakes (magnitude less than 4), and red color dots are for significant earthquakes (magnitude greater than 4).</p>

<h2>Magnitude Statistics</h2>
<h3>What are some basic statistics (mas, min, average etc) for the magnitudes of the dataset?</h3>
<h3>Magnitude is a quantitative measure of the size of the earthquake at its source. The higher the magnitude the stronger the earthquake is. We divided the magnitude into 3 levels, small, moderate, and significant. Which magnitude strength appears the most often?</h3>
<h3>Which magnitude strength occur the most frequently in each states?</h3>


```python
minimum = us["mag"].min()
maximum = us["mag"].max()
average = us["mag"].mean()

print("Minimum:", minimum)
print("Maximum:",maximum)
print("Mean",average)
```

    ('Minimum:', -0.91000000000000003)
    ('Maximum:', 5.2999999999999998)
    ('Mean', 1.2374354862224841)


<p>The minimum magnitude in this dataset is -0.91. The maximum magnitude is 5.30. Average is 1.24 which means small earthquakes occurred the most frequently. Is this true for every states?</p>


```python
s = list()
m = list()
l = list()
for state in states:
    state_name = get_stat(state)
    co = get_co_data(state_name)
    vals = co['small'] #significant count value
    s.append(vals)
    valm = co['moderate'] 
    m.append(valm)
    vall = co['significant']
    l.append(vall)
sb = pd.DataFrame({'small':s, 'moderate':m, 'significant':l})
sb.plot(kind='bar', stacked=True, color = ['blue','yellow','red'])
plt.ylabel('Numbers of Earthquakes')
plt.title('Stacked Bar Plot for Magnitude Strength of United States')
plt.xticks(np.arange(len(states)), states, rotation = 90,size = 8)

plt.show()
```


![png](output_16_0.png)


<i>This is a stacked bar plot for the magnitude strength. However, let's break down the y-axis to visualize the data more clearly.</i>


```python
s = list()
m = list()
l = list()
for state in states:
    state_name = get_stat(state)
    co = get_co_data(state_name)
    vals = co['small'] #significant count value
    s.append(vals)
    valm = co['moderate'] 
    m.append(valm)
    vall = co['significant']
    l.append(vall)
df = pd.DataFrame({'small':s, 'moderate':m, 'significant':l})
f, axis = plt.subplots(2, 1, sharex=True)
df.plot(kind='bar', ax=axis[0],stacked=True, color = ['blue','yellow','red'])
df.plot(kind='bar', ax=axis[1],stacked=True, color = ['blue','yellow','red'])
plt.xticks(np.arange(len(states)), states, rotation = 90,size = 8)

axis[0].set_ylim(450, 2480)
axis[1].set_ylim(0, 100)
axis[1].legend().set_visible(False)

axis[0].spines['bottom'].set_visible(False)
axis[1].spines['top'].set_visible(False)
axis[0].xaxis.tick_top()
axis[0].tick_params(labeltop='off')
axis[1].xaxis.tick_bottom()
d = .015
kwargs = dict(transform=axis[0].transAxes, color='k', clip_on=False)
axis[0].plot((-d,+d),(-d,+d), **kwargs)
axis[0].plot((1-d,1+d),(-d,+d), **kwargs)
kwargs.update(transform=axis[1].transAxes)
axis[1].plot((-d,+d),(1-d,1+d), **kwargs)
axis[1].plot((1-d,1+d),(1-d,1+d), **kwargs)
plt.show()
```


![png](output_188_0.png)


<p>The stacked bar plot above shows the distribution of the magnitude strength for each states in the USA. We set the small earthquakes (red color) with magnitudes less than 2, moderate earthquakes (blue color) with magnitudes less than 4, and significant earthquakes (yellow color) with magnitudes greater than 4. If we look at magnitude strength by ratio, most of the states have the highest ratio on small earthquakes and the lowest ratio on significant earthquakes. However, Georgia has the highest ratio for significant earthquakes Oklahoma has the highest ratio for moderate earthquakes.</p>


```python
gr = get_stat('Georgia')
grv = get_co_data(gr)
print grv
print ratio(gr)
ok = get_stat('Oklahoma')
okv = get_co_data(ok)
print okv
print ratio(ok)
```

    Counter({'significant': 11})
    [100.0]
    Counter({'moderate': 60, 'small': 3})
    [4.761904761904762, 95.23809523809523]


<p>The function (above) shows that Georgia has 100% significant earthquakes, and Oklahoma has 95% for the moderate earthquakes. However, the dataset for Georgia and Oklahoma are too small that if we want a further analysis, we will have to include more dataset for it to be unbiased.</p>

<h2>What is the top 2 states in the USA that has the most earthquakes? </h2>


```python
c = us['state'].value_counts()
x = c.values
y = c.index
l = np.arange(len(c))

fig,ax = plt.subplots()
rects = ax.patches
plt.bar(l, x, color = 'pink')
plt.xticks(l, y, size = 9,rotation = 90)
plt.ylabel('Numbers of Earthquakes')
plt.xlabel('State Names')
plt.title('Numbers of Earthquakes by States Names')
labels = x
for rect, label in zip(rects, labels):
    height = rect.get_height()
    ax.text(rect.get_x() + rect.get_width()/2, height + 5, label, ha='center', va='bottom',fontweight='bold',rotation = 45)
plt.show()
```


![png](output_23_0.png)


<p>This is a histogram with numbers of earthquakes by state names. The numbers on top of each bar shows the exact amount earthquakes occured in that state. Most of the earthquakes occurs in Alaska and California, which has occured 2455 and 2441 times respectively during the month.
</p>

<h2>Alaska vs. California</h2>
<h3>What is a rough geographical distribution of the dataset for Alaska and California? Are some areas more "cluttered" or concentrated than others?</h3>
<h3>Is there any findings by comparing their magnitude strength?</h3>
<h3>How about comparing their magnitude strength ratio?</h3>


```python
ak = get_stat('Alaska')
ca = get_stat('California')
```


```python
"""Alaska"""
map_mag(ak, -172, 48, -126, 72, 'Alaska')
"""California"""
map_mag(ca, -125, 32, -114, 42, 'California')
```


![png](output_27_0.png)



![png](output_27_1.png)


<p>The earthquakes in Alaska seems to be more concentrated than the earthquakes in California that are spread out along the coast and along the east side of California. How about their magnitude?</p>


```python
akv = get_co_data(ak).values()
cav = get_co_data(ca).values()
twobar = pd.DataFrame({'Alaska':akv, 'California':cav})
twobar.plot(kind='bar',color = ['yellow','cornflowerblue'])
ind = np.arange(3)
plt.xticks(range(3),get_co_data(ak).keys(),rotation = 0)
plt.ylabel('Numbers of Earthquakes')
plt.xlabel('Strength of Earthquakes')
plt.title('Bar Plots for California vs Alaska Magnitudes')
for a,b in zip(ind, akv): 
    plt.text(a, b, str(b),fontweight='bold',va='bottom',ha='right',color = 'darkgoldenrod')
for a,b in zip(ind, cav): 
    plt.text(a, b, str(b),fontweight='bold',va='bottom',ha='left',color = 'darkblue')

plt.show()
```


![png](output_29_0.png)


<p>Comparing the barplots of magnitude strength for California and Alaska, we can see that the most earthquakes for both states are small earthquakes which has magnitude that are less than 2. However, Alaska has 379 less small earthquakes than the numbers of small earthquakes in California, but Alaska has 377 and 16 more moderate and significant earthquakes than the earthquakes in California. What does that mean? Let's look at their ratio.</p>


```python
rak = ratio(ak)
rca = ratio(ca)
ind = range(len(rak))
co = get_co_data(ak)
plt.plot(rak,'blue',label = 'Alaska')
plt.plot(rca,'red',label = 'California')
plt.ylabel('Ratio in Percentage (%)')
plt.xlabel('Earthquake Strength')
plt.title('Ratio of Alaska vs. California')
plt.xticks(range(3), co.keys())

for a,b in zip(ind, rak): 
    plt.text(a, b, str(round(b,1)) + '%',fontweight='bold',color = 'midnightblue')
for a,b in zip(ind, rca): 
    plt.text(a, b, str(round(b,1)) + '%',fontweight='bold',va='top', color = 'darkred')
plt.legend(loc = 'upper right')

plt.show()
```


![png](output_31_0.png)


I divided each values by its total numbers of earthquakes, and then times 100 to get the percentage. Around 22% more of the earthquakes in Alaska are moderate or significant earthquakes, comparing to California's earthquakes. Alaska tends to have more moderate and significant earthquakes than California.

<h2>Is there any correlation between depth of the earthquake and magnitude of the earthquake?</h2>

<p>In the previous part, we discussed that earthquakes with magnitudes 5.5 to 6.5 does not have an relationship with depth of the earthquake. However, we want to test it with a different dataset which include magnitudes range from -0.91 to 5.30. </p>


```python
plt.scatter(us['mag'],us['depth'])
plt.ylabel('depth')
plt.xlabel('magnitude')
plt.title('Magnitude vs Depth Scattered Plot in the USA')
plt.show()
```


![png](output_35_0.png)


<p>This plot tells me that earthquakes with magnitudes roughly 0 to 5.3 can be found in a range of depths, from 0 meters to around 270 meters. There seems to have correlation for the earthquakes with small magnitude. As the magnitude gets larger, depth range gets larger. Does that means there is a correlation between the two? We get a correlation coefficient of 0.33 which indicate a weak positive linear relationship, so there is a weak correlation between depth of the earthquake and magnitude of the earthquake. </p>


```python
df = us[['mag','depth']]
df.corr()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mag</th>
      <th>depth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>mag</th>
      <td>1.000000</td>
      <td>0.327762</td>
    </tr>
    <tr>
      <th>depth</th>
      <td>0.327762</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



<h2>Conclusion</h2>
<p>The majority of earthquakes in the USA are concentrated in the West Coast of the United States, South of Alaska, and some in Nevada and Hawaii area. Most of the earthquakes in each states has the highest ratio on small earthquakes and lowest ratio on significant earthquakes. Alaska and California are the two states that have the most earthquakes. By comparing Alaska with California, we found that Alaska tends to have more moderate and significant earthquakes than California. By comparing depth of the earthquake to the magnitude of the earthquake, we found that there is a weak correlation between depth of the earthquake and magnitude of the earthquake.</p>


## Part 3: The Relationship Between Earthquakes and Tsunamis

In this part of the project, I took a dataset of significant earthquakes from 1965-2012. The tsunami dataset contains a list of observations of tsunamis that have occured throughout history and comes from the NOAA website that contains many types of information related to tsunamis. Big earthquakes are said to cause tsunamis so I will be analyzing how earthquakes and tsunamis are related what how big of earthquakes usually cause tsunamis.


```python
import pandas as pd
import matplotlib
from matplotlib import pyplot as plt
import numpy as np
```


```python
plt.style.use('ggplot')
```


```python
# earthquakes dataframe
earthquakes = pd.read_csv('world_eq.csv')
earthquakes.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Type</th>
      <th>Depth</th>
      <th>Depth Error</th>
      <th>Depth Seismic Stations</th>
      <th>Magnitude</th>
      <th>Magnitude Type</th>
      <th>...</th>
      <th>Magnitude Seismic Stations</th>
      <th>Azimuthal Gap</th>
      <th>Horizontal Distance</th>
      <th>Horizontal Error</th>
      <th>Root Mean Square</th>
      <th>ID</th>
      <th>Source</th>
      <th>Location Source</th>
      <th>Magnitude Source</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1/2/1965</td>
      <td>13:44:18</td>
      <td>19.246</td>
      <td>145.616</td>
      <td>Earthquake</td>
      <td>131.6</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.0</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM860706</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1/4/1965</td>
      <td>11:29:49</td>
      <td>1.863</td>
      <td>127.352</td>
      <td>Earthquake</td>
      <td>80.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.8</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM860737</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1/5/1965</td>
      <td>18:05:58</td>
      <td>-20.579</td>
      <td>-173.972</td>
      <td>Earthquake</td>
      <td>20.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.2</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM860762</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1/8/1965</td>
      <td>18:49:43</td>
      <td>-59.076</td>
      <td>-23.557</td>
      <td>Earthquake</td>
      <td>15.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.8</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM860856</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1/9/1965</td>
      <td>13:32:50</td>
      <td>11.938</td>
      <td>126.427</td>
      <td>Earthquake</td>
      <td>15.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.8</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM860890</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1/10/1965</td>
      <td>13:36:32</td>
      <td>-13.405</td>
      <td>166.629</td>
      <td>Earthquake</td>
      <td>35.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.7</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM860922</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1/12/1965</td>
      <td>13:32:25</td>
      <td>27.357</td>
      <td>87.867</td>
      <td>Earthquake</td>
      <td>20.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.9</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM861007</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1/15/1965</td>
      <td>23:17:42</td>
      <td>-13.309</td>
      <td>166.212</td>
      <td>Earthquake</td>
      <td>35.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.0</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM861111</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1/16/1965</td>
      <td>11:32:37</td>
      <td>-56.452</td>
      <td>-27.043</td>
      <td>Earthquake</td>
      <td>95.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>6.0</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEMSUP861125</td>
      <td>ISCGEMSUP</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1/17/1965</td>
      <td>10:43:17</td>
      <td>-24.563</td>
      <td>178.487</td>
      <td>Earthquake</td>
      <td>565.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>5.8</td>
      <td>MW</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISCGEM861148</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>ISCGEM</td>
      <td>Automatic</td>
    </tr>
  </tbody>
</table>
<p>10 rows × 21 columns</p>
</div>




```python
len(earthquakes.index)
```




    23412




```python
earthquakes = earthquakes[["Date", "Time", "Latitude","Longitude","Magnitude", "Depth"]]
earthquakes.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1/2/1965</td>
      <td>13:44:18</td>
      <td>19.246</td>
      <td>145.616</td>
      <td>6.0</td>
      <td>131.6</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1/4/1965</td>
      <td>11:29:49</td>
      <td>1.863</td>
      <td>127.352</td>
      <td>5.8</td>
      <td>80.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1/5/1965</td>
      <td>18:05:58</td>
      <td>-20.579</td>
      <td>-173.972</td>
      <td>6.2</td>
      <td>20.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1/8/1965</td>
      <td>18:49:43</td>
      <td>-59.076</td>
      <td>-23.557</td>
      <td>5.8</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1/9/1965</td>
      <td>13:32:50</td>
      <td>11.938</td>
      <td>126.427</td>
      <td>5.8</td>
      <td>15.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
# tsunamis dataframe
tsunamis = pd.read_excel('tsevent.xlsx')
tsunamis.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>SECOND</th>
      <th>EVENT_VALIDITY</th>
      <th>CAUSE_CODE</th>
      <th>FOCAL_DEPTH</th>
      <th>...</th>
      <th>TOTAL_MISSING</th>
      <th>TOTAL_MISSING_DESCRIPTION</th>
      <th>TOTAL_INJURIES</th>
      <th>TOTAL_INJURIES_DESCRIPTION</th>
      <th>TOTAL_DAMAGE_MILLIONS_DOLLARS</th>
      <th>TOTAL_DAMAGE_DESCRIPTION</th>
      <th>TOTAL_HOUSES_DESTROYED</th>
      <th>TOTAL_HOUSES_DESTROYED_DESCRIPTION</th>
      <th>TOTAL_HOUSES_DAMAGED</th>
      <th>TOTAL_HOUSES_DAMAGED_DESCRIPTION</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>-2000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>-1610</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4.0</td>
      <td>6.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>-1365</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5</td>
      <td>-1300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6</td>
      <td>-760</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2.0</td>
      <td>0.0</td>
      <td>NaN</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 46 columns</p>
</div>




```python
from mpl_toolkits.basemap import Basemap
from matplotlib.colors import rgb2hex
from matplotlib.patches import Polygon
```


```python
for i in range(0, len(tsunamis.columns.values)):
    tsunamis.columns.values[i] = str(tsunamis.columns.values[i])
```


```python
# delete unnecessary columns
tsunamis.drop(tsunamis.columns[[range(16,46)]], inplace = True, axis = 1)
tsunamis = tsunamis[["ID", "YEAR", "MONTH", "DAY", "HOUR", "MINUTE", "COUNTRY", "STATE", "LOCATION_NAME", "LATITUDE", "LONGITUDE"]]
tsunamis.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>-2000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SYRIA</td>
      <td>NaN</td>
      <td>SYRIAN COASTS</td>
      <td>35.683</td>
      <td>35.80</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>-1610</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>GREECE</td>
      <td>NaN</td>
      <td>THERA ISLAND (SANTORINI)</td>
      <td>36.400</td>
      <td>25.40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>-1365</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SYRIA</td>
      <td>NaN</td>
      <td>SYRIAN COASTS</td>
      <td>35.683</td>
      <td>35.80</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5</td>
      <td>-1300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>TURKEY</td>
      <td>NaN</td>
      <td>IONIAN COASTS, TROAD</td>
      <td>39.960</td>
      <td>26.24</td>
    </tr>
    <tr>
      <th>4</th>
      <td>6</td>
      <td>-760</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>ISRAEL</td>
      <td>NaN</td>
      <td>ISRAEL AND LEBANON COASTS</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



I felt that some of these variables in the tsunami datasets, with most of them being the number of destructions, injured, and damages were unnecessary in this part of the project so I deleted those variables from the dataset.


```python
# Drop N/A lon/lat values for tsunami
# I filtered with longitude because if longitude has N/A, corresponding latitude also has it
tsu = tsunamis.loc[np.isnan(tsunamis['LONGITUDE']) == False]
tsu.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>-2000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SYRIA</td>
      <td>NaN</td>
      <td>SYRIAN COASTS</td>
      <td>35.683</td>
      <td>35.80</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>-1610</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>GREECE</td>
      <td>NaN</td>
      <td>THERA ISLAND (SANTORINI)</td>
      <td>36.400</td>
      <td>25.40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4</td>
      <td>-1365</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>SYRIA</td>
      <td>NaN</td>
      <td>SYRIAN COASTS</td>
      <td>35.683</td>
      <td>35.80</td>
    </tr>
    <tr>
      <th>3</th>
      <td>5</td>
      <td>-1300</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>TURKEY</td>
      <td>NaN</td>
      <td>IONIAN COASTS, TROAD</td>
      <td>39.960</td>
      <td>26.24</td>
    </tr>
    <tr>
      <th>5</th>
      <td>7</td>
      <td>-590</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>LEBANON</td>
      <td>NaN</td>
      <td>LEBANON COASTS</td>
      <td>33.270</td>
      <td>35.22</td>
    </tr>
  </tbody>
</table>
</div>



I noticed that my tsunami dataset had some N/A values for some observations so I dropped those observations or I would not have been able to plot those observations on a new map.


```python
recenttsu = tsu.loc[tsunamis['YEAR'] > 1964]
recenttsu.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2026</th>
      <td>1963</td>
      <td>1965</td>
      <td>1.0</td>
      <td>24.0</td>
      <td>0.0</td>
      <td>11.0</td>
      <td>INDONESIA</td>
      <td>NaN</td>
      <td>SANANA ISLAND</td>
      <td>-2.400</td>
      <td>126.100</td>
    </tr>
    <tr>
      <th>2027</th>
      <td>1964</td>
      <td>1965</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>5.0</td>
      <td>1.0</td>
      <td>USA</td>
      <td>AK</td>
      <td>RAT ISLANDS, ALEUTIAN ISLANDS, AK</td>
      <td>51.290</td>
      <td>178.550</td>
    </tr>
    <tr>
      <th>2028</th>
      <td>5470</td>
      <td>1965</td>
      <td>2.0</td>
      <td>19.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CHILE</td>
      <td>NaN</td>
      <td>SOUTHERN CHILE</td>
      <td>-41.755</td>
      <td>-72.396</td>
    </tr>
    <tr>
      <th>2029</th>
      <td>1965</td>
      <td>1965</td>
      <td>2.0</td>
      <td>23.0</td>
      <td>22.0</td>
      <td>11.0</td>
      <td>CHILE</td>
      <td>NaN</td>
      <td>NORTHERN CHILE</td>
      <td>-25.670</td>
      <td>-70.630</td>
    </tr>
    <tr>
      <th>2030</th>
      <td>3042</td>
      <td>1965</td>
      <td>3.0</td>
      <td>9.0</td>
      <td>17.0</td>
      <td>57.0</td>
      <td>GREECE</td>
      <td>NaN</td>
      <td>AEGEAN SEA</td>
      <td>39.400</td>
      <td>24.000</td>
    </tr>
  </tbody>
</table>
</div>




```python
len(recenttsu.index)
```




    546



The dataset above is a list of tsunamis that happened from 1965-2017 which corresponds with the timeframe of the earthquakes dataset.

## What is the geographical distribution of the earthquakes and tsunamis list and how much do they overlap?


```python
# draw world map

plt.figure(figsize=(15,10))
displaymap = Basemap(llcrnrlon=-180,llcrnrlat=-90,urcrnrlon=180,urcrnrlat=90)
```


```python
displaymap.drawmapboundary()
displaymap.drawcountries()
displaymap.drawcoastlines()
```

    C:\Users\Apus\Anaconda2\lib\site-packages\mpl_toolkits\basemap\__init__.py:1623: MatplotlibDeprecationWarning: The get_axis_bgcolor function was deprecated in version 2.0. Use get_facecolor instead.
      fill_color = ax.get_axis_bgcolor()





    <matplotlib.collections.LineCollection at 0xc4725c0>




```python
# Convert longitudes and latitudes to list of floats
longitude = earthquakes[['Longitude']].values.tolist()
for i in range(0, len(longitude)):
    longitude[i] = float(longitude[i][0])
latitude = earthquakes[['Latitude']].values.tolist()
for i in range(0, len(latitude)):
    latitude[i] = float(latitude[i][0])
tlongitude = recenttsu[[u'LONGITUDE']].values.tolist()
for i in range(0, len(tlongitude)):
    tlongitude[i] = float(tlongitude[i][0])
tlatitude = recenttsu[[u'LATITUDE']].values.tolist()
for i in range(0, len(tlatitude)):
    tlatitude[i] = float(tlatitude[i][0])
```


```python
lons,lats = displaymap(longitude, latitude)
tlons, tlats = displaymap(tlongitude, tlatitude)
displaymap.plot(lons, lats, 'bo', color = "blue")
displaymap.plot(tlons, tlats, 'bo', color = "red")
```

    C:\Users\Apus\Anaconda2\lib\site-packages\mpl_toolkits\basemap\__init__.py:3260: MatplotlibDeprecationWarning: The ishold function was deprecated in version 2.0.
      b = ax.ishold()
    C:\Users\Apus\Anaconda2\lib\site-packages\mpl_toolkits\basemap\__init__.py:3269: MatplotlibDeprecationWarning: axes.hold is deprecated.
        See the API Changes document (http://matplotlib.org/api/api_changes.html)
        for more details.
      ax.hold(b)





    [<matplotlib.lines.Line2D at 0xcbd1c50>]




```python
plt.title("Earthquakes and Tsunamis around the World from `1965-2017")
plt.show()
```


![png](output_22_0.png)


First, I converted all the observations for longitude and latitude in both sets from strings to floats. Then I plotted a map and all the known points for the earthquakes dataset and all the known points for the tsunami datasets. It seems that a lot of the points both overlap somewhere in the North American region and in the East Asian region and in the area known as the Ring of Fire where a large number of earthquakes and volcanic activity occur. It also looks like more tsunamis have occured in the Europe region rather than earthquakes.


```python
dates = earthquakes[['Date']].values.tolist()
years = []
months = []
days = []
for i in range(0, len(dates)):
    dates[i] = dates[i][0].split("/")
    try:
        years.append(dates[i][2])
    except IndexError:
        years.append('NaN')
    try:
        months.append(dates[i][0])
    except IndexError:
        months.append('NaN')
    try:
        days.append(dates[i][1])
    except IndexError:
        days.append('NaN')
```


```python
idlist = []
for i in range(0, len(earthquakes.index)):
    idlist.append(i)
```


```python
earthquakes['Year'] = years
earthquakes['Month'] = months
earthquakes['Days'] = days
earthquakes['ID'] = idlist
```


```python
earthquakes.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1/2/1965</td>
      <td>13:44:18</td>
      <td>19.246</td>
      <td>145.616</td>
      <td>6.0</td>
      <td>131.6</td>
      <td>1965</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1/4/1965</td>
      <td>11:29:49</td>
      <td>1.863</td>
      <td>127.352</td>
      <td>5.8</td>
      <td>80.0</td>
      <td>1965</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1/5/1965</td>
      <td>18:05:58</td>
      <td>-20.579</td>
      <td>-173.972</td>
      <td>6.2</td>
      <td>20.0</td>
      <td>1965</td>
      <td>1</td>
      <td>5</td>
      <td>2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1/8/1965</td>
      <td>18:49:43</td>
      <td>-59.076</td>
      <td>-23.557</td>
      <td>5.8</td>
      <td>15.0</td>
      <td>1965</td>
      <td>1</td>
      <td>8</td>
      <td>3</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1/9/1965</td>
      <td>13:32:50</td>
      <td>11.938</td>
      <td>126.427</td>
      <td>5.8</td>
      <td>15.0</td>
      <td>1965</td>
      <td>1</td>
      <td>9</td>
      <td>4</td>
    </tr>
  </tbody>
</table>
</div>



I split the dates into days, months, and years and added those rows to the dataset so I can analyze the dataset more flexibly. I also added IDs to each observation in order to remember specific ones.

## How often do earthquakes cause tsunamis? How much of the tsunamis in the dataset are caused by earthquakes?

I am interested in seeing how many earthquakes cause tsunamis in each year and their magnitude so I will pick two random years and analyze the earthquakes and tsunamis in those years.


```python
float(len(recenttsu.index))/float(len(earthquakes.index))
```




    0.023321373654536137



Earthquakes are sometimes said to cause tsunamis and based on this, about 2.3% of earthquakes cause tsunamis.


```python
eq2012 = earthquakes.loc[(earthquakes['Year'] == '2012')]
tsu2012 = tsu.loc[tsu[u'YEAR'] == 2012]
```


```python
tsu2012
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2515</th>
      <td>5442</td>
      <td>2012</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>13.0</td>
      <td>34.0</td>
      <td>VANUATU</td>
      <td>NaN</td>
      <td>VANUATU ISLANDS</td>
      <td>-17.827</td>
      <td>167.133</td>
    </tr>
    <tr>
      <th>2516</th>
      <td>5446</td>
      <td>2012</td>
      <td>3.0</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>8.0</td>
      <td>JAPAN</td>
      <td>NaN</td>
      <td>HOKKAIDO ISLAND</td>
      <td>40.887</td>
      <td>144.944</td>
    </tr>
    <tr>
      <th>2517</th>
      <td>5447</td>
      <td>2012</td>
      <td>3.0</td>
      <td>20.0</td>
      <td>18.0</td>
      <td>2.0</td>
      <td>MEXICO</td>
      <td>NaN</td>
      <td>S. MEXICO</td>
      <td>16.493</td>
      <td>-98.231</td>
    </tr>
    <tr>
      <th>2518</th>
      <td>5449</td>
      <td>2012</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>8.0</td>
      <td>38.0</td>
      <td>INDONESIA</td>
      <td>NaN</td>
      <td>OFF W. COAST OF N SUMATRA</td>
      <td>2.327</td>
      <td>93.063</td>
    </tr>
    <tr>
      <th>2519</th>
      <td>5450</td>
      <td>2012</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>10.0</td>
      <td>43.0</td>
      <td>INDONESIA</td>
      <td>NaN</td>
      <td>OFF W. COAST OF N SUMATRA</td>
      <td>0.802</td>
      <td>92.463</td>
    </tr>
    <tr>
      <th>2520</th>
      <td>5451</td>
      <td>2012</td>
      <td>4.0</td>
      <td>14.0</td>
      <td>22.0</td>
      <td>5.0</td>
      <td>VANUATU</td>
      <td>NaN</td>
      <td>VANUATU ISLANDS</td>
      <td>-18.972</td>
      <td>168.741</td>
    </tr>
    <tr>
      <th>2521</th>
      <td>5460</td>
      <td>2012</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>GREENLAND</td>
      <td>NaN</td>
      <td>ILULISSAT ICEFJORD</td>
      <td>69.200</td>
      <td>-51.300</td>
    </tr>
    <tr>
      <th>2522</th>
      <td>5462</td>
      <td>2012</td>
      <td>8.0</td>
      <td>27.0</td>
      <td>4.0</td>
      <td>37.0</td>
      <td>NICARAGUA</td>
      <td>NaN</td>
      <td>OFF THE COAST</td>
      <td>12.139</td>
      <td>-88.590</td>
    </tr>
    <tr>
      <th>2523</th>
      <td>5463</td>
      <td>2012</td>
      <td>8.0</td>
      <td>31.0</td>
      <td>12.0</td>
      <td>47.0</td>
      <td>PHILIPPINES</td>
      <td>NaN</td>
      <td>PHILIPPINE ISLANDS</td>
      <td>10.811</td>
      <td>126.638</td>
    </tr>
    <tr>
      <th>2524</th>
      <td>5464</td>
      <td>2012</td>
      <td>9.0</td>
      <td>5.0</td>
      <td>14.0</td>
      <td>42.0</td>
      <td>COSTA RICA</td>
      <td>NaN</td>
      <td>COSTA RICA</td>
      <td>10.085</td>
      <td>-85.315</td>
    </tr>
    <tr>
      <th>2525</th>
      <td>5467</td>
      <td>2012</td>
      <td>10.0</td>
      <td>28.0</td>
      <td>3.0</td>
      <td>4.0</td>
      <td>CANADA</td>
      <td>BC</td>
      <td>BRITISH COLUMBIA</td>
      <td>52.788</td>
      <td>-132.101</td>
    </tr>
    <tr>
      <th>2526</th>
      <td>5468</td>
      <td>2012</td>
      <td>11.0</td>
      <td>7.0</td>
      <td>16.0</td>
      <td>35.0</td>
      <td>GUATEMALA</td>
      <td>NaN</td>
      <td>GUATEMALA</td>
      <td>13.988</td>
      <td>-91.895</td>
    </tr>
    <tr>
      <th>2527</th>
      <td>5469</td>
      <td>2012</td>
      <td>12.0</td>
      <td>7.0</td>
      <td>8.0</td>
      <td>18.0</td>
      <td>JAPAN</td>
      <td>NaN</td>
      <td>OFF EAST COAST OF HONSHU ISLAND</td>
      <td>37.890</td>
      <td>143.949</td>
    </tr>
    <tr>
      <th>2528</th>
      <td>5471</td>
      <td>2012</td>
      <td>12.0</td>
      <td>28.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CHINA</td>
      <td>NaN</td>
      <td>ZHAOJUN BRIDGE, HUBEI PROVINCE</td>
      <td>31.256</td>
      <td>110.733</td>
    </tr>
  </tbody>
</table>
</div>




```python
print len(tsu2012), len(eq2012)
```

    14 445


In the year 2012, it looks like there is 1 tsunami that occured in February, 2 in March, 3 in April, 1 in July, 2 in August, 1 in September, 1 in October, 1 in Novemer, and 2 in December with a total of 14 tsunamis. There are 445 earthquakes total in the year 2012.


```python
tsu2012.loc[tsu2012[u'MONTH'] == 2]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2515</th>
      <td>5442</td>
      <td>2012</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>13.0</td>
      <td>34.0</td>
      <td>VANUATU</td>
      <td>NaN</td>
      <td>VANUATU ISLANDS</td>
      <td>-17.827</td>
      <td>167.133</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '2') & (eq2012['Days'] == '2')]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21142</th>
      <td>2/2/2012</td>
      <td>6:46:30</td>
      <td>-6.563</td>
      <td>149.774</td>
      <td>5.6</td>
      <td>51.3</td>
      <td>2012</td>
      <td>2</td>
      <td>2</td>
      <td>21142</td>
    </tr>
    <tr>
      <th>21143</th>
      <td>2/2/2012</td>
      <td>9:32:17</td>
      <td>-6.586</td>
      <td>149.718</td>
      <td>5.6</td>
      <td>38.6</td>
      <td>2012</td>
      <td>2</td>
      <td>2</td>
      <td>21143</td>
    </tr>
    <tr>
      <th>21144</th>
      <td>2/2/2012</td>
      <td>13:34:41</td>
      <td>-17.827</td>
      <td>167.133</td>
      <td>7.1</td>
      <td>23.0</td>
      <td>2012</td>
      <td>2</td>
      <td>2</td>
      <td>21144</td>
    </tr>
    <tr>
      <th>21145</th>
      <td>2/2/2012</td>
      <td>17:27:07</td>
      <td>-17.954</td>
      <td>167.179</td>
      <td>5.5</td>
      <td>20.6</td>
      <td>2012</td>
      <td>2</td>
      <td>2</td>
      <td>21145</td>
    </tr>
  </tbody>
</table>
</div>



I will look at the time, longitude, and latitude of the observations in the earthquakes and if any matches the tsunami values, then it is assumed that that specific earthquake caused the tsunami. The earthquake observation that matches this tsunami observation is the third observation in the earthquakes that happened in February 2012.


```python
earthquakes.loc[earthquakes['ID'] == 21144]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21144</th>
      <td>2/2/2012</td>
      <td>13:34:41</td>
      <td>-17.827</td>
      <td>167.133</td>
      <td>7.1</td>
      <td>23.0</td>
      <td>2012</td>
      <td>2</td>
      <td>2</td>
      <td>21144</td>
    </tr>
  </tbody>
</table>
</div>



Now I will do the same for March and the rest of the months


```python
tsu2012.loc[tsu2012[u'MONTH'] == 3]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2516</th>
      <td>5446</td>
      <td>2012</td>
      <td>3.0</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>8.0</td>
      <td>JAPAN</td>
      <td>NaN</td>
      <td>HOKKAIDO ISLAND</td>
      <td>40.887</td>
      <td>144.944</td>
    </tr>
    <tr>
      <th>2517</th>
      <td>5447</td>
      <td>2012</td>
      <td>3.0</td>
      <td>20.0</td>
      <td>18.0</td>
      <td>2.0</td>
      <td>MEXICO</td>
      <td>NaN</td>
      <td>S. MEXICO</td>
      <td>16.493</td>
      <td>-98.231</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '3') & ((eq2012['Days'] == '14') | (eq2012['Days'] == '20'))]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21192</th>
      <td>3/14/2012</td>
      <td>9:08:35</td>
      <td>40.887</td>
      <td>144.944</td>
      <td>6.9</td>
      <td>12.0</td>
      <td>2012</td>
      <td>3</td>
      <td>14</td>
      <td>21192</td>
    </tr>
    <tr>
      <th>21193</th>
      <td>3/14/2012</td>
      <td>10:49:25</td>
      <td>40.781</td>
      <td>144.761</td>
      <td>6.1</td>
      <td>10.0</td>
      <td>2012</td>
      <td>3</td>
      <td>14</td>
      <td>21193</td>
    </tr>
    <tr>
      <th>21194</th>
      <td>3/14/2012</td>
      <td>10:57:40</td>
      <td>40.755</td>
      <td>144.806</td>
      <td>5.6</td>
      <td>12.0</td>
      <td>2012</td>
      <td>3</td>
      <td>14</td>
      <td>21194</td>
    </tr>
    <tr>
      <th>21195</th>
      <td>3/14/2012</td>
      <td>12:05:05</td>
      <td>35.687</td>
      <td>140.695</td>
      <td>6.0</td>
      <td>10.0</td>
      <td>2012</td>
      <td>3</td>
      <td>14</td>
      <td>21195</td>
    </tr>
    <tr>
      <th>21196</th>
      <td>3/14/2012</td>
      <td>21:13:08</td>
      <td>-5.595</td>
      <td>151.042</td>
      <td>6.2</td>
      <td>28.0</td>
      <td>2012</td>
      <td>3</td>
      <td>14</td>
      <td>21196</td>
    </tr>
    <tr>
      <th>21202</th>
      <td>3/20/2012</td>
      <td>17:56:19</td>
      <td>-3.812</td>
      <td>140.266</td>
      <td>6.1</td>
      <td>66.0</td>
      <td>2012</td>
      <td>3</td>
      <td>20</td>
      <td>21202</td>
    </tr>
    <tr>
      <th>21203</th>
      <td>3/20/2012</td>
      <td>18:02:47</td>
      <td>16.493</td>
      <td>-98.231</td>
      <td>7.4</td>
      <td>20.0</td>
      <td>2012</td>
      <td>3</td>
      <td>20</td>
      <td>21203</td>
    </tr>
  </tbody>
</table>
</div>




```python
earthquakes.loc[(earthquakes['ID'] == 21192) | (earthquakes['ID'] == 21203)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21192</th>
      <td>3/14/2012</td>
      <td>9:08:35</td>
      <td>40.887</td>
      <td>144.944</td>
      <td>6.9</td>
      <td>12.0</td>
      <td>2012</td>
      <td>3</td>
      <td>14</td>
      <td>21192</td>
    </tr>
    <tr>
      <th>21203</th>
      <td>3/20/2012</td>
      <td>18:02:47</td>
      <td>16.493</td>
      <td>-98.231</td>
      <td>7.4</td>
      <td>20.0</td>
      <td>2012</td>
      <td>3</td>
      <td>20</td>
      <td>21203</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu2012.loc[tsu2012[u'MONTH'] == 4]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2518</th>
      <td>5449</td>
      <td>2012</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>8.0</td>
      <td>38.0</td>
      <td>INDONESIA</td>
      <td>NaN</td>
      <td>OFF W. COAST OF N SUMATRA</td>
      <td>2.327</td>
      <td>93.063</td>
    </tr>
    <tr>
      <th>2519</th>
      <td>5450</td>
      <td>2012</td>
      <td>4.0</td>
      <td>11.0</td>
      <td>10.0</td>
      <td>43.0</td>
      <td>INDONESIA</td>
      <td>NaN</td>
      <td>OFF W. COAST OF N SUMATRA</td>
      <td>0.802</td>
      <td>92.463</td>
    </tr>
    <tr>
      <th>2520</th>
      <td>5451</td>
      <td>2012</td>
      <td>4.0</td>
      <td>14.0</td>
      <td>22.0</td>
      <td>5.0</td>
      <td>VANUATU</td>
      <td>NaN</td>
      <td>VANUATU ISLANDS</td>
      <td>-18.972</td>
      <td>168.741</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '4') & ((eq2012['Days'] == '11') | (eq2012['Days'] == '14'))]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21219</th>
      <td>4/11/2012</td>
      <td>8:38:37</td>
      <td>2.327</td>
      <td>93.063</td>
      <td>8.6</td>
      <td>20.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21219</td>
    </tr>
    <tr>
      <th>21220</th>
      <td>4/11/2012</td>
      <td>8:55:47</td>
      <td>1.271</td>
      <td>91.748</td>
      <td>5.8</td>
      <td>10.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21220</td>
    </tr>
    <tr>
      <th>21221</th>
      <td>4/11/2012</td>
      <td>9:00:10</td>
      <td>51.364</td>
      <td>-176.097</td>
      <td>5.5</td>
      <td>20.8</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21221</td>
    </tr>
    <tr>
      <th>21222</th>
      <td>4/11/2012</td>
      <td>9:01:07</td>
      <td>2.199</td>
      <td>89.441</td>
      <td>5.9</td>
      <td>10.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21222</td>
    </tr>
    <tr>
      <th>21223</th>
      <td>4/11/2012</td>
      <td>9:27:57</td>
      <td>1.254</td>
      <td>91.735</td>
      <td>6.0</td>
      <td>10.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21223</td>
    </tr>
    <tr>
      <th>21224</th>
      <td>4/11/2012</td>
      <td>10:43:11</td>
      <td>0.802</td>
      <td>92.463</td>
      <td>8.2</td>
      <td>25.1</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21224</td>
    </tr>
    <tr>
      <th>21225</th>
      <td>4/11/2012</td>
      <td>11:53:36</td>
      <td>2.913</td>
      <td>89.544</td>
      <td>5.7</td>
      <td>10.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21225</td>
    </tr>
    <tr>
      <th>21226</th>
      <td>4/11/2012</td>
      <td>13:58:05</td>
      <td>1.495</td>
      <td>90.854</td>
      <td>5.5</td>
      <td>5.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21226</td>
    </tr>
    <tr>
      <th>21227</th>
      <td>4/11/2012</td>
      <td>19:04:20</td>
      <td>1.190</td>
      <td>92.092</td>
      <td>5.5</td>
      <td>14.5</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21227</td>
    </tr>
    <tr>
      <th>21228</th>
      <td>4/11/2012</td>
      <td>22:41:46</td>
      <td>43.584</td>
      <td>-127.638</td>
      <td>6.0</td>
      <td>8.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21228</td>
    </tr>
    <tr>
      <th>21229</th>
      <td>4/11/2012</td>
      <td>22:55:10</td>
      <td>18.229</td>
      <td>-102.689</td>
      <td>6.5</td>
      <td>20.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21229</td>
    </tr>
    <tr>
      <th>21230</th>
      <td>4/11/2012</td>
      <td>23:56:33</td>
      <td>1.841</td>
      <td>89.685</td>
      <td>5.8</td>
      <td>10.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21230</td>
    </tr>
    <tr>
      <th>21235</th>
      <td>4/14/2012</td>
      <td>10:56:19</td>
      <td>-57.679</td>
      <td>-65.308</td>
      <td>6.2</td>
      <td>15.0</td>
      <td>2012</td>
      <td>4</td>
      <td>14</td>
      <td>21235</td>
    </tr>
    <tr>
      <th>21236</th>
      <td>4/14/2012</td>
      <td>15:13:14</td>
      <td>49.380</td>
      <td>155.651</td>
      <td>5.6</td>
      <td>90.3</td>
      <td>2012</td>
      <td>4</td>
      <td>14</td>
      <td>21236</td>
    </tr>
    <tr>
      <th>21237</th>
      <td>4/14/2012</td>
      <td>19:26:43</td>
      <td>-6.810</td>
      <td>105.457</td>
      <td>5.8</td>
      <td>62.7</td>
      <td>2012</td>
      <td>4</td>
      <td>14</td>
      <td>21237</td>
    </tr>
    <tr>
      <th>21238</th>
      <td>4/14/2012</td>
      <td>22:05:26</td>
      <td>-18.972</td>
      <td>168.741</td>
      <td>6.2</td>
      <td>11.0</td>
      <td>2012</td>
      <td>4</td>
      <td>14</td>
      <td>21238</td>
    </tr>
  </tbody>
</table>
</div>




```python
earthquakes.loc[(earthquakes['ID'] == 21219) | (earthquakes['ID'] == 21224) | (earthquakes['ID'] == 21238)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21219</th>
      <td>4/11/2012</td>
      <td>8:38:37</td>
      <td>2.327</td>
      <td>93.063</td>
      <td>8.6</td>
      <td>20.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21219</td>
    </tr>
    <tr>
      <th>21224</th>
      <td>4/11/2012</td>
      <td>10:43:11</td>
      <td>0.802</td>
      <td>92.463</td>
      <td>8.2</td>
      <td>25.1</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21224</td>
    </tr>
    <tr>
      <th>21238</th>
      <td>4/14/2012</td>
      <td>22:05:26</td>
      <td>-18.972</td>
      <td>168.741</td>
      <td>6.2</td>
      <td>11.0</td>
      <td>2012</td>
      <td>4</td>
      <td>14</td>
      <td>21238</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu2012.loc[tsu2012[u'MONTH'] == 7]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2521</th>
      <td>5460</td>
      <td>2012</td>
      <td>7.0</td>
      <td>15.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>GREENLAND</td>
      <td>NaN</td>
      <td>ILULISSAT ICEFJORD</td>
      <td>69.2</td>
      <td>-51.3</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '7') & (eq2012['Days'] == '15')]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
tsu2012.loc[tsu2012[u'MONTH'] == 8]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2522</th>
      <td>5462</td>
      <td>2012</td>
      <td>8.0</td>
      <td>27.0</td>
      <td>4.0</td>
      <td>37.0</td>
      <td>NICARAGUA</td>
      <td>NaN</td>
      <td>OFF THE COAST</td>
      <td>12.139</td>
      <td>-88.590</td>
    </tr>
    <tr>
      <th>2523</th>
      <td>5463</td>
      <td>2012</td>
      <td>8.0</td>
      <td>31.0</td>
      <td>12.0</td>
      <td>47.0</td>
      <td>PHILIPPINES</td>
      <td>NaN</td>
      <td>PHILIPPINE ISLANDS</td>
      <td>10.811</td>
      <td>126.638</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '8') & ((eq2012['Days'] == '27') | (eq2012['Days'] == '31'))]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21405</th>
      <td>8/27/2012</td>
      <td>4:37:19</td>
      <td>12.139</td>
      <td>-88.590</td>
      <td>7.3</td>
      <td>28.0</td>
      <td>2012</td>
      <td>8</td>
      <td>27</td>
      <td>21405</td>
    </tr>
    <tr>
      <th>21406</th>
      <td>8/27/2012</td>
      <td>5:38:04</td>
      <td>12.297</td>
      <td>-88.612</td>
      <td>5.5</td>
      <td>35.0</td>
      <td>2012</td>
      <td>8</td>
      <td>27</td>
      <td>21406</td>
    </tr>
    <tr>
      <th>21411</th>
      <td>8/31/2012</td>
      <td>12:47:33</td>
      <td>10.811</td>
      <td>126.638</td>
      <td>7.6</td>
      <td>28.0</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
      <td>21411</td>
    </tr>
    <tr>
      <th>21412</th>
      <td>8/31/2012</td>
      <td>23:37:58</td>
      <td>10.388</td>
      <td>126.719</td>
      <td>5.6</td>
      <td>40.3</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
      <td>21412</td>
    </tr>
  </tbody>
</table>
</div>




```python
earthquakes.loc[(earthquakes['ID'] == 21405) | (earthquakes['ID'] == 21411)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21405</th>
      <td>8/27/2012</td>
      <td>4:37:19</td>
      <td>12.139</td>
      <td>-88.590</td>
      <td>7.3</td>
      <td>28.0</td>
      <td>2012</td>
      <td>8</td>
      <td>27</td>
      <td>21405</td>
    </tr>
    <tr>
      <th>21411</th>
      <td>8/31/2012</td>
      <td>12:47:33</td>
      <td>10.811</td>
      <td>126.638</td>
      <td>7.6</td>
      <td>28.0</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
      <td>21411</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu2012.loc[tsu2012[u'MONTH'] == 9]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2524</th>
      <td>5464</td>
      <td>2012</td>
      <td>9.0</td>
      <td>5.0</td>
      <td>14.0</td>
      <td>42.0</td>
      <td>COSTA RICA</td>
      <td>NaN</td>
      <td>COSTA RICA</td>
      <td>10.085</td>
      <td>-85.315</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '9') & (eq2012['Days'] == '5')]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21417</th>
      <td>9/5/2012</td>
      <td>13:09:10</td>
      <td>-12.476</td>
      <td>166.513</td>
      <td>6.0</td>
      <td>27.0</td>
      <td>2012</td>
      <td>9</td>
      <td>5</td>
      <td>21417</td>
    </tr>
    <tr>
      <th>21418</th>
      <td>9/5/2012</td>
      <td>14:42:08</td>
      <td>10.085</td>
      <td>-85.315</td>
      <td>7.6</td>
      <td>35.0</td>
      <td>2012</td>
      <td>9</td>
      <td>5</td>
      <td>21418</td>
    </tr>
  </tbody>
</table>
</div>




```python
earthquakes.loc[(earthquakes['ID'] == 21418)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21418</th>
      <td>9/5/2012</td>
      <td>14:42:08</td>
      <td>10.085</td>
      <td>-85.315</td>
      <td>7.6</td>
      <td>35.0</td>
      <td>2012</td>
      <td>9</td>
      <td>5</td>
      <td>21418</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu2012.loc[tsu2012[u'MONTH'] == 10]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2525</th>
      <td>5467</td>
      <td>2012</td>
      <td>10.0</td>
      <td>28.0</td>
      <td>3.0</td>
      <td>4.0</td>
      <td>CANADA</td>
      <td>BC</td>
      <td>BRITISH COLUMBIA</td>
      <td>52.788</td>
      <td>-132.101</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '10') & (eq2012['Days'] == '28')]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21477</th>
      <td>10/28/2012</td>
      <td>3:04:09</td>
      <td>52.788</td>
      <td>-132.101</td>
      <td>7.8</td>
      <td>14.0</td>
      <td>2012</td>
      <td>10</td>
      <td>28</td>
      <td>21477</td>
    </tr>
    <tr>
      <th>21478</th>
      <td>10/28/2012</td>
      <td>3:52:20</td>
      <td>52.576</td>
      <td>-131.962</td>
      <td>5.5</td>
      <td>10.0</td>
      <td>2012</td>
      <td>10</td>
      <td>28</td>
      <td>21478</td>
    </tr>
    <tr>
      <th>21479</th>
      <td>10/28/2012</td>
      <td>18:54:21</td>
      <td>52.674</td>
      <td>-132.602</td>
      <td>6.3</td>
      <td>9.0</td>
      <td>2012</td>
      <td>10</td>
      <td>28</td>
      <td>21479</td>
    </tr>
    <tr>
      <th>21480</th>
      <td>10/28/2012</td>
      <td>19:09:54</td>
      <td>52.294</td>
      <td>-132.082</td>
      <td>5.6</td>
      <td>10.0</td>
      <td>2012</td>
      <td>10</td>
      <td>28</td>
      <td>21480</td>
    </tr>
  </tbody>
</table>
</div>




```python
earthquakes.loc[(earthquakes['ID'] == 21477)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21477</th>
      <td>10/28/2012</td>
      <td>3:04:09</td>
      <td>52.788</td>
      <td>-132.101</td>
      <td>7.8</td>
      <td>14.0</td>
      <td>2012</td>
      <td>10</td>
      <td>28</td>
      <td>21477</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu2012.loc[tsu2012[u'MONTH'] == 11]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2526</th>
      <td>5468</td>
      <td>2012</td>
      <td>11.0</td>
      <td>7.0</td>
      <td>16.0</td>
      <td>35.0</td>
      <td>GUATEMALA</td>
      <td>NaN</td>
      <td>GUATEMALA</td>
      <td>13.988</td>
      <td>-91.895</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '11') & (eq2012['Days'] == '7')]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21493</th>
      <td>11/7/2012</td>
      <td>16:35:47</td>
      <td>13.988</td>
      <td>-91.895</td>
      <td>7.4</td>
      <td>24.0</td>
      <td>2012</td>
      <td>11</td>
      <td>7</td>
      <td>21493</td>
    </tr>
    <tr>
      <th>21494</th>
      <td>11/7/2012</td>
      <td>22:42:48</td>
      <td>13.849</td>
      <td>-92.156</td>
      <td>5.7</td>
      <td>35.0</td>
      <td>2012</td>
      <td>11</td>
      <td>7</td>
      <td>21494</td>
    </tr>
    <tr>
      <th>21495</th>
      <td>11/7/2012</td>
      <td>23:42:19</td>
      <td>-8.652</td>
      <td>148.034</td>
      <td>5.6</td>
      <td>118.4</td>
      <td>2012</td>
      <td>11</td>
      <td>7</td>
      <td>21495</td>
    </tr>
  </tbody>
</table>
</div>




```python
earthquakes.loc[(earthquakes['ID'] == 21493)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21493</th>
      <td>11/7/2012</td>
      <td>16:35:47</td>
      <td>13.988</td>
      <td>-91.895</td>
      <td>7.4</td>
      <td>24.0</td>
      <td>2012</td>
      <td>11</td>
      <td>7</td>
      <td>21493</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu2012.loc[tsu2012[u'MONTH'] == 12]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2527</th>
      <td>5469</td>
      <td>2012</td>
      <td>12.0</td>
      <td>7.0</td>
      <td>8.0</td>
      <td>18.0</td>
      <td>JAPAN</td>
      <td>NaN</td>
      <td>OFF EAST COAST OF HONSHU ISLAND</td>
      <td>37.890</td>
      <td>143.949</td>
    </tr>
    <tr>
      <th>2528</th>
      <td>5471</td>
      <td>2012</td>
      <td>12.0</td>
      <td>28.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>CHINA</td>
      <td>NaN</td>
      <td>ZHAOJUN BRIDGE, HUBEI PROVINCE</td>
      <td>31.256</td>
      <td>110.733</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq2012.loc[(eq2012['Month'] == '12') & ((eq2012['Days'] == '7') | (eq2012['Days'] == '28'))]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21530</th>
      <td>12/7/2012</td>
      <td>8:18:23</td>
      <td>37.890</td>
      <td>143.949</td>
      <td>7.3</td>
      <td>31.0</td>
      <td>2012</td>
      <td>12</td>
      <td>7</td>
      <td>21530</td>
    </tr>
    <tr>
      <th>21531</th>
      <td>12/7/2012</td>
      <td>8:31:15</td>
      <td>37.914</td>
      <td>143.764</td>
      <td>6.2</td>
      <td>32.0</td>
      <td>2012</td>
      <td>12</td>
      <td>7</td>
      <td>21531</td>
    </tr>
    <tr>
      <th>21532</th>
      <td>12/7/2012</td>
      <td>8:48:13</td>
      <td>37.828</td>
      <td>143.607</td>
      <td>5.5</td>
      <td>20.2</td>
      <td>2012</td>
      <td>12</td>
      <td>7</td>
      <td>21532</td>
    </tr>
    <tr>
      <th>21533</th>
      <td>12/7/2012</td>
      <td>18:19:06</td>
      <td>-38.428</td>
      <td>176.067</td>
      <td>6.3</td>
      <td>163.0</td>
      <td>2012</td>
      <td>12</td>
      <td>7</td>
      <td>21533</td>
    </tr>
    <tr>
      <th>21534</th>
      <td>12/7/2012</td>
      <td>19:50:23</td>
      <td>-7.661</td>
      <td>146.954</td>
      <td>5.7</td>
      <td>139.8</td>
      <td>2012</td>
      <td>12</td>
      <td>7</td>
      <td>21534</td>
    </tr>
    <tr>
      <th>21553</th>
      <td>12/28/2012</td>
      <td>17:32:18</td>
      <td>-0.145</td>
      <td>122.918</td>
      <td>5.5</td>
      <td>112.1</td>
      <td>2012</td>
      <td>12</td>
      <td>28</td>
      <td>21553</td>
    </tr>
  </tbody>
</table>
</div>




```python
earthquakes.loc[(earthquakes['ID'] == 21530)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21530</th>
      <td>12/7/2012</td>
      <td>8:18:23</td>
      <td>37.89</td>
      <td>143.949</td>
      <td>7.3</td>
      <td>31.0</td>
      <td>2012</td>
      <td>12</td>
      <td>7</td>
      <td>21530</td>
    </tr>
  </tbody>
</table>
</div>




```python
eqtsu2012 = earthquakes.loc[(earthquakes['ID'] == 21144) | (earthquakes['ID'] == 21192) | (earthquakes['ID'] == 21203) | 
                (earthquakes['ID'] == 21405) | (earthquakes['ID'] == 21219) | (earthquakes['ID'] == 21224) | 
                (earthquakes['ID'] == 21238) | (earthquakes['ID'] == 21405) | (earthquakes['ID'] == 21411) | 
                (earthquakes['ID'] == 21418) | (earthquakes['ID'] == 21477) | (earthquakes['ID'] == 21493) | 
                (earthquakes['ID'] == 21530)]
```


```python
eqtsu2012
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21144</th>
      <td>2/2/2012</td>
      <td>13:34:41</td>
      <td>-17.827</td>
      <td>167.133</td>
      <td>7.1</td>
      <td>23.0</td>
      <td>2012</td>
      <td>2</td>
      <td>2</td>
      <td>21144</td>
    </tr>
    <tr>
      <th>21192</th>
      <td>3/14/2012</td>
      <td>9:08:35</td>
      <td>40.887</td>
      <td>144.944</td>
      <td>6.9</td>
      <td>12.0</td>
      <td>2012</td>
      <td>3</td>
      <td>14</td>
      <td>21192</td>
    </tr>
    <tr>
      <th>21203</th>
      <td>3/20/2012</td>
      <td>18:02:47</td>
      <td>16.493</td>
      <td>-98.231</td>
      <td>7.4</td>
      <td>20.0</td>
      <td>2012</td>
      <td>3</td>
      <td>20</td>
      <td>21203</td>
    </tr>
    <tr>
      <th>21219</th>
      <td>4/11/2012</td>
      <td>8:38:37</td>
      <td>2.327</td>
      <td>93.063</td>
      <td>8.6</td>
      <td>20.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21219</td>
    </tr>
    <tr>
      <th>21224</th>
      <td>4/11/2012</td>
      <td>10:43:11</td>
      <td>0.802</td>
      <td>92.463</td>
      <td>8.2</td>
      <td>25.1</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21224</td>
    </tr>
    <tr>
      <th>21238</th>
      <td>4/14/2012</td>
      <td>22:05:26</td>
      <td>-18.972</td>
      <td>168.741</td>
      <td>6.2</td>
      <td>11.0</td>
      <td>2012</td>
      <td>4</td>
      <td>14</td>
      <td>21238</td>
    </tr>
    <tr>
      <th>21405</th>
      <td>8/27/2012</td>
      <td>4:37:19</td>
      <td>12.139</td>
      <td>-88.590</td>
      <td>7.3</td>
      <td>28.0</td>
      <td>2012</td>
      <td>8</td>
      <td>27</td>
      <td>21405</td>
    </tr>
    <tr>
      <th>21411</th>
      <td>8/31/2012</td>
      <td>12:47:33</td>
      <td>10.811</td>
      <td>126.638</td>
      <td>7.6</td>
      <td>28.0</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
      <td>21411</td>
    </tr>
    <tr>
      <th>21418</th>
      <td>9/5/2012</td>
      <td>14:42:08</td>
      <td>10.085</td>
      <td>-85.315</td>
      <td>7.6</td>
      <td>35.0</td>
      <td>2012</td>
      <td>9</td>
      <td>5</td>
      <td>21418</td>
    </tr>
    <tr>
      <th>21477</th>
      <td>10/28/2012</td>
      <td>3:04:09</td>
      <td>52.788</td>
      <td>-132.101</td>
      <td>7.8</td>
      <td>14.0</td>
      <td>2012</td>
      <td>10</td>
      <td>28</td>
      <td>21477</td>
    </tr>
    <tr>
      <th>21493</th>
      <td>11/7/2012</td>
      <td>16:35:47</td>
      <td>13.988</td>
      <td>-91.895</td>
      <td>7.4</td>
      <td>24.0</td>
      <td>2012</td>
      <td>11</td>
      <td>7</td>
      <td>21493</td>
    </tr>
    <tr>
      <th>21530</th>
      <td>12/7/2012</td>
      <td>8:18:23</td>
      <td>37.890</td>
      <td>143.949</td>
      <td>7.3</td>
      <td>31.0</td>
      <td>2012</td>
      <td>12</td>
      <td>7</td>
      <td>21530</td>
    </tr>
  </tbody>
</table>
</div>




```python
print float(len(eqtsu2012))/float(len(tsu2012)), float(len(eqtsu2012))/float(len(eq2012))
```

    0.857142857143 0.0269662921348


About 86% of the tsunamis in 2012 were caused by earthquakes and about 2.7% of earthquakes in 2012 cause tsunamis.


```python
plt.figure(figsize=(15,10))
displaymap2012 = Basemap(llcrnrlon=-180,llcrnrlat=-90,urcrnrlon=180,urcrnrlat=90)
displaymap2012.drawmapboundary()
displaymap2012.drawcountries()
displaymap2012.drawcoastlines()
longitude2012 = eqtsu2012[['Longitude']].values.tolist()
for i in range(0, len(longitude2012)):
    longitude2012[i] = float(longitude2012[i][0])
latitude2012 = eqtsu2012[['Latitude']].values.tolist()
for i in range(0, len(latitude2012)):
    latitude2012[i] = float(latitude2012[i][0])
lons2012,lats2012 = displaymap(longitude2012, latitude2012)
displaymap2012.plot(lons2012, lats2012, 'bo', color = "blue")
```




    [<matplotlib.lines.Line2D at 0xc44cf98>]




```python
plt.title("Earthquakes that Caused Tsunamis in 2012")
plt.show()
```


![](output_70_0.png)


From the world map, all the earthquakes that caused the tsunamis were from areas near bodies of water.


```python
min2012 = eqtsu2012['Magnitude'].min()
max2012 = eqtsu2012['Magnitude'].max()
print min2012, max2012
```

    6.2 8.6


The magnitudes of earthquakes that caused tsunamis in 2012 ranges from 6.2 to 8.6


```python
plt.figure(figsize=(10,10))
plt.hist(eqtsu2012['Magnitude'], bins = 5, alpha = 0.4)
plt.xlabel('Magnitude')
plt.ylabel('Frequency')
plt.title("Frequencies of Earthquakes that Caused Tsunamis in 2012")
plt.show()
```


![png](output_74_0.png)


From the histogram, most of the earthquakes that caused tsunamis lies between the range of 7 to 7.5 degrees of magnitude.

Now I pick another year, 1997 to see how much and what degree magnitudes of earthquakes cause tsunamis and see if the results are similar or consistent with the year 2012.


```python
eq1997 = earthquakes.loc[(earthquakes['Year'] == '1997')]
tsu1997 = tsu.loc[tsu[u'YEAR'] == 1997]
```


```python
tsu1997
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2362</th>
      <td>5416</td>
      <td>1997</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>HONDURAS</td>
      <td>NaN</td>
      <td>GULF OF FONSECA</td>
      <td>13.100</td>
      <td>-87.600</td>
    </tr>
    <tr>
      <th>2364</th>
      <td>2273</td>
      <td>1997</td>
      <td>4.0</td>
      <td>21.0</td>
      <td>12.0</td>
      <td>2.0</td>
      <td>SOLOMON ISLANDS</td>
      <td>NaN</td>
      <td>SANTA CRUZ IS. VANUATU</td>
      <td>-12.584</td>
      <td>166.676</td>
    </tr>
    <tr>
      <th>2365</th>
      <td>2274</td>
      <td>1997</td>
      <td>7.0</td>
      <td>9.0</td>
      <td>19.0</td>
      <td>24.0</td>
      <td>VENEZUELA</td>
      <td>NaN</td>
      <td>CARIACO-CUMANA</td>
      <td>10.598</td>
      <td>-63.486</td>
    </tr>
    <tr>
      <th>2366</th>
      <td>3034</td>
      <td>1997</td>
      <td>9.0</td>
      <td>30.0</td>
      <td>6.0</td>
      <td>27.0</td>
      <td>JAPAN</td>
      <td>NaN</td>
      <td>S. OF HONSHU ISLAND</td>
      <td>31.959</td>
      <td>141.878</td>
    </tr>
    <tr>
      <th>2367</th>
      <td>2275</td>
      <td>1997</td>
      <td>10.0</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>53.0</td>
      <td>TONGA</td>
      <td>NaN</td>
      <td>TONGA ISLANDS</td>
      <td>-22.100</td>
      <td>-176.770</td>
    </tr>
    <tr>
      <th>2368</th>
      <td>2277</td>
      <td>1997</td>
      <td>12.0</td>
      <td>5.0</td>
      <td>11.0</td>
      <td>26.0</td>
      <td>RUSSIA</td>
      <td>NaN</td>
      <td>KAMCHATKA</td>
      <td>54.841</td>
      <td>162.035</td>
    </tr>
    <tr>
      <th>2369</th>
      <td>2278</td>
      <td>1997</td>
      <td>12.0</td>
      <td>14.0</td>
      <td>3.0</td>
      <td>30.0</td>
      <td>RUSSIA</td>
      <td>NaN</td>
      <td>KAMCHATKA</td>
      <td>54.841</td>
      <td>162.035</td>
    </tr>
    <tr>
      <th>2370</th>
      <td>2279</td>
      <td>1997</td>
      <td>12.0</td>
      <td>26.0</td>
      <td>8.0</td>
      <td>NaN</td>
      <td>MONTSERRAT</td>
      <td>NaN</td>
      <td>WHITE RIVER VALLEY</td>
      <td>16.720</td>
      <td>-62.180</td>
    </tr>
  </tbody>
</table>
</div>




```python
print len(tsu1997.index), len(eq1997.index)
```

    8 456


In the year 1997, it looks like there are 2 tsunamis in April, 1 in July, 1 in September, 1 in October, and 3 in December with a total of 8 tsunamis. There are 456 earthquakes total in the year 1997.


```python
tsu1997.loc[tsu1997[u'MONTH'] == 4]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2362</th>
      <td>5416</td>
      <td>1997</td>
      <td>4.0</td>
      <td>10.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>HONDURAS</td>
      <td>NaN</td>
      <td>GULF OF FONSECA</td>
      <td>13.100</td>
      <td>-87.600</td>
    </tr>
    <tr>
      <th>2364</th>
      <td>2273</td>
      <td>1997</td>
      <td>4.0</td>
      <td>21.0</td>
      <td>12.0</td>
      <td>2.0</td>
      <td>SOLOMON ISLANDS</td>
      <td>NaN</td>
      <td>SANTA CRUZ IS. VANUATU</td>
      <td>-12.584</td>
      <td>166.676</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['Month'] == '4') & ((eq1997['Days'] == '10') | (eq1997['Days'] == '21'))]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13495</th>
      <td>4/21/1997</td>
      <td>2:42:45</td>
      <td>-0.149</td>
      <td>124.073</td>
      <td>5.5</td>
      <td>50.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13495</td>
    </tr>
    <tr>
      <th>13496</th>
      <td>4/21/1997</td>
      <td>12:02:26</td>
      <td>-12.584</td>
      <td>166.676</td>
      <td>7.7</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13496</td>
    </tr>
    <tr>
      <th>13497</th>
      <td>4/21/1997</td>
      <td>12:06:34</td>
      <td>-12.881</td>
      <td>166.464</td>
      <td>6.1</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13497</td>
    </tr>
    <tr>
      <th>13498</th>
      <td>4/21/1997</td>
      <td>12:11:28</td>
      <td>-13.500</td>
      <td>166.541</td>
      <td>6.2</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13498</td>
    </tr>
    <tr>
      <th>13499</th>
      <td>4/21/1997</td>
      <td>12:15:57</td>
      <td>-13.406</td>
      <td>166.344</td>
      <td>6.0</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13499</td>
    </tr>
    <tr>
      <th>13500</th>
      <td>4/21/1997</td>
      <td>12:20:50</td>
      <td>-13.602</td>
      <td>166.832</td>
      <td>5.7</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13500</td>
    </tr>
    <tr>
      <th>13501</th>
      <td>4/21/1997</td>
      <td>12:23:46</td>
      <td>-13.673</td>
      <td>166.455</td>
      <td>5.5</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13501</td>
    </tr>
    <tr>
      <th>13502</th>
      <td>4/21/1997</td>
      <td>12:28:28</td>
      <td>-13.541</td>
      <td>166.426</td>
      <td>5.5</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13502</td>
    </tr>
    <tr>
      <th>13503</th>
      <td>4/21/1997</td>
      <td>14:01:24</td>
      <td>-7.382</td>
      <td>125.715</td>
      <td>5.9</td>
      <td>432.3</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13503</td>
    </tr>
    <tr>
      <th>13504</th>
      <td>4/21/1997</td>
      <td>21:23:54</td>
      <td>-13.158</td>
      <td>166.522</td>
      <td>5.5</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13504</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['ID'] == 13496)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13496</th>
      <td>4/21/1997</td>
      <td>12:02:26</td>
      <td>-12.584</td>
      <td>166.676</td>
      <td>7.7</td>
      <td>33.0</td>
      <td>1997</td>
      <td>4</td>
      <td>21</td>
      <td>13496</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu1997.loc[tsu1997[u'MONTH'] == 7]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2365</th>
      <td>2274</td>
      <td>1997</td>
      <td>7.0</td>
      <td>9.0</td>
      <td>19.0</td>
      <td>24.0</td>
      <td>VENEZUELA</td>
      <td>NaN</td>
      <td>CARIACO-CUMANA</td>
      <td>10.598</td>
      <td>-63.486</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['Month'] == '7') & (eq1997['Days'] == '9')]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13600</th>
      <td>7/9/1997</td>
      <td>19:24:13</td>
      <td>10.598</td>
      <td>-63.486</td>
      <td>7.0</td>
      <td>19.9</td>
      <td>1997</td>
      <td>7</td>
      <td>9</td>
      <td>13600</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['ID'] == 13600)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13600</th>
      <td>7/9/1997</td>
      <td>19:24:13</td>
      <td>10.598</td>
      <td>-63.486</td>
      <td>7.0</td>
      <td>19.9</td>
      <td>1997</td>
      <td>7</td>
      <td>9</td>
      <td>13600</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu1997.loc[tsu1997[u'MONTH'] == 9]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2366</th>
      <td>3034</td>
      <td>1997</td>
      <td>9.0</td>
      <td>30.0</td>
      <td>6.0</td>
      <td>27.0</td>
      <td>JAPAN</td>
      <td>NaN</td>
      <td>S. OF HONSHU ISLAND</td>
      <td>31.959</td>
      <td>141.878</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['Month'] == '9') & (eq1997['Days'] == '30')]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13688</th>
      <td>9/30/1997</td>
      <td>6:27:25</td>
      <td>31.959</td>
      <td>141.878</td>
      <td>6.2</td>
      <td>10.0</td>
      <td>1997</td>
      <td>9</td>
      <td>30</td>
      <td>13688</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['ID'] == 13688)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13688</th>
      <td>9/30/1997</td>
      <td>6:27:25</td>
      <td>31.959</td>
      <td>141.878</td>
      <td>6.2</td>
      <td>10.0</td>
      <td>1997</td>
      <td>9</td>
      <td>30</td>
      <td>13688</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu1997.loc[tsu1997[u'MONTH'] == 10]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2367</th>
      <td>2275</td>
      <td>1997</td>
      <td>10.0</td>
      <td>14.0</td>
      <td>9.0</td>
      <td>53.0</td>
      <td>TONGA</td>
      <td>NaN</td>
      <td>TONGA ISLANDS</td>
      <td>-22.1</td>
      <td>-176.77</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['Month'] == '10') & (eq1997['Days'] == '14')]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13711</th>
      <td>10/14/1997</td>
      <td>9:53:18</td>
      <td>-22.101</td>
      <td>-176.772</td>
      <td>7.8</td>
      <td>167.3</td>
      <td>1997</td>
      <td>10</td>
      <td>14</td>
      <td>13711</td>
    </tr>
    <tr>
      <th>13712</th>
      <td>10/14/1997</td>
      <td>15:23:10</td>
      <td>42.962</td>
      <td>12.892</td>
      <td>5.5</td>
      <td>10.0</td>
      <td>1997</td>
      <td>10</td>
      <td>14</td>
      <td>13712</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['ID'] == 13711)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13711</th>
      <td>10/14/1997</td>
      <td>9:53:18</td>
      <td>-22.101</td>
      <td>-176.772</td>
      <td>7.8</td>
      <td>167.3</td>
      <td>1997</td>
      <td>10</td>
      <td>14</td>
      <td>13711</td>
    </tr>
  </tbody>
</table>
</div>




```python
tsu1997.loc[tsu1997[u'MONTH'] == 12]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>YEAR</th>
      <th>MONTH</th>
      <th>DAY</th>
      <th>HOUR</th>
      <th>MINUTE</th>
      <th>COUNTRY</th>
      <th>STATE</th>
      <th>LOCATION_NAME</th>
      <th>LATITUDE</th>
      <th>LONGITUDE</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2368</th>
      <td>2277</td>
      <td>1997</td>
      <td>12.0</td>
      <td>5.0</td>
      <td>11.0</td>
      <td>26.0</td>
      <td>RUSSIA</td>
      <td>NaN</td>
      <td>KAMCHATKA</td>
      <td>54.841</td>
      <td>162.035</td>
    </tr>
    <tr>
      <th>2369</th>
      <td>2278</td>
      <td>1997</td>
      <td>12.0</td>
      <td>14.0</td>
      <td>3.0</td>
      <td>30.0</td>
      <td>RUSSIA</td>
      <td>NaN</td>
      <td>KAMCHATKA</td>
      <td>54.841</td>
      <td>162.035</td>
    </tr>
    <tr>
      <th>2370</th>
      <td>2279</td>
      <td>1997</td>
      <td>12.0</td>
      <td>26.0</td>
      <td>8.0</td>
      <td>NaN</td>
      <td>MONTSERRAT</td>
      <td>NaN</td>
      <td>WHITE RIVER VALLEY</td>
      <td>16.720</td>
      <td>-62.180</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['Month'] == '12') & ((eq1997['Days'] == '5') | (eq1997['Days'] == '14') | (eq1997['Days'] == '26'))]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13784</th>
      <td>12/5/1997</td>
      <td>8:08:50</td>
      <td>55.281</td>
      <td>162.444</td>
      <td>5.5</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>5</td>
      <td>13784</td>
    </tr>
    <tr>
      <th>13785</th>
      <td>12/5/1997</td>
      <td>11:26:55</td>
      <td>54.841</td>
      <td>162.035</td>
      <td>7.8</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>5</td>
      <td>13785</td>
    </tr>
    <tr>
      <th>13786</th>
      <td>12/5/1997</td>
      <td>11:35:20</td>
      <td>53.909</td>
      <td>161.550</td>
      <td>5.7</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>5</td>
      <td>13786</td>
    </tr>
    <tr>
      <th>13787</th>
      <td>12/5/1997</td>
      <td>11:37:09</td>
      <td>54.512</td>
      <td>162.318</td>
      <td>5.6</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>5</td>
      <td>13787</td>
    </tr>
    <tr>
      <th>13788</th>
      <td>12/5/1997</td>
      <td>13:56:12</td>
      <td>0.656</td>
      <td>125.114</td>
      <td>5.5</td>
      <td>89.4</td>
      <td>1997</td>
      <td>12</td>
      <td>5</td>
      <td>13788</td>
    </tr>
    <tr>
      <th>13789</th>
      <td>12/5/1997</td>
      <td>18:48:23</td>
      <td>53.752</td>
      <td>161.746</td>
      <td>6.4</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>5</td>
      <td>13789</td>
    </tr>
    <tr>
      <th>13790</th>
      <td>12/5/1997</td>
      <td>19:04:07</td>
      <td>53.792</td>
      <td>161.596</td>
      <td>5.5</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>5</td>
      <td>13790</td>
    </tr>
    <tr>
      <th>13806</th>
      <td>12/14/1997</td>
      <td>2:39:17</td>
      <td>-59.574</td>
      <td>-26.186</td>
      <td>5.7</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>14</td>
      <td>13806</td>
    </tr>
    <tr>
      <th>13807</th>
      <td>12/14/1997</td>
      <td>8:48:36</td>
      <td>-3.081</td>
      <td>136.106</td>
      <td>5.6</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>14</td>
      <td>13807</td>
    </tr>
    <tr>
      <th>13808</th>
      <td>12/14/1997</td>
      <td>23:10:04</td>
      <td>-15.571</td>
      <td>-173.173</td>
      <td>5.6</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>14</td>
      <td>13808</td>
    </tr>
    <tr>
      <th>13829</th>
      <td>12/26/1997</td>
      <td>5:34:25</td>
      <td>-22.338</td>
      <td>-179.690</td>
      <td>5.9</td>
      <td>588.4</td>
      <td>1997</td>
      <td>12</td>
      <td>26</td>
      <td>13829</td>
    </tr>
    <tr>
      <th>13830</th>
      <td>12/26/1997</td>
      <td>21:18:18</td>
      <td>51.310</td>
      <td>178.802</td>
      <td>5.6</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>26</td>
      <td>13830</td>
    </tr>
  </tbody>
</table>
</div>




```python
eq1997.loc[(eq1997['ID'] == 13785)]
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13785</th>
      <td>12/5/1997</td>
      <td>11:26:55</td>
      <td>54.841</td>
      <td>162.035</td>
      <td>7.8</td>
      <td>33.0</td>
      <td>1997</td>
      <td>12</td>
      <td>5</td>
      <td>13785</td>
    </tr>
  </tbody>
</table>
</div>




```python
eqtsu1997 = earthquakes.loc[(earthquakes['ID'] == 13469) | (earthquakes['ID'] == 13600) | (earthquakes['ID'] == 13688) | 
                (earthquakes['ID'] == 13711) | (earthquakes['ID'] == 23785)]
```


```python
eqtsu1997
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13469</th>
      <td>4/2/1997</td>
      <td>19:33:22</td>
      <td>31.824</td>
      <td>130.089</td>
      <td>5.5</td>
      <td>10.0</td>
      <td>1997</td>
      <td>4</td>
      <td>2</td>
      <td>13469</td>
    </tr>
    <tr>
      <th>13600</th>
      <td>7/9/1997</td>
      <td>19:24:13</td>
      <td>10.598</td>
      <td>-63.486</td>
      <td>7.0</td>
      <td>19.9</td>
      <td>1997</td>
      <td>7</td>
      <td>9</td>
      <td>13600</td>
    </tr>
    <tr>
      <th>13688</th>
      <td>9/30/1997</td>
      <td>6:27:25</td>
      <td>31.959</td>
      <td>141.878</td>
      <td>6.2</td>
      <td>10.0</td>
      <td>1997</td>
      <td>9</td>
      <td>30</td>
      <td>13688</td>
    </tr>
    <tr>
      <th>13711</th>
      <td>10/14/1997</td>
      <td>9:53:18</td>
      <td>-22.101</td>
      <td>-176.772</td>
      <td>7.8</td>
      <td>167.3</td>
      <td>1997</td>
      <td>10</td>
      <td>14</td>
      <td>13711</td>
    </tr>
  </tbody>
</table>
</div>




```python
print float(len(eqtsu1997))/float(len(tsu1997)), float(len(eqtsu1997))/float(len(eq1997))
```

    0.5 0.00877192982456


About 50% of tsunamis were caused by earthquakes in 1997 and about 1% of earthquakes that year caused tsunamis.


```python
plt.figure(figsize=(15,10))
displaymap1997 = Basemap(llcrnrlon=-180,llcrnrlat=-90,urcrnrlon=180,urcrnrlat=90)
displaymap1997.drawmapboundary()
displaymap1997.drawcountries()
displaymap1997.drawcoastlines()
longitude1997 = eqtsu1997[['Longitude']].values.tolist()
for i in range(0, len(longitude1997)):
    longitude1997[i] = float(longitude1997[i][0])
latitude1997 = eqtsu1997[['Latitude']].values.tolist()
for i in range(0, len(latitude1997)):
    latitude1997[i] = float(latitude1997[i][0])
lons1997,lats1997 = displaymap(longitude1997, latitude1997)
displaymap1997.plot(lons1997, lats1997, 'bo', color = "blue")
```




    [<matplotlib.lines.Line2D at 0xb6ece10>]




```python
plt.title("Earthquakes that Caused Tsunamis in 1997")
plt.show()
```


![png](output_101_0.png)


Again, all the earthquakes that caused tsunamis happened near or in bodies of water so it's consistent with the observation from the world map for 2012. I will not provide a histogram for this year as there are only 4 earthquakes that caused tsunamis this year.


```python
min1997 = eqtsu1997['Magnitude'].min()
max1997 = eqtsu1997['Magnitude'].max()
print min1997, max1997
```

    5.5 7.8


The range of earthquakes that caused tsunamis for 1997 is between 5.5 and 7.8.

Now I want to combine the datasets of earthquakes that cause tsunamis I have gotten in the previous parts to see how that fits in with the observations I have obtained so far.


```python
eqcom = earthquakes.loc[(earthquakes['Year'] == '1997') | (earthquakes['Year'] == '2012')]
tsucom = tsu.loc[(tsu[u'YEAR'] == 1997) | (tsu[u'YEAR'] == 2012)]
frames = [eqtsu1997, eqtsu2012]
eqtsucom = pd.concat(frames)
```


```python
eqtsucom
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Time</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Magnitude</th>
      <th>Depth</th>
      <th>Year</th>
      <th>Month</th>
      <th>Days</th>
      <th>ID</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>13469</th>
      <td>4/2/1997</td>
      <td>19:33:22</td>
      <td>31.824</td>
      <td>130.089</td>
      <td>5.5</td>
      <td>10.0</td>
      <td>1997</td>
      <td>4</td>
      <td>2</td>
      <td>13469</td>
    </tr>
    <tr>
      <th>13600</th>
      <td>7/9/1997</td>
      <td>19:24:13</td>
      <td>10.598</td>
      <td>-63.486</td>
      <td>7.0</td>
      <td>19.9</td>
      <td>1997</td>
      <td>7</td>
      <td>9</td>
      <td>13600</td>
    </tr>
    <tr>
      <th>13688</th>
      <td>9/30/1997</td>
      <td>6:27:25</td>
      <td>31.959</td>
      <td>141.878</td>
      <td>6.2</td>
      <td>10.0</td>
      <td>1997</td>
      <td>9</td>
      <td>30</td>
      <td>13688</td>
    </tr>
    <tr>
      <th>13711</th>
      <td>10/14/1997</td>
      <td>9:53:18</td>
      <td>-22.101</td>
      <td>-176.772</td>
      <td>7.8</td>
      <td>167.3</td>
      <td>1997</td>
      <td>10</td>
      <td>14</td>
      <td>13711</td>
    </tr>
    <tr>
      <th>21144</th>
      <td>2/2/2012</td>
      <td>13:34:41</td>
      <td>-17.827</td>
      <td>167.133</td>
      <td>7.1</td>
      <td>23.0</td>
      <td>2012</td>
      <td>2</td>
      <td>2</td>
      <td>21144</td>
    </tr>
    <tr>
      <th>21192</th>
      <td>3/14/2012</td>
      <td>9:08:35</td>
      <td>40.887</td>
      <td>144.944</td>
      <td>6.9</td>
      <td>12.0</td>
      <td>2012</td>
      <td>3</td>
      <td>14</td>
      <td>21192</td>
    </tr>
    <tr>
      <th>21203</th>
      <td>3/20/2012</td>
      <td>18:02:47</td>
      <td>16.493</td>
      <td>-98.231</td>
      <td>7.4</td>
      <td>20.0</td>
      <td>2012</td>
      <td>3</td>
      <td>20</td>
      <td>21203</td>
    </tr>
    <tr>
      <th>21219</th>
      <td>4/11/2012</td>
      <td>8:38:37</td>
      <td>2.327</td>
      <td>93.063</td>
      <td>8.6</td>
      <td>20.0</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21219</td>
    </tr>
    <tr>
      <th>21224</th>
      <td>4/11/2012</td>
      <td>10:43:11</td>
      <td>0.802</td>
      <td>92.463</td>
      <td>8.2</td>
      <td>25.1</td>
      <td>2012</td>
      <td>4</td>
      <td>11</td>
      <td>21224</td>
    </tr>
    <tr>
      <th>21238</th>
      <td>4/14/2012</td>
      <td>22:05:26</td>
      <td>-18.972</td>
      <td>168.741</td>
      <td>6.2</td>
      <td>11.0</td>
      <td>2012</td>
      <td>4</td>
      <td>14</td>
      <td>21238</td>
    </tr>
    <tr>
      <th>21405</th>
      <td>8/27/2012</td>
      <td>4:37:19</td>
      <td>12.139</td>
      <td>-88.590</td>
      <td>7.3</td>
      <td>28.0</td>
      <td>2012</td>
      <td>8</td>
      <td>27</td>
      <td>21405</td>
    </tr>
    <tr>
      <th>21411</th>
      <td>8/31/2012</td>
      <td>12:47:33</td>
      <td>10.811</td>
      <td>126.638</td>
      <td>7.6</td>
      <td>28.0</td>
      <td>2012</td>
      <td>8</td>
      <td>31</td>
      <td>21411</td>
    </tr>
    <tr>
      <th>21418</th>
      <td>9/5/2012</td>
      <td>14:42:08</td>
      <td>10.085</td>
      <td>-85.315</td>
      <td>7.6</td>
      <td>35.0</td>
      <td>2012</td>
      <td>9</td>
      <td>5</td>
      <td>21418</td>
    </tr>
    <tr>
      <th>21477</th>
      <td>10/28/2012</td>
      <td>3:04:09</td>
      <td>52.788</td>
      <td>-132.101</td>
      <td>7.8</td>
      <td>14.0</td>
      <td>2012</td>
      <td>10</td>
      <td>28</td>
      <td>21477</td>
    </tr>
    <tr>
      <th>21493</th>
      <td>11/7/2012</td>
      <td>16:35:47</td>
      <td>13.988</td>
      <td>-91.895</td>
      <td>7.4</td>
      <td>24.0</td>
      <td>2012</td>
      <td>11</td>
      <td>7</td>
      <td>21493</td>
    </tr>
    <tr>
      <th>21530</th>
      <td>12/7/2012</td>
      <td>8:18:23</td>
      <td>37.890</td>
      <td>143.949</td>
      <td>7.3</td>
      <td>31.0</td>
      <td>2012</td>
      <td>12</td>
      <td>7</td>
      <td>21530</td>
    </tr>
  </tbody>
</table>
</div>




```python
print float(len(eqtsucom))/float(len(tsucom)), float(len(eqtsucom))/float(len(eqcom))
```

    0.727272727273 0.0177580466149


When averaged, approximately 72% of tsunamis are caused by earthquakes and about 2% of those earthquakes cause tsunamis.


```python
plt.figure(figsize=(15,10))
displaymapcom = Basemap(llcrnrlon=-180,llcrnrlat=-90,urcrnrlon=180,urcrnrlat=90)
displaymapcom.drawmapboundary()
displaymapcom.drawcountries()
displaymapcom.drawcoastlines()
longitudecom = eqtsucom[['Longitude']].values.tolist()
for i in range(0, len(longitudecom)):
    longitudecom[i] = float(longitudecom[i][0])
latitudecom = eqtsucom[['Latitude']].values.tolist()
for i in range(0, len(latitudecom)):
    latitudecom[i] = float(latitudecom[i][0])
lonscom,latscom = displaymap(longitudecom, latitudecom)
displaymapcom.plot(lonscom, latscom, 'bo', color = "blue")
```




    [<matplotlib.lines.Line2D at 0xb707048>]




```python
plt.title("Earthquakes that Caused Tsunamis in Both Years")
plt.show()
```


![png](output_1111_0.png)


All the earthquakes that cause tsunamis are located near or in bodies of water. This has been a consistent observation so far.


```python
mincom = eqtsucom['Magnitude'].min()
maxcom = eqtsucom['Magnitude'].max()
print mincom, maxcom
```

    5.5 8.6


The range of earthquakes that cause earthquakes for this set of observations is between 5.5 o 8.6 degrees of magnitude.


```python
plt.figure(figsize=(10,10))
plt.hist(eqtsucom['Magnitude'], bins = 5, alpha = 0.4)
plt.xlabel('Magnitude')
plt.ylabel('Frequency')
plt.title("Frequencies of Earthquakes that Caused Tsunamis in Combined Dataset")
plt.show()
```


![png](output_115_0.png)


In the histogram, it is shown that a majority of tsunamis are caused by earthquakes between 7 to 8 degrees of magnitude which is consistent with the observation I obtained in the 2012 dataset.

## Conclusion
In conclusion, most tsunamis are caused by earthquakes located near or in bodies of water on the world map but about less than 5% of earthquakes in the world actually cause tsunamis itself. I have found that the majority of earthquakes that cause tsunamis have a magnitude between 5 and 9 which are the big earthquakes. The samples I have taken are not representative of the whole dataset because the dataset could not be merged together but I believe that the results would be more accurate if there is more data that had been analyzed and if there is a larger sample for the data.


<h1>Part 4: The Relationship Between Earthquakes and Volcanos</h1>

<h3>Natalie Marcom</h3> 

Adding onto the discussion of how earthquakes affect tsunamis, we will also discuss how earthquakes may affect volcanic eruptions. There are approximetely 1.5k active volcanos on earth. However, I will focus on connecting earthquakes and volcanic eruptions to stay within the scope of the class, as I am not a geophysicist.

I used data from NOAA, a website from Oregonstate.edu with the list of volcanos with their latitude and longitude, volcano and plate boundary shapefiles from ArcMap (Esri), as well as data from volcanodiscovery.org to find data concerning recent earthquakes near volcanos.


```python
import requests
from lxml import html
from mpl_toolkits.basemap import Basemap

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

Let's plot all 1500 volcanos on a map to see where most of them are located. Due to the difficulty to acquire a reasonable dataset of volcanos, besides a shapefile from Arcmap, we will scrape from a website that indicates the Latitude and longiude of all the volcanos to make plotting easy. We will also plot the volcanos on a basemap by the size of the volcano, via it's elevation height in meters.


```python
page = requests.get('http://volcano.oregonstate.edu/oldroot/volcanoes/alpha.html')
tree = html.fromstring(page.content)
tables = tree.xpath('//table')

volcano_data = []
for volc in range(4, len(tables)):
    df = pd.read_html(html.tostring(tables[volc]), header=0)[0]
    volcano_data.append(df)
```


```python
df_volc = pd.concat(volcano_data, ignore_index=True)

```

Let's look at a small snippet of the volcano dataset that was scraped. We will take note of the main observations of this dataset.


```python
df_volc.head(10)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>Location</th>
      <th>Type</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Elevation (m)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abu</td>
      <td>Honshu-Japan</td>
      <td>Shield volcanoes</td>
      <td>34.50</td>
      <td>131.60</td>
      <td>641.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Acamarachi</td>
      <td>Chile-N</td>
      <td>Stratovolcano</td>
      <td>-23.30</td>
      <td>-67.62</td>
      <td>6046.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Acatenango</td>
      <td>Guatemala</td>
      <td>Stratovolcano</td>
      <td>14.50</td>
      <td>-90.88</td>
      <td>3976.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Acigöl-Nevsehir</td>
      <td>Turkey</td>
      <td>Caldera</td>
      <td>38.57</td>
      <td>34.52</td>
      <td>1689.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Adams</td>
      <td>US-Washington</td>
      <td>Stratovolcano</td>
      <td>46.21</td>
      <td>-121.49</td>
      <td>3742.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Adams  Seamount</td>
      <td>Pacific-C</td>
      <td>Submarine volcano</td>
      <td>-25.37</td>
      <td>-129.27</td>
      <td>-39.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Adatara</td>
      <td>Honshu-Japan</td>
      <td>Stratovolcanoes</td>
      <td>37.64</td>
      <td>140.29</td>
      <td>1718.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Adwa</td>
      <td>Ethiopia</td>
      <td>Stratovolcano</td>
      <td>10.07</td>
      <td>40.84</td>
      <td>1733.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Afderà</td>
      <td>Ethiopia</td>
      <td>Stratovolcano</td>
      <td>13.08</td>
      <td>40.85</td>
      <td>1295.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Agrigan</td>
      <td>Mariana Is-C Pacific</td>
      <td>Stratovolcano</td>
      <td>18.77</td>
      <td>145.67</td>
      <td>965.0</td>
    </tr>
  </tbody>
</table>
</div>



<h1>Where are the volcanos located? Are they near tetonic plates?<h1>


```python
import pandas as pd
from mpl_toolkits.basemap import Basemap
import matplotlib.pyplot as plt1
import matplotlib as mpl
import shapefile
from mpl_toolkits.basemap import Basemap
import geopandas as gp
```


```python
import os as osf
osf.chdir('C:\Users\jenat\\Documents\\ringoffire\\new')

volc = gp.GeoDataFrame.from_file('volcs.shp')
plt1.figure(figsize = (20, 12))
y = volc.LATX
x = volc.LONGX
map1 = Basemap()
map1.readshapefile('plate', 'plate')
map1.drawmapboundary(fill_color = 'lightskyblue')
map1.fillcontinents(color = 'lavender',lake_color = 'aqua')
map1.drawcountries()
map1.drawcoastlines()
volc_info = map1.readshapefile('volc1', 'volcs')

x1,y1 = map1(x,y)
map1.scatter(x1,y1,c = 'red',marker = "o",alpha = 1.0)
plt1.title("Map of Volcanos and Plate Boundaries", fontsize = 25)
plt1.show()
```


![png](output_12_0.png)


Using two shape files (one for plate bounaries, the other of the world's volcanos), we see that majority of the volcanos are very close to plate boundaries, that or they *are* along the tetonic plate boundaries.

------

However, besides plotting the volcanos on a map, let us take it a step further and plot volcanos as well as data that indicates whether one of these volcanos, had an eruption that was associated with an earthquake.  We will use two datasets to answer this question. The second dataset with the earthquake information mainly looks at volcano eruptions from 1790 to the present. I have decided to look at world volcanos for that data and not focus on a particular region of the world.

---------

<h1><u><center>How many of the volcanos have had eruptions that were associated with earthquakes?</center></h1></u>


```python
import os
import pandas as pd
from mpl_toolkits.basemap import Basemap
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
os.chdir('C:\Users\jenat\Documents')
#second dataset
data = pd.read_csv("new_world_data_results_up1.csv")
```


```python
data
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Month</th>
      <th>Day</th>
      <th>TSU</th>
      <th>EQ</th>
      <th>Name</th>
      <th>Location</th>
      <th>Country</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Elevation</th>
      <th>Type</th>
      <th>Status</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-1610.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Santorini</td>
      <td>Greece</td>
      <td>Greece</td>
      <td>36.404</td>
      <td>25.396</td>
      <td>329.0</td>
      <td>Shield volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>2</th>
      <td>766.0</td>
      <td>7.0</td>
      <td>20.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Sakura-jima</td>
      <td>Kyushu-Japan</td>
      <td>Japan</td>
      <td>31.580</td>
      <td>130.670</td>
      <td>1117.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1169.0</td>
      <td>2.0</td>
      <td>4.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Etna</td>
      <td>Italy</td>
      <td>Italy</td>
      <td>37.734</td>
      <td>15.004</td>
      <td>3350.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1565.0</td>
      <td>8.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Pacaya</td>
      <td>Guatemala</td>
      <td>Guatemala</td>
      <td>14.381</td>
      <td>-90.601</td>
      <td>2552.0</td>
      <td>Complex volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>5</th>
      <td>1600.0</td>
      <td>2.0</td>
      <td>19.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Huaynaputina</td>
      <td>Peru</td>
      <td>Peru</td>
      <td>-16.608</td>
      <td>-70.850</td>
      <td>4850.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>6</th>
      <td>1631.0</td>
      <td>2.0</td>
      <td>14.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Dama Ali</td>
      <td>Africa-NE</td>
      <td>Ethiopia</td>
      <td>11.280</td>
      <td>41.630</td>
      <td>1068.0</td>
      <td>Shield volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>7</th>
      <td>1631.0</td>
      <td>12.0</td>
      <td>16.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Vesuvius</td>
      <td>Italy</td>
      <td>Italy</td>
      <td>40.821</td>
      <td>14.426</td>
      <td>1281.0</td>
      <td>Complex volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>8</th>
      <td>1640.0</td>
      <td>7.0</td>
      <td>31.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Komaga-take</td>
      <td>Hokkaido-Japan</td>
      <td>Japan</td>
      <td>42.070</td>
      <td>140.680</td>
      <td>1140.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>9</th>
      <td>1659.0</td>
      <td>9.0</td>
      <td>30.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>San Salvador</td>
      <td>El Salvador</td>
      <td>El Salvador</td>
      <td>13.736</td>
      <td>-89.286</td>
      <td>1893.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>10</th>
      <td>1669.0</td>
      <td>3.0</td>
      <td>11.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Etna</td>
      <td>Italy</td>
      <td>Italy</td>
      <td>37.734</td>
      <td>15.004</td>
      <td>3350.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>11</th>
      <td>1679.0</td>
      <td>9.0</td>
      <td>21.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Zukur</td>
      <td>Red Sea</td>
      <td>Yemen</td>
      <td>14.020</td>
      <td>42.750</td>
      <td>624.0</td>
      <td>Shield volcano</td>
      <td>Holocene</td>
    </tr>
    <tr>
      <th>12</th>
      <td>1693.0</td>
      <td>1.0</td>
      <td>9.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Etna</td>
      <td>Italy</td>
      <td>Italy</td>
      <td>37.734</td>
      <td>15.004</td>
      <td>3350.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>13</th>
      <td>1707.0</td>
      <td>12.0</td>
      <td>16.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Fuji</td>
      <td>Honshu-Japan</td>
      <td>Japan</td>
      <td>35.350</td>
      <td>138.730</td>
      <td>3776.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>14</th>
      <td>1716.0</td>
      <td>9.0</td>
      <td>24.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Taal</td>
      <td>Luzon-Philippines</td>
      <td>Philippines</td>
      <td>14.002</td>
      <td>120.993</td>
      <td>400.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1741.0</td>
      <td>8.0</td>
      <td>23.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Oshima-Oshima</td>
      <td>Hokkaido-Japan</td>
      <td>Japan</td>
      <td>41.500</td>
      <td>139.370</td>
      <td>737.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>16</th>
      <td>1749.0</td>
      <td>8.0</td>
      <td>11.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Taal</td>
      <td>Luzon-Philippines</td>
      <td>Philippines</td>
      <td>14.002</td>
      <td>120.993</td>
      <td>400.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>17</th>
      <td>1754.0</td>
      <td>5.0</td>
      <td>13.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Taal</td>
      <td>Luzon-Philippines</td>
      <td>Philippines</td>
      <td>14.002</td>
      <td>120.993</td>
      <td>400.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>18</th>
      <td>1757.0</td>
      <td>7.0</td>
      <td>9.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>San Jorge</td>
      <td>Azores</td>
      <td>Portugal</td>
      <td>38.650</td>
      <td>-28.080</td>
      <td>1053.0</td>
      <td>Fissure vent</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>19</th>
      <td>1792.0</td>
      <td>5.0</td>
      <td>21.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Unzen</td>
      <td>Kyushu-Japan</td>
      <td>Japan</td>
      <td>32.750</td>
      <td>130.300</td>
      <td>1500.0</td>
      <td>Complex volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>20</th>
      <td>1820.0</td>
      <td>3.0</td>
      <td>1.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Westdahl</td>
      <td>Aleutian Is</td>
      <td>United States</td>
      <td>54.520</td>
      <td>-164.650</td>
      <td>1654.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1827.0</td>
      <td>6.0</td>
      <td>27.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Avachinsky</td>
      <td>Kamchatka</td>
      <td>Russia</td>
      <td>53.255</td>
      <td>158.830</td>
      <td>2741.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>22</th>
      <td>1837.0</td>
      <td>9.0</td>
      <td>NaN</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Peuet Sague</td>
      <td>Sumatra</td>
      <td>Indonesia</td>
      <td>4.914</td>
      <td>96.329</td>
      <td>2801.0</td>
      <td>Complex volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>23</th>
      <td>1840.0</td>
      <td>2.0</td>
      <td>2.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Gamalama</td>
      <td>Halmahera-Indonesia</td>
      <td>Indonesia</td>
      <td>0.800</td>
      <td>127.325</td>
      <td>1715.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>24</th>
      <td>1845.0</td>
      <td>2.0</td>
      <td>8.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Soputan</td>
      <td>Sulawesi-Indonesia</td>
      <td>Indonesia</td>
      <td>1.108</td>
      <td>124.725</td>
      <td>1784.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>25</th>
      <td>1857.0</td>
      <td>4.0</td>
      <td>17.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Umboi</td>
      <td>New Guinea-NE of</td>
      <td>Papua New Guinea</td>
      <td>-5.589</td>
      <td>147.875</td>
      <td>1548.0</td>
      <td>Complex volcano</td>
      <td>Holocene</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1863.0</td>
      <td>8.0</td>
      <td>17.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Yasur</td>
      <td>Vanuatu-SW Pacific</td>
      <td>Vanuatu</td>
      <td>-19.520</td>
      <td>169.425</td>
      <td>361.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>27</th>
      <td>1868.0</td>
      <td>4.0</td>
      <td>3.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Mauna Loa</td>
      <td>Hawaiian Is</td>
      <td>United States</td>
      <td>19.475</td>
      <td>-155.608</td>
      <td>4170.0</td>
      <td>Shield volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>28</th>
      <td>1868.0</td>
      <td>9.0</td>
      <td>5.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Etna</td>
      <td>Italy</td>
      <td>Italy</td>
      <td>37.734</td>
      <td>15.004</td>
      <td>3350.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>29</th>
      <td>1871.0</td>
      <td>4.0</td>
      <td>30.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Camiguin</td>
      <td>Mindanao-Philippines</td>
      <td>Philippines</td>
      <td>9.203</td>
      <td>124.673</td>
      <td>1332.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>30</th>
      <td>1877.0</td>
      <td>2.0</td>
      <td>14.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Mauna Loa</td>
      <td>Hawaiian Is</td>
      <td>United States</td>
      <td>19.475</td>
      <td>-155.608</td>
      <td>4170.0</td>
      <td>Shield volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>31</th>
      <td>1878.0</td>
      <td>2.0</td>
      <td>11.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Yasur</td>
      <td>Vanuatu-SW Pacific</td>
      <td>Vanuatu</td>
      <td>-19.520</td>
      <td>169.425</td>
      <td>361.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>32</th>
      <td>1878.0</td>
      <td>8.0</td>
      <td>29.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Okmok</td>
      <td>Aleutian Is</td>
      <td>United States</td>
      <td>53.420</td>
      <td>-168.130</td>
      <td>1073.0</td>
      <td>Shield volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>33</th>
      <td>1885.0</td>
      <td>5.0</td>
      <td>25.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Purace</td>
      <td>Colombia</td>
      <td>Colombia</td>
      <td>2.320</td>
      <td>-76.400</td>
      <td>4650.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>34</th>
      <td>1889.0</td>
      <td>9.0</td>
      <td>6.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Banua Wuhu</td>
      <td>Sangihe Is-Indonesia</td>
      <td>Indonesia</td>
      <td>3.138</td>
      <td>125.491</td>
      <td>-5.0</td>
      <td>Submarine volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>35</th>
      <td>1901.0</td>
      <td>8.0</td>
      <td>9.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Epi</td>
      <td>Vanuatu-SW Pacific</td>
      <td>Vanuatu</td>
      <td>-16.680</td>
      <td>168.370</td>
      <td>833.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>36</th>
      <td>1909.0</td>
      <td>4.0</td>
      <td>28.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Cameroon, Mt.</td>
      <td>Africa-W</td>
      <td>Cameroon</td>
      <td>4.203</td>
      <td>9.170</td>
      <td>4095.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>37</th>
      <td>1911.0</td>
      <td>1.0</td>
      <td>30.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Taal</td>
      <td>Luzon-Philippines</td>
      <td>Philippines</td>
      <td>14.002</td>
      <td>120.993</td>
      <td>400.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>38</th>
      <td>1913.0</td>
      <td>3.0</td>
      <td>14.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Awu</td>
      <td>Sangihe Is-Indonesia</td>
      <td>Indonesia</td>
      <td>3.670</td>
      <td>125.500</td>
      <td>1320.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>39</th>
      <td>1914.0</td>
      <td>1.0</td>
      <td>12.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Sakura-jima</td>
      <td>Kyushu-Japan</td>
      <td>Japan</td>
      <td>31.580</td>
      <td>130.670</td>
      <td>1117.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>40</th>
      <td>1917.0</td>
      <td>6.0</td>
      <td>7.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>San Salvador</td>
      <td>El Salvador</td>
      <td>El Salvador</td>
      <td>13.736</td>
      <td>-89.286</td>
      <td>1893.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>41</th>
      <td>1933.0</td>
      <td>1.0</td>
      <td>8.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Kharimkotan</td>
      <td>Kuril Is</td>
      <td>Russia</td>
      <td>49.120</td>
      <td>154.508</td>
      <td>1145.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>42</th>
      <td>1937.0</td>
      <td>5.0</td>
      <td>29.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Rabaul</td>
      <td>New Britain-SW Pac</td>
      <td>Papua New Guinea</td>
      <td>-4.271</td>
      <td>152.203</td>
      <td>688.0</td>
      <td>Pyroclastic shield</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>43</th>
      <td>1951.0</td>
      <td>8.0</td>
      <td>3.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Cosiguina</td>
      <td>Nicaragua</td>
      <td>Nicaragua</td>
      <td>12.980</td>
      <td>-87.570</td>
      <td>872.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>44</th>
      <td>1957.0</td>
      <td>3.0</td>
      <td>11.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Vsevidof</td>
      <td>Aleutian Is</td>
      <td>United States</td>
      <td>53.130</td>
      <td>-168.680</td>
      <td>2149.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>45</th>
      <td>1960.0</td>
      <td>5.0</td>
      <td>25.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Puyehue</td>
      <td>Chile-C</td>
      <td>Chile</td>
      <td>-40.590</td>
      <td>-72.117</td>
      <td>2236.0</td>
      <td>Stratovolcano</td>
      <td>Holocene</td>
    </tr>
    <tr>
      <th>46</th>
      <td>1963.0</td>
      <td>5.0</td>
      <td>16.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Agung</td>
      <td>Lesser Sunda Is</td>
      <td>Indonesia</td>
      <td>-8.342</td>
      <td>115.508</td>
      <td>3142.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>47</th>
      <td>1975.0</td>
      <td>11.0</td>
      <td>29.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Kilauea</td>
      <td>Hawaiian Is</td>
      <td>United States</td>
      <td>19.425</td>
      <td>-155.292</td>
      <td>1222.0</td>
      <td>Shield volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>48</th>
      <td>1980.0</td>
      <td>5.0</td>
      <td>18.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>St. Helens</td>
      <td>US-Washington</td>
      <td>United States</td>
      <td>46.200</td>
      <td>-122.180</td>
      <td>2549.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>49</th>
      <td>1982.0</td>
      <td>3.0</td>
      <td>28.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Chichon, El</td>
      <td>Mexico</td>
      <td>Mexico</td>
      <td>17.360</td>
      <td>-93.228</td>
      <td>1150.0</td>
      <td>Tuff cone</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>50</th>
      <td>1983.0</td>
      <td>10.0</td>
      <td>3.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Miyake-jima</td>
      <td>Izu Is-Japan</td>
      <td>Japan</td>
      <td>34.080</td>
      <td>139.530</td>
      <td>815.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>51</th>
      <td>1987.0</td>
      <td>12.0</td>
      <td>1.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Sirung</td>
      <td>Lesser Sunda Is</td>
      <td>Indonesia</td>
      <td>-8.510</td>
      <td>124.148</td>
      <td>862.0</td>
      <td>Complex volcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>52</th>
      <td>1991.0</td>
      <td>6.0</td>
      <td>15.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Pinatubo</td>
      <td>Luzon-Philippines</td>
      <td>Philippines</td>
      <td>15.130</td>
      <td>120.350</td>
      <td>1486.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>53</th>
      <td>2000.0</td>
      <td>6.0</td>
      <td>27.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Miyake-jima</td>
      <td>Izu Is-Japan</td>
      <td>Japan</td>
      <td>34.080</td>
      <td>139.530</td>
      <td>815.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>54</th>
      <td>2002.0</td>
      <td>8.0</td>
      <td>28.0</td>
      <td>NaN</td>
      <td>EQ</td>
      <td>Etna</td>
      <td>Italy</td>
      <td>Italy</td>
      <td>37.734</td>
      <td>15.004</td>
      <td>3350.0</td>
      <td>Stratovolcano</td>
      <td>Historical</td>
    </tr>
    <tr>
      <th>55</th>
      <td>2010.0</td>
      <td>5.0</td>
      <td>29.0</td>
      <td>TSU</td>
      <td>EQ</td>
      <td>Sarigan</td>
      <td>Mariana Is-C Pacific</td>
      <td>United States</td>
      <td>16.708</td>
      <td>145.780</td>
      <td>538.0</td>
      <td>Stratovolcano</td>
      <td>Holocene</td>
    </tr>
  </tbody>
</table>
</div>




```python
def plot_map2(lons, lats, elevations, llcrnrlat = -80, urcrnrlat = 90, llcrnrlon = -180, urcrnrlon = 180,resolution = 'i', projection='mill', lat_0 = 39.5, lon_0 = 1,min_marker_size=5):
    bins = np.linspace(0, elevations.max(), 10)
    marker_sizes = np.digitize(elevations, bins) + min_marker_size
    m2 = Basemap(projection=projection, llcrnrlat=llcrnrlat, urcrnrlat=urcrnrlat, llcrnrlon=llcrnrlon, urcrnrlon=urcrnrlon, resolution=resolution)
    m2.drawcountries()
    m2.drawmapboundary(fill_color='lightskyblue')
    m2.fillcontinents(color = '#ddaa66',lake_color='aqua')
    m2.drawcoastlines()

    for lon, lat, m2size in zip(lons, lats, marker_sizes):
        x, y = m2(lon, lat)
        m2.plot(x, y, 'bs', markersize=m2size, alpha=.7, zorder=4)

    return m2

def plot_map1(lons, lats, elevations, llcrnrlat=-80, urcrnrlat=90, llcrnrlon=-180, urcrnrlon=180,resolution='i', projection='mill', lat_0 = 39.5, lon_0 = 1,min_marker_size=2):
    bins = np.linspace(0, elevations.max(), 10)
    marker_sizes = np.digitize(elevations, bins) + min_marker_size
    m = Basemap(projection=projection, llcrnrlat=llcrnrlat, urcrnrlat=urcrnrlat, llcrnrlon=llcrnrlon, urcrnrlon=urcrnrlon, resolution=resolution)
    m.drawcountries()
    m.drawmapboundary(fill_color='lightskyblue')
    m.fillcontinents(color = '#ddaa66',lake_color='aqua')
    m.drawcoastlines()

    for lon, lat, msize in zip(lons, lats, marker_sizes):
        x, y = m(lon, lat)
        m.plot(x, y, '^r', markersize=msize, alpha=.7, zorder=4)

    return m

plt.figure(figsize=(60, 30))
m2 = plot_map2(data['Longitude'], data['Latitude'], data['Elevation'], min_marker_size=35)
m = plot_map1(df_volc['Longitude'], df_volc['Latitude'], df_volc['Elevation (m)'], min_marker_size=10)


plt.title('Volcano Eruptions with Associated Earthquakes', color='#000000', fontsize=50)

plt.show()
```


![png](output_1888_0.png)



----

In the original NOAA dataset, there are 797 volcanic eruption observations, and 55 of them are eruptions associated with earthquakes. Taking this into account from this dataset (Volcanic eruptions from 1790-2016), <b>6.9% of the volcanic eruptions from the NOAA dataset, had an association with an earthquake</b>.

-------

The red triangles indicate the volcanos, and the blue squares indicate the volcanos who had an association with an earthquake prior to its eruption. Out of 1500 volcanos, there were about 55 volcanic eruptions that had this association. Many have these occurred in the 20th century. We also see that the majority of these earthquake and volcano association have happened along the ring of fire, which stretches along the Eastern edge of Asia, down to New Zealand, as well as from Alaska down to South America.

-----------

<h1><center><u>Closer Examination of Volcano Eruptions with Associated Earthquakes</h1></u></center>

Let's examine the different types of volcanos as well as the top 10 countries that had the most volcanic eruptions with associated earthquakes. Is there a particular region that had the most volcano eruptions?


```python
import matplotlib.pyplot as plt; plt.rcdefaults()
import numpy as np
import matplotlib.pyplot as plt
```

<h3>Is there a type of Volcano that is more frequent with eruptions?</h3>


```python
objects = ('Stratovolcano', 'Shield Volcano', 'Complex Volcano', 'Pyroclastic shield', 'Tuff cone', 'Fissure vent','Submarine volcano')
y_pos = np.arange(len(objects))
performance = [38,7,6,1,1,1,1]
plt.barh(y_pos, performance, align='center', alpha=0.5)
plt.yticks(y_pos, objects)
plt.xlabel('Amount')
plt.title('Variation of Volcano Types with Associated Earthquakes')
plt.show()
```


![png](output_25_0.png)


We see that stratovolcanos (for instance Mount St.Helens, is a stratovolcano) had the overall highest frequency of volcanic eruptions, and by a large proportion.

<h3> Which country has had volcanic eruptions the most?</h3>


```python
data['Country'].value_counts()[:10].plot(kind = 'barh', title = 'Top 10 Countries with Volcanic Eruptions with Associated Earthquakes')
plt.show()
```


![png](output_288_0.png)


We see that the United States and Japan have an equal amount of volcanic eruptions that had associations with earthquakes.

<h5>Due to the lack of magnitude observation, from the NOAA data (which gave an option of volcanic eruptions with association of earthquakes), a goal is to have a better observation of more detailed variables to help establish a correlatiopn between earthquakes and volcanic eruptions. However, because this is a topic that scientists are still debating, and many do not see an exact correlation between the two, we will take a different approach that may lead us to answers that we are looking for, which is establishing a correlation between earthquakes and volcanos.

<h2><center>Using data concerning earthquakes occurring close to volcanos</h2>

<h1><center><u>Examining link between Earthquakes and Volcanic eruptions</u></center></h1>

As stated before, scientists still are debating whether earthquakes and volcanic eruptions are connected or not, and there is a lack of information available that proved that the two are substantially linked to one or the other. However, I have found enough data indicating that earthquakes do occur near volcanos, which can suggest that it is *possible* for earthquakes and volcanos to be somewhat linked.

<h2>Is it possible for earthquakes and volcanos to come into close contact with one another?<h2>


```python
import os
os.chdir('C:\Users\jenat\\Documents\\ringoffire')
import pandas as pd
import numpy as np
from mpl_toolkits.basemap import Basemap
import matplotlib.pyplot as plt
import matplotlib as mpl

```


```python
os.chdir('C:\Users\jenat\Documents\\ringoffire')
eqdata = pd.read_csv('earthquakesdata.csv')#dataset
eqdata1 = eqdata.convert_objects(convert_numeric=True)
```

    C:\Users\jenat\Anaconda2\lib\site-packages\ipykernel\__main__.py:3: FutureWarning: convert_objects is deprecated.  Use the data-type specific converters pd.to_datetime, pd.to_timedelta and pd.to_numeric.
      app.launch_new_instance()


<h2><center><u>Feb and March 2017 Earthquakes near Volcano data:</u></h2></center>


```python
eqdata1
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Time</th>
      <th>Mag</th>
      <th>Depth</th>
      <th>Location</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sat, 18 ar 19:47 UTC</td>
      <td>2.3</td>
      <td>13.2</td>
      <td>- 3 SSW of Volcano, Hawaii</td>
      <td>19.4000</td>
      <td>-155.2500</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Sat, 18 ar 14:48 UTC</td>
      <td>1.9</td>
      <td>17.6</td>
      <td>11  SSW fro Corinth</td>
      <td>19.3975</td>
      <td>-155.2522</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sat, 18 ar 13:57 UTC</td>
      <td>1.6</td>
      <td>2.2</td>
      <td>4.6  SSW of Her�ubrei�</td>
      <td>37.7902</td>
      <td>14.9158</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sat, 18 ar 13:13 UTC</td>
      <td>2.3</td>
      <td>1</td>
      <td>016  S 66? W of Wao (Lanao Del Sur)</td>
      <td>37.8527</td>
      <td>22.8490</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Sat, 18 ar 12:57 UTC</td>
      <td>2.2</td>
      <td>27.7</td>
      <td>- 5 NNW of Volcano, Hawaii</td>
      <td>65.1360</td>
      <td>-16.3860</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Sat, 18 ar 12:29 UTC</td>
      <td>1.8</td>
      <td>1.8</td>
      <td>- 5 WSW of Volcano, Hawaii</td>
      <td>7.5900</td>
      <td>124.6200</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Sat, 18 ar 12:08 UTC</td>
      <td>1.6</td>
      <td>0.7</td>
      <td>2.7  ESE of Go�abunga</td>
      <td>19.4768</td>
      <td>-155.2662</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Sat, 18 ar 11:41 UTC</td>
      <td>1.8</td>
      <td>4</td>
      <td>3.7  SW of Her�ubrei�</td>
      <td>19.4047</td>
      <td>-155.2835</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Sat, 18 ar 11:27 UTC</td>
      <td>3</td>
      <td>17</td>
      <td>012  S 87? W of Wao (Lanao Del Sur)I FELT IT</td>
      <td>19.4372</td>
      <td>-155.6165</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Sat, 18 ar 11:20 UTC</td>
      <td>1.5</td>
      <td>4</td>
      <td>4.1  SW of Her�ubrei�</td>
      <td>63.6350</td>
      <td>-19.1960</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Sat, 18 ar 10:51 UTC</td>
      <td>4.7</td>
      <td>10</td>
      <td>Northern Suatra, IndonesiaI FELT IT</td>
      <td>65.1510</td>
      <td>-16.4060</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Sat, 18 ar 10:43 UTC</td>
      <td>1.6</td>
      <td>3.3</td>
      <td>3.8  SW of Her�ubrei�</td>
      <td>7.6400</td>
      <td>124.6500</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Sat, 18 ar 10:07 UTC</td>
      <td>2.3</td>
      <td>3.1</td>
      <td>3.1  SW of Her�ubrei�</td>
      <td>65.1460</td>
      <td>-16.4070</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Sat, 18 ar 10:07 UTC</td>
      <td>1.6</td>
      <td>6.4</td>
      <td>4.4  SW of Her�ubrei�</td>
      <td>3.4200</td>
      <td>98.4800</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Sat, 18 ar 09:51 UTC</td>
      <td>1.6</td>
      <td>7.7</td>
      <td>5.7  SW of Her�ubrei�</td>
      <td>65.1500</td>
      <td>-16.4070</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Sat, 18 ar 09:28 UTC</td>
      <td>1.6</td>
      <td>4.8</td>
      <td>5.3  SW of Her�ubrei�</td>
      <td>65.1570</td>
      <td>-16.4030</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Sat, 18 ar 09:25 UTC</td>
      <td>2</td>
      <td>4.8</td>
      <td>5.3  SW of Her�ubrei�</td>
      <td>65.1460</td>
      <td>-16.4140</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Sat, 18 ar 09:25 UTC</td>
      <td>1.5</td>
      <td>9.2</td>
      <td>5.1  N of Her�ubrei�art�gl</td>
      <td>65.1370</td>
      <td>-16.4340</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Sat, 18 ar 08:59 UTC</td>
      <td>1.5</td>
      <td>7.6</td>
      <td>3.1  N of B�r�arbunga</td>
      <td>65.1370</td>
      <td>-16.4200</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Sat, 18 ar 08:44 UTC</td>
      <td>1.6</td>
      <td>3.8</td>
      <td>- 11 WNW of Calipatria, CA</td>
      <td>65.1350</td>
      <td>-16.4140</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Sat, 18 ar 08:40 UTC</td>
      <td>2.1</td>
      <td>3.4</td>
      <td>4.0  SW of Her�ubrei�</td>
      <td>65.1330</td>
      <td>-16.3990</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Sat, 18 ar 08:26 UTC</td>
      <td>2.5</td>
      <td>4</td>
      <td>SOUTHERN CALIFORNIA</td>
      <td>64.6680</td>
      <td>-17.5160</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Sat, 18 ar 08:26 UTC</td>
      <td>1.5</td>
      <td>4.8</td>
      <td>4.6  SW of Her�ubrei�</td>
      <td>33.1607</td>
      <td>-115.6203</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Sat, 18 ar 06:47 UTC</td>
      <td>1.5</td>
      <td>6.6</td>
      <td>4.8  SW of Her�ubrei�</td>
      <td>65.1480</td>
      <td>-16.4070</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Sat, 18 ar 06:22 UTC</td>
      <td>2.9</td>
      <td>7.1</td>
      <td>5.1  SW of Her�ubrei�I FELT IT</td>
      <td>33.1500</td>
      <td>-115.6300</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Sat, 18 ar 06:22 UTC</td>
      <td>2.1</td>
      <td>3.3</td>
      <td>5.0  SW of Her�ubrei�</td>
      <td>65.1440</td>
      <td>-16.4170</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Sat, 18 ar 05:28 UTC</td>
      <td>2.4</td>
      <td>5.1</td>
      <td>4.9  SW of Her�ubrei�</td>
      <td>65.1450</td>
      <td>-16.4230</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Sat, 18 ar 05:28 UTC</td>
      <td>2.2</td>
      <td>5.1</td>
      <td>4.9  SW of Her�ubrei�</td>
      <td>65.1410</td>
      <td>-16.4240</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Sat, 18 ar 05:05 UTC</td>
      <td>2</td>
      <td>26.4</td>
      <td>- 5 NW of Volcano, Hawaii</td>
      <td>65.1420</td>
      <td>-16.4240</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Sat, 18 ar 04:37 UTC</td>
      <td>1.5</td>
      <td>3.6</td>
      <td>3.8  SW of Her�ubrei�</td>
      <td>65.1440</td>
      <td>-16.4250</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>826</th>
      <td>Thu, 2 Feb 18:04 UTC</td>
      <td>3</td>
      <td>7</td>
      <td>SOUTHERN GREECE</td>
      <td>-39.2588</td>
      <td>173.9287</td>
    </tr>
    <tr>
      <th>827</th>
      <td>Thu, 2 Feb 12:47 UTC</td>
      <td>3.3</td>
      <td>8</td>
      <td>16  al Norte de Cascajal, V. de Coronado.</td>
      <td>19.3812</td>
      <td>-155.2410</td>
    </tr>
    <tr>
      <th>828</th>
      <td>Thu, 2 Feb 06:52 UTC</td>
      <td>2.1</td>
      <td>1.2</td>
      <td>- 128 NNW of Kodiak Station, Alaska</td>
      <td>58.3636</td>
      <td>-154.7016</td>
    </tr>
    <tr>
      <th>829</th>
      <td>Thu, 2 Feb 04:50 UTC</td>
      <td>1.9</td>
      <td>3</td>
      <td>Alaska</td>
      <td>19.3073</td>
      <td>-155.2138</td>
    </tr>
    <tr>
      <th>830</th>
      <td>Thu, 2 Feb 01:37 UTC</td>
      <td>2.4</td>
      <td>5.9</td>
      <td>- 123 NNW of Kodiak Station, Alaska</td>
      <td>-39.4653</td>
      <td>175.7146</td>
    </tr>
    <tr>
      <th>831</th>
      <td>Wed, 1 Feb 21:37 UTC</td>
      <td>1.9</td>
      <td>9.3</td>
      <td>- 119 SE of Old Iliana, Alaska</td>
      <td>55.6660</td>
      <td>160.3470</td>
    </tr>
    <tr>
      <th>832</th>
      <td>Wed, 1 Feb 21:33 UTC</td>
      <td>2.3</td>
      <td>8.5</td>
      <td>- 127 SE of Old Iliana, Alaska</td>
      <td>38.8077</td>
      <td>-122.7707</td>
    </tr>
    <tr>
      <th>833</th>
      <td>Wed, 1 Feb 21:29 UTC</td>
      <td>1.9</td>
      <td>14.8</td>
      <td>Catania</td>
      <td>55.6980</td>
      <td>160.4760</td>
    </tr>
    <tr>
      <th>834</th>
      <td>Wed, 1 Feb 18:52 UTC</td>
      <td>2.1</td>
      <td>3.1</td>
      <td>Avellino</td>
      <td>37.5500</td>
      <td>23.5900</td>
    </tr>
    <tr>
      <th>835</th>
      <td>Wed, 1 Feb 18:24 UTC</td>
      <td>2</td>
      <td>1.2</td>
      <td>Catania</td>
      <td>10.1290</td>
      <td>-83.9620</td>
    </tr>
    <tr>
      <th>836</th>
      <td>Wed, 1 Feb 17:58 UTC</td>
      <td>2.3</td>
      <td>23.5</td>
      <td>14.4  SW fro Leni (E)</td>
      <td>58.7621</td>
      <td>-153.6923</td>
    </tr>
    <tr>
      <th>837</th>
      <td>Wed, 1 Feb 16:43 UTC</td>
      <td>2.4</td>
      <td>1</td>
      <td>058  N 45? E of  Davao City</td>
      <td>58.8027</td>
      <td>-153.8385</td>
    </tr>
    <tr>
      <th>838</th>
      <td>Wed, 1 Feb 16:21 UTC</td>
      <td>2</td>
      <td>2</td>
      <td>NORTHERN CALIFORNIA</td>
      <td>58.7243</td>
      <td>-153.6634</td>
    </tr>
    <tr>
      <th>839</th>
      <td>Wed, 1 Feb 14:23 UTC</td>
      <td>2.1</td>
      <td>2.8</td>
      <td>- 7 SW of Volcano, Hawaii</td>
      <td>58.9080</td>
      <td>-153.6289</td>
    </tr>
    <tr>
      <th>840</th>
      <td>Wed, 1 Feb 14:19 UTC</td>
      <td>2.3</td>
      <td>1.9</td>
      <td>- 2 SSW of Cobb, California</td>
      <td>58.8481</td>
      <td>-153.5432</td>
    </tr>
    <tr>
      <th>841</th>
      <td>Wed, 1 Feb 13:25 UTC</td>
      <td>1.9</td>
      <td>11.5</td>
      <td>21  SSE fro Aigina</td>
      <td>37.6653</td>
      <td>14.9807</td>
    </tr>
    <tr>
      <th>842</th>
      <td>Wed, 1 Feb 13:18 UTC</td>
      <td>2.6</td>
      <td>3</td>
      <td>ISLAND OF HAWAII, HAWAII</td>
      <td>40.8987</td>
      <td>14.6692</td>
    </tr>
    <tr>
      <th>843</th>
      <td>Wed, 1 Feb 12:33 UTC</td>
      <td>2.1</td>
      <td>5</td>
      <td>1.5  ENE of Kr�suv�k</td>
      <td>37.7540</td>
      <td>15.0060</td>
    </tr>
    <tr>
      <th>844</th>
      <td>Wed, 1 Feb 11:24 UTC</td>
      <td>2.4</td>
      <td>12</td>
      <td>SOUTHERN GREECE</td>
      <td>38.4690</td>
      <td>14.7060</td>
    </tr>
    <tr>
      <th>845</th>
      <td>Wed, 1 Feb 10:40 UTC</td>
      <td>3</td>
      <td>4</td>
      <td>8  al Norte de Capellades, Alvarado.</td>
      <td>7.4800</td>
      <td>125.9900</td>
    </tr>
    <tr>
      <th>846</th>
      <td>Wed, 1 Feb 09:59 UTC</td>
      <td>2.2</td>
      <td>5.2</td>
      <td>New Zealand</td>
      <td>38.7600</td>
      <td>-122.7300</td>
    </tr>
    <tr>
      <th>847</th>
      <td>Wed, 1 Feb 09:47 UTC</td>
      <td>2.3</td>
      <td>15</td>
      <td>Alaska</td>
      <td>19.3827</td>
      <td>-155.2812</td>
    </tr>
    <tr>
      <th>848</th>
      <td>Wed, 1 Feb 09:20 UTC</td>
      <td>2.7</td>
      <td>1</td>
      <td>SOUTHERN GREECE</td>
      <td>38.8025</td>
      <td>-122.7377</td>
    </tr>
    <tr>
      <th>849</th>
      <td>Wed, 1 Feb 08:16 UTC</td>
      <td>2.8</td>
      <td>0.2</td>
      <td>- 96 NNW of Nikiski, Alaska</td>
      <td>37.5725</td>
      <td>23.5370</td>
    </tr>
    <tr>
      <th>850</th>
      <td>Wed, 1 Feb 07:29 UTC</td>
      <td>2.1</td>
      <td>3</td>
      <td>NORTHERN CALIFORNIA</td>
      <td>19.3900</td>
      <td>-155.2800</td>
    </tr>
    <tr>
      <th>851</th>
      <td>Wed, 1 Feb 05:56 UTC</td>
      <td>1.9</td>
      <td>17</td>
      <td>Catania</td>
      <td>63.8930</td>
      <td>-22.0380</td>
    </tr>
    <tr>
      <th>852</th>
      <td>Wed, 1 Feb 02:32 UTC</td>
      <td>2.3</td>
      <td>3</td>
      <td>NORTHERN CALIFORNIA</td>
      <td>37.6000</td>
      <td>23.5100</td>
    </tr>
    <tr>
      <th>853</th>
      <td>Wed, 1 Feb 00:41 UTC</td>
      <td>2.3</td>
      <td>3</td>
      <td>ISLAND OF HAWAII, HAWAII</td>
      <td>9.9900</td>
      <td>-83.8030</td>
    </tr>
    <tr>
      <th>854</th>
      <td>Wed, 1 Feb 00:39 UTC</td>
      <td>2.1</td>
      <td>2</td>
      <td>NORTHERN CALIFORNIA</td>
      <td>-37.6903</td>
      <td>177.2383</td>
    </tr>
    <tr>
      <th>855</th>
      <td>Wed, 1 Feb 00:39 UTC</td>
      <td>2.8</td>
      <td>3</td>
      <td>ISLAND OF HAWAII, HAWAII</td>
      <td>61.4317</td>
      <td>-152.2931</td>
    </tr>
  </tbody>
</table>
<p>856 rows × 6 columns</p>
</div>



These are two small datasets consisting of earthquakes that have happened near volcanos since Feb 1-March 18th.  As we can see from these datasets, particularly the distance (km) from the volcano itself, we see that it is very likely that earthquakes and volcanos can come into close contact with another, thus the possibiltiy of volcanic eruptions and earthquakes occurring is a possibility, as it is proven in the first dataset. The question remains, how frequenly does it occur, and what causes it (two questions for Geologists!)

------------


```python
latlong = pd.read_csv('latlong.csv')
eqdata = pd.read_csv('earthquakesdata.csv')

#earth.Latitude
#earth.Longitude


def earth_near(lons, lats, magnitude, min_marker_size=2):
    bins = np.linspace(0, magnitude.max(), 10)
    marker_sizes = np.digitize(magnitude, bins) + min_marker_size

    m = Basemap()
    m.readshapefile('C:\Users\jenat\\Documents\\ringoffire\\new\\plate', 'plate')
    
    
    m.bluemarble(alpha=0.42)

    for lon, lat, msize in zip(lons, lats, marker_sizes):
        x, y = m(lon, lat)
        m.plot(x, y, '*', c='#fff8dc',markersize=msize, alpha=1.0, zorder=10)

    return m
    


```

<center><b><u>Legend for Plot</u></b>:</center><table><td><b><center>Symbol</b></center><td><b>Meaning</b><tr><td><b>*</b><td>Earthquake<tr><td><b>o</b><td>Volcano<tr><td><b>Line</b><td>Plate boundary</table></tr></td>


```python
plt.figure(figsize=(15, 12))
map1.scatter(x1,y1,c='red',marker="o",alpha=0.7)
m = earth_near(eqdata1['Longitude'], eqdata1['Latitude'], eqdata1['Mag'], min_marker_size=2)
plt.title('Earthquakes near Volcanos Since Feb 1', color='#000000', fontsize=40)
plt.show()
```


![png](output_42_0.png)


---------

We see that they are quite close to tetonic plates. The white stars are the earthquakes, and the red circles are the volcanos. As we see, the earthquakes are all quite close to the volcanos. In addition, the size of the stars is based upon the magnitude of the earthquake.

----------

<h2> Where are these earthquakes happening the most?</h2>


```python
import matplotlib.pyplot as plt; plt.rcdefaults()
import numpy as np
import matplotlib.pyplot as plt
```


```python
eqdata1['Location'].value_counts()[:10].plot(kind = 'barh', title = 'Top 10 Locations with Earthquakes near Volcanos since Feb 1')
plt.show()
```


![png](output_46_0.png)


For Top 3 (out of 10) : We see that New Zealand has had the most earthquakes, followed by the big island of Hawai'i, then Russia. We also see that Central California, southern California, Northern California (which should include the Geysers) also have a lot of activity as well.

------------------

<h2>Is there a specific magnitude that is happening more frequently?</h2>


```python
plt.figure()
plt.hist(eqdata1['Mag'].dropna(), bins = 20)
plt.xlabel('Magnitude')
plt.ylabel('Amount')
plt.title("Variation and Amount of Earthquake Magnitudes Since Feb 1")
plt.show()
```


![png](output_49_0.png)


Most of the earthquakes magnitudes are quite small, as in 2.5 or below.

-------------------

<h2>Is there a correlation between the depth of the earthquake and the magnitude of the earthquake?</h2>


```python
import matplotlib.pyplot
import pylab
import os
os.chdir('C:\Users\jenat\\Documents\\ringoffire')
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl

tes3 = pd.read_csv('earthquakesdata.csv',usecols = [1,2])#dataset
data1 = tes3.convert_objects(convert_numeric=True)
data1 = data1.rename(columns={' Depth': 'Depth'})

matplotlib.pyplot.scatter(data1.Mag,data1.Depth)
matplotlib.pyplot.title('Scatter Plot of Magnitudes and Depths of Earthquakes')
matplotlib.pyplot.xlabel("Magnitude")
matplotlib.pyplot.ylabel("Depth (M)")
matplotlib.pyplot.show()

```

    C:\Users\jenat\Anaconda2\lib\site-packages\ipykernel\__main__.py:11: FutureWarning: convert_objects is deprecated.  Use the data-type specific converters pd.to_datetime, pd.to_timedelta and pd.to_numeric.



![png](output_52_1.png)


-----

As we can see, there is not a strong correlation between Magnitudes of earthquakes and the depths of the earthquakes. Most of the earthquakes from smaller magnitudes to the larger ones are typically within the same range of depth, which indicates that magnitude an depth are likely not correlated.

--------------------

<h3> Let's use a Spearman, non-parametric correlation test between Magnitude and Depth</h3>


```python
import os
os.chdir('C:\Users\jenat\\Documents\\ringoffire')
import pandas as pd

data1.corr()

data1.corr(method='spearman', min_periods=1)
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Mag</th>
      <th>Depth</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Mag</th>
      <td>1.000000</td>
      <td>0.139534</td>
    </tr>
    <tr>
      <th>Depth</th>
      <td>0.139534</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



The matrix correlation, using the spearman test concerning the two columns magnitude and Depth, indicates too that there is not a strong correlation between Magnitude and Depth. 

-------

-----

<h2>How close are the earthquakes happening (km) from the volcanos?:</h2>

When earthquakes occurr before a volcanic eruption, as seen in the 1980 eruption of Mount St.Helens, these earthquakes are caused by the movement of magma, from the earth's crust towards the mouth of the volcano. General earthquakes are caused by movement between two or more tetonic plates rubbing against each other.

with this in mind, we can speculate that the closer the earthquake occurs towards the volcano itself, the more we can speculate possible volcanic activity (should also keep in mind the history of the volcano itself and the last time it erupted and if it is in fact active)

With this in mind, we will look at the month of March 2017, and the distance from the volcanos the earthquakes occurred.


```python
import os
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib as mpl
os.chdir('C:\Users\jenat\\Documents\\ringoffire')
nv=pd.read_csv("march_near_volc.csv")
nv.columns = ['Volcano', 'Distance']
```


```python
nv
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Volcano</th>
      <th>Distance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Kilauea</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Bardarbunga</td>
      <td>5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Bardarbunga</td>
      <td>7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bardarbunga</td>
      <td>10</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Bardarbunga</td>
      <td>7</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Bardarbunga</td>
      <td>7</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Santo Tomas</td>
      <td>10</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Nisyros</td>
      <td>22</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Tongariro</td>
      <td>2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Paco</td>
      <td>21</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Kilauea</td>
      <td>2</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Katla</td>
      <td>4</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Katla</td>
      <td>3</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Grímsvötn</td>
      <td>13</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Süphan</td>
      <td>20</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Clear Lake</td>
      <td>20</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Unzen</td>
      <td>22</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Vesuvius</td>
      <td>8</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Askja</td>
      <td>23</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Kilauea</td>
      <td>23</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Panay</td>
      <td>7</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Panay</td>
      <td>6</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Clear Lake</td>
      <td>20</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Akyarlar</td>
      <td>13</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Lassen</td>
      <td>2</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Trident</td>
      <td>2</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Sabancaya</td>
      <td>8</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Ragang</td>
      <td>12</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Long Valley</td>
      <td>9</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Hakkoda</td>
      <td>10</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>366</th>
      <td>Etna</td>
      <td>14</td>
    </tr>
    <tr>
      <th>367</th>
      <td>Katla</td>
      <td>7</td>
    </tr>
    <tr>
      <th>368</th>
      <td>Mauna</td>
      <td>15</td>
    </tr>
    <tr>
      <th>369</th>
      <td>Kilauea</td>
      <td>23</td>
    </tr>
    <tr>
      <th>370</th>
      <td>Katla</td>
      <td>3</td>
    </tr>
    <tr>
      <th>371</th>
      <td>Kilauea</td>
      <td>18</td>
    </tr>
    <tr>
      <th>372</th>
      <td>Ruapehu</td>
      <td>22</td>
    </tr>
    <tr>
      <th>373</th>
      <td>Sabancaya</td>
      <td>12</td>
    </tr>
    <tr>
      <th>374</th>
      <td>Kilauea</td>
      <td>18</td>
    </tr>
    <tr>
      <th>375</th>
      <td>Clear</td>
      <td>20</td>
    </tr>
    <tr>
      <th>376</th>
      <td>Hrómundartindur</td>
      <td>5</td>
    </tr>
    <tr>
      <th>377</th>
      <td>Salton</td>
      <td>24</td>
    </tr>
    <tr>
      <th>378</th>
      <td>Salton</td>
      <td>23</td>
    </tr>
    <tr>
      <th>379</th>
      <td>Reykjanes</td>
      <td>7</td>
    </tr>
    <tr>
      <th>380</th>
      <td>Taranaki</td>
      <td>22</td>
    </tr>
    <tr>
      <th>381</th>
      <td>Clear Lake</td>
      <td>17</td>
    </tr>
    <tr>
      <th>382</th>
      <td>Abu</td>
      <td>12</td>
    </tr>
    <tr>
      <th>383</th>
      <td>Bardarbunga</td>
      <td>7</td>
    </tr>
    <tr>
      <th>384</th>
      <td>Bardarbunga</td>
      <td>7</td>
    </tr>
    <tr>
      <th>385</th>
      <td>Bardarbunga</td>
      <td>9</td>
    </tr>
    <tr>
      <th>386</th>
      <td>Bardarbunga</td>
      <td>8</td>
    </tr>
    <tr>
      <th>387</th>
      <td>Bardarbunga</td>
      <td>8</td>
    </tr>
    <tr>
      <th>388</th>
      <td>Reykjanes</td>
      <td>13</td>
    </tr>
    <tr>
      <th>389</th>
      <td>Bardarbunga</td>
      <td>10</td>
    </tr>
    <tr>
      <th>390</th>
      <td>Bardarbunga</td>
      <td>7</td>
    </tr>
    <tr>
      <th>391</th>
      <td>Bardarbunga</td>
      <td>8</td>
    </tr>
    <tr>
      <th>392</th>
      <td>Askja</td>
      <td>13</td>
    </tr>
    <tr>
      <th>393</th>
      <td>Reykjanes</td>
      <td>13</td>
    </tr>
    <tr>
      <th>394</th>
      <td>Baru</td>
      <td>6</td>
    </tr>
    <tr>
      <th>395</th>
      <td>Tjörnes Fracture Zone</td>
      <td>16</td>
    </tr>
  </tbody>
</table>
<p>396 rows × 2 columns</p>
</div>



<b>Let's see how many of the earthquakes happened less than 2.0 km from the volcano</b>:


```python
nv.Distance[nv.Distance< 2.0 ].count() 
```




    7



This indicates that <b>7 earthquakes</b>, so far in March 2017 were less than 2.0 km from a volcano.

More specifically:


```python
nv['Distance'].describe()
```




    count    396.000000
    mean      13.212121
    std        6.906644
    min        0.000000
    25%        7.000000
    50%       14.000000
    75%       20.000000
    max       24.000000
    Name: Distance, dtype: float64



As we see: out of the 396 earthquakes report for March 2017 that were documented to be near volcanos, 
the <b>mean</b> was 13.2, and <b>25%</b> of the earthquakes happened 7 km from a volcano, while <b>50%</b> of the earthquakes happened 14 km from a volcano, while <b>75%</b> of the earthquakes happened at least 20 km from a volcano.

We also see that the <b>max</b> was 24 km and the <b>min</b> was 0. The earthquake with 0 km was located in Mammoth Mountain which is in Southern California.


```python
plt.figure()
v_plot = nv['Distance'].hist(bins=20)
v_plot.set_title("Distribution of Earthquakes by their Distances from Volcanos")
v_plot.set_xlabel("Distance from Volcano (km)")
v_plot.set_ylabel("Number of Earthquakes")
plt.show()
```


![png](output_69_0.png)


Since we do not know whether the earthquakes are caused by magma movement or are simply regular earthquakes, we can not say whether these earthquakes are related to volcanic eruptions. However, typically earthquakes before a volcanic eruption happen in many clusters. However, ones that are very close in my mind would raise some speculation.

<b>CONCLUSION</b>

There is not enough scientific evidence, or data to link earthquakes and volcano eruptions as being statistically significant to one another. More specifically, if an Earthquake can cause a volcanic eruption. While scientists are still debating the connection between the two, there is evidence that earthquakes occur (and rather frequently) near volcanos. With that information given, this brings the possibility that it is possible for earthquakes and volcanos to correlate with one another.

Another aspect worth looking into, is determing which earthquake is an aftershock and which earthquake is not.


```python

```






