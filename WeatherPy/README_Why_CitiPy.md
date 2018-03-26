
Proving the randomness of data using CitiPy

The largest hurdle to using random latitiudes and longitudes to call on OpenWeatherMap I could see is that 71% of the earth is water.  Any lat/long that fell into these areas too far from the shore was likely to be returned by OpenWeatherMap as "not found".  By using CitiPy first, you can map the coordinates that fall into the ocean to the nearest city on land.  You might still get some cities that are "not found" on OpenWeatherMap, as well as duplicate cities (most likely due to being the closest city on land to two different sets of coordinates over the ocean).  But overall passing coordinates through CitiPy gives you cleaner data to pass on to OpenWeatherMap.  But do CitiPy's methods of locating the closest city keep the cities spread out over the world?


```python
import csv
import pandas as pd
```


```python
cities_df = pd.read_csv("WeatherPy.csv")
```

Out of 195 countries in the world, how many did our random coordinates find?


```python
cities_df["Country"].nunique()
```




    113




```python
continents_df = pd.read_csv("country_continent.csv")
continents_df = continents_df.rename(columns={"iso 3166 country":"Country","continent code":"Continent"})
continents_df.set_index('Country', inplace=True)
```


```python
coastal_df = cities_df[["Country","City ID"]]
coastal_df = coastal_df.groupby(["Country"]).count()
coastal_df = coastal_df.sort_values(["City ID"],ascending = False)
coastal_df = coastal_df.rename(columns={"City ID": "Cities in Random"})
```


```python
coastal_df = coastal_df.join(continents_df)
```

I would expect to see more cities from the largest countries, as well as those countries with the longest coastlines (referencing the ocean coordinates being mapped to land).


```python
rank = ['3rd','8th','1st','17th','6th','2nd','11th','unk','4th','14th']
mass = ['1st','3rd','2nd','5th','6th','unk','4th','8th','unk','14th']
x = 10
for x in range(103):
    #coastlines.append('unk')
    rank.append('unk')
    mass.append('unk')
    x = x + 1
#coastal_df["Coastline (in miles)"] = coastlines
coastal_df["Rank in World (coastline)"] = rank
coastal_df["Rank in World (mass)"] = mass
coastal_df.head(10) 
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
      <th>Cities in Random</th>
      <th>Continent</th>
      <th>Rank in World (coastline)</th>
      <th>Rank in World (mass)</th>
    </tr>
    <tr>
      <th>Country</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>RU</th>
      <td>62</td>
      <td>Europe</td>
      <td>3rd</td>
      <td>1st</td>
    </tr>
    <tr>
      <th>US</th>
      <td>48</td>
      <td>North America</td>
      <td>8th</td>
      <td>3rd</td>
    </tr>
    <tr>
      <th>CA</th>
      <td>36</td>
      <td>North America</td>
      <td>1st</td>
      <td>2nd</td>
    </tr>
    <tr>
      <th>BR</th>
      <td>28</td>
      <td>South America</td>
      <td>17th</td>
      <td>5th</td>
    </tr>
    <tr>
      <th>AU</th>
      <td>18</td>
      <td>Oceania</td>
      <td>6th</td>
      <td>6th</td>
    </tr>
    <tr>
      <th>ID</th>
      <td>15</td>
      <td>Asia</td>
      <td>2nd</td>
      <td>unk</td>
    </tr>
    <tr>
      <th>CN</th>
      <td>15</td>
      <td>Asia</td>
      <td>11th</td>
      <td>4th</td>
    </tr>
    <tr>
      <th>AR</th>
      <td>12</td>
      <td>South America</td>
      <td>unk</td>
      <td>8th</td>
    </tr>
    <tr>
      <th>PH</th>
      <td>11</td>
      <td>Asia</td>
      <td>4th</td>
      <td>unk</td>
    </tr>
    <tr>
      <th>MX</th>
      <td>10</td>
      <td>North America</td>
      <td>14th</td>
      <td>14th</td>
    </tr>
  </tbody>
</table>
</div>



But are the countries, regardless of size, spread across all regions of the earth?


```python
coastal_df["Continent"].value_counts()
```




    Africa           32
    Asia             26
    Europe           20
    North America    15
    South America    10
    Oceania          10
    Name: Continent, dtype: int64



CitiPy, using random lat/long generation, pulled cities from 6 of 7 continents.  Antarctica has no permenant cities, and thus no returns.  So I conclude that passing coordinates through CitiPy to make OpenWeatherMap calls easier, still gave us appropriately random and diverse data.
