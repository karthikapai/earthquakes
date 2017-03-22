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



