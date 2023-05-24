
### Guided Project: Analyzing CIA Factbook Data Using SQL
In this project, we'll work with data from the CIA World Factbook, a compendium of statistics about all of the countries on Earth

we'll use SQL in Jupyter Notebook to explore and analyze data from this database


```python
%%capture
%load_ext sql
%sql sqlite:///factbook.db
```




    'Connected: None@factbook.db'



To run SQL queries in this project we add %%sql on its own line to the start of our query


```python
%%sql
SELECT *
  FROM sqlite_master
 WHERE type='table';
```

    Done.





<table>
    <tr>
        <th>type</th>
        <th>name</th>
        <th>tbl_name</th>
        <th>rootpage</th>
        <th>sql</th>
    </tr>
    <tr>
        <td>table</td>
        <td>sqlite_sequence</td>
        <td>sqlite_sequence</td>
        <td>3</td>
        <td>CREATE TABLE sqlite_sequence(name,seq)</td>
    </tr>
    <tr>
        <td>table</td>
        <td>facts</td>
        <td>facts</td>
        <td>47</td>
        <td>CREATE TABLE &quot;facts&quot; (&quot;id&quot; INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, &quot;code&quot; varchar(255) NOT NULL, &quot;name&quot; varchar(255) NOT NULL, &quot;area&quot; integer, &quot;area_land&quot; integer, &quot;area_water&quot; integer, &quot;population&quot; integer, &quot;population_growth&quot; float, &quot;birth_rate&quot; float, &quot;death_rate&quot; float, &quot;migration_rate&quot; float)</td>
    </tr>
</table>



Write a query to return information on the tables in the database.
In a different code cell, write and run another query that returns the first 5 rows of the facts table in the database.


```python

%%sql
SELECT *
  FROM facts
 LIMIT 5;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>1</td>
        <td>af</td>
        <td>Afghanistan</td>
        <td>652230</td>
        <td>652230</td>
        <td>0</td>
        <td>32564342</td>
        <td>2.32</td>
        <td>38.57</td>
        <td>13.89</td>
        <td>1.51</td>
    </tr>
    <tr>
        <td>2</td>
        <td>al</td>
        <td>Albania</td>
        <td>28748</td>
        <td>27398</td>
        <td>1350</td>
        <td>3029278</td>
        <td>0.3</td>
        <td>12.92</td>
        <td>6.58</td>
        <td>3.3</td>
    </tr>
    <tr>
        <td>3</td>
        <td>ag</td>
        <td>Algeria</td>
        <td>2381741</td>
        <td>2381741</td>
        <td>0</td>
        <td>39542166</td>
        <td>1.84</td>
        <td>23.67</td>
        <td>4.31</td>
        <td>0.92</td>
    </tr>
    <tr>
        <td>4</td>
        <td>an</td>
        <td>Andorra</td>
        <td>468</td>
        <td>468</td>
        <td>0</td>
        <td>85580</td>
        <td>0.12</td>
        <td>8.13</td>
        <td>6.96</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>5</td>
        <td>ao</td>
        <td>Angola</td>
        <td>1246700</td>
        <td>1246700</td>
        <td>0</td>
        <td>19625353</td>
        <td>2.78</td>
        <td>38.78</td>
        <td>11.49</td>
        <td>0.46</td>
    </tr>
</table>



A few things stick out from the summary statistics in the last screen:

There's a country with a population of 0
There's a country with a population of 7256490011 (or more than 7.2 billion people)
Let's use subqueries to zoom in on just these countries without using the specific values.

Write a single query that returns the:

Minimum population
Maximum population
Minimum population growth
Maximum population growth


```python
%%sql
SELECT MIN(population), MAX(population),
MIN(population_growth), MAX(population_growth)
FROM facts;
```

    Done.





<table>
    <tr>
        <th>MIN(population)</th>
        <th>MAX(population)</th>
        <th>MIN(population_growth)</th>
        <th>MAX(population_growth)</th>
    </tr>
    <tr>
        <td>0</td>
        <td>7256490011</td>
        <td>0.0</td>
        <td>4.02</td>
    </tr>
</table>



Write a query that returns the countrie(s) with the minimum population.
Write a query that returns the countrie(s) with the maximum population.


```python
%%sql
SELECT name, population
FROM facts
WHERE population == (SELECT MIN(population)
                     FROM facts)
;
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
    </tr>
    <tr>
        <td>Antarctica</td>
        <td>0</td>
    </tr>
</table>




```python
%%sql
SELECT name, population
FROM facts
WHERE population == (SELECT MAX(population)
                     FROM facts)
;
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
    </tr>
    <tr>
        <td>World</td>
        <td>7256490011</td>
    </tr>
</table>



It seems like the table contains a row for the whole world, which explains the population of over 7.2 billion. It also seems like the table contains a row for Antarctica, which explains the population of 0.

Now that we know this, we should recalculate the summary statistics we calculated earlier, while excluding the row for the whole world.


```python
%%sql
SELECT MIN(population)Min_pop, MAX(population)Max_pop,
MIN(population_growth)Min_growth, MAX(population_growth)Max_growth
FROM facts
WHERE name<>'World'
;
```

    Done.





<table>
    <tr>
        <th>Min_pop</th>
        <th>Max_pop</th>
        <th>Min_growth</th>
        <th>Max_growth</th>
    </tr>
    <tr>
        <td>0</td>
        <td>1367485388</td>
        <td>0.0</td>
        <td>4.02</td>
    </tr>
</table>



Exploring the population and area


```python
%%sql
SELECT AVG(population)Avg_Pop, AVG(area)Area_Avg
FROM facts 
WHERE name<>'World'
```

    Done.





<table>
    <tr>
        <th>Avg_Pop</th>
        <th>Area_Avg</th>
    </tr>
    <tr>
        <td>32242666.56846473</td>
        <td>555093.546184739</td>
    </tr>
</table>



Identify countries that meet Above average values for population.
Below average values for area.


```python
%%sql
SELECT * 
FROM facts
WHERE population > (SELECT AVG(population)
                   FROM facts
                   WHERE name <> 'World')
AND area < (SELECT AVG(area)
           FROM facts
           WHERE name <> 'World');
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>14</td>
        <td>bg</td>
        <td>Bangladesh</td>
        <td>148460</td>
        <td>130170</td>
        <td>18290</td>
        <td>168957745</td>
        <td>1.6</td>
        <td>21.14</td>
        <td>5.61</td>
        <td>0.46</td>
    </tr>
    <tr>
        <td>65</td>
        <td>gm</td>
        <td>Germany</td>
        <td>357022</td>
        <td>348672</td>
        <td>8350</td>
        <td>80854408</td>
        <td>0.17</td>
        <td>8.47</td>
        <td>11.42</td>
        <td>1.24</td>
    </tr>
    <tr>
        <td>80</td>
        <td>iz</td>
        <td>Iraq</td>
        <td>438317</td>
        <td>437367</td>
        <td>950</td>
        <td>37056169</td>
        <td>2.93</td>
        <td>31.45</td>
        <td>3.77</td>
        <td>1.62</td>
    </tr>
    <tr>
        <td>83</td>
        <td>it</td>
        <td>Italy</td>
        <td>301340</td>
        <td>294140</td>
        <td>7200</td>
        <td>61855120</td>
        <td>0.27</td>
        <td>8.74</td>
        <td>10.19</td>
        <td>4.1</td>
    </tr>
    <tr>
        <td>85</td>
        <td>ja</td>
        <td>Japan</td>
        <td>377915</td>
        <td>364485</td>
        <td>13430</td>
        <td>126919659</td>
        <td>0.16</td>
        <td>7.93</td>
        <td>9.51</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>91</td>
        <td>ks</td>
        <td>Korea, South</td>
        <td>99720</td>
        <td>96920</td>
        <td>2800</td>
        <td>49115196</td>
        <td>0.14</td>
        <td>8.19</td>
        <td>6.75</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>120</td>
        <td>mo</td>
        <td>Morocco</td>
        <td>446550</td>
        <td>446300</td>
        <td>250</td>
        <td>33322699</td>
        <td>1.0</td>
        <td>18.2</td>
        <td>4.81</td>
        <td>3.36</td>
    </tr>
    <tr>
        <td>138</td>
        <td>rp</td>
        <td>Philippines</td>
        <td>300000</td>
        <td>298170</td>
        <td>1830</td>
        <td>100998376</td>
        <td>1.61</td>
        <td>24.27</td>
        <td>6.11</td>
        <td>2.09</td>
    </tr>
    <tr>
        <td>139</td>
        <td>pl</td>
        <td>Poland</td>
        <td>312685</td>
        <td>304255</td>
        <td>8430</td>
        <td>38562189</td>
        <td>0.09</td>
        <td>9.74</td>
        <td>10.19</td>
        <td>0.46</td>
    </tr>
    <tr>
        <td>163</td>
        <td>sp</td>
        <td>Spain</td>
        <td>505370</td>
        <td>498980</td>
        <td>6390</td>
        <td>48146134</td>
        <td>0.89</td>
        <td>9.64</td>
        <td>9.04</td>
        <td>8.31</td>
    </tr>
    <tr>
        <td>173</td>
        <td>th</td>
        <td>Thailand</td>
        <td>513120</td>
        <td>510890</td>
        <td>2230</td>
        <td>67976405</td>
        <td>0.34</td>
        <td>11.19</td>
        <td>7.8</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>182</td>
        <td>ug</td>
        <td>Uganda</td>
        <td>241038</td>
        <td>197100</td>
        <td>43938</td>
        <td>37101745</td>
        <td>3.24</td>
        <td>43.79</td>
        <td>10.69</td>
        <td>0.74</td>
    </tr>
    <tr>
        <td>185</td>
        <td>uk</td>
        <td>United Kingdom</td>
        <td>243610</td>
        <td>241930</td>
        <td>1680</td>
        <td>64088222</td>
        <td>0.54</td>
        <td>12.17</td>
        <td>9.35</td>
        <td>2.54</td>
    </tr>
    <tr>
        <td>192</td>
        <td>vm</td>
        <td>Vietnam</td>
        <td>331210</td>
        <td>310070</td>
        <td>21140</td>
        <td>94348835</td>
        <td>0.97</td>
        <td>15.96</td>
        <td>5.93</td>
        <td>0.3</td>
    </tr>
</table>



What country has the most people? What country has the highest growth rate?
Which countries have the highest ratios of water to land? Which countries have more water than land?
Which countries will add the most people to their population next year?
Which countries have a higher death rate than birth rate?
What countries have the highest population/area ratio and how does it compare to list we found in the previous screen?


```python
%%sql
--country with most people
SELECT name, population
FROM facts
WHERE population >= (SELECT MAX(population)
                    FROM facts 
                    WHERE name <> 'World')
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
    </tr>
    <tr>
        <td>China</td>
        <td>1367485388</td>
    </tr>
    <tr>
        <td>World</td>
        <td>7256490011</td>
    </tr>
</table>




```python
%%sql
--country with highest growth 
SELECT name, population, population_growth
FROM facts
WHERE population_growth >= (SELECT MAX(population_growth)
                    FROM facts 
                    WHERE name <> 'World')
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
        <th>population_growth</th>
    </tr>
    <tr>
        <td>South Sudan</td>
        <td>12042910</td>
        <td>4.02</td>
    </tr>
</table>




```python
%%sql
--country with highest ratios of water to land 
SELECT name, area_water, area_land, area_water/area_land as w_l
FROM facts
WHERE w_l >= (SELECT MAX(area_water/area_land)
                    FROM facts 
                    WHERE name <> 'World')
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>area_water</th>
        <th>area_land</th>
        <th>w_l</th>
    </tr>
    <tr>
        <td>British Indian Ocean Territory</td>
        <td>54340</td>
        <td>60</td>
        <td>905</td>
    </tr>
</table>




```python
%%sql
--country with more water than land 
SELECT name, area_water, area_land, area_water - area_land as water_mass
FROM facts
WHERE water_mass >= (SELECT MAX(area_water - area_land)
                    FROM facts 
                    WHERE name <> 'World')
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>area_water</th>
        <th>area_land</th>
        <th>water_mass</th>
    </tr>
    <tr>
        <td>British Indian Ocean Territory</td>
        <td>54340</td>
        <td>60</td>
        <td>54280</td>
    </tr>
</table>




```python
%%sql
--country will add the most people to their population next year 
SELECT name, birth_rate 
FROM facts
WHERE birth_rate >= (SELECT MAX(birth_rate )
                    FROM facts 
                    WHERE name <> 'World')
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>birth_rate</th>
    </tr>
    <tr>
        <td>Niger</td>
        <td>45.45</td>
    </tr>
</table>




```python
%%sql
--country have a higher death rate than birth rate 
SELECT name, birth_rate, death_rate
FROM facts
WHERE death_rate > birth_rate AND  name <> 'World'
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>birth_rate</th>
        <th>death_rate</th>
    </tr>
    <tr>
        <td>Austria</td>
        <td>9.41</td>
        <td>9.42</td>
    </tr>
    <tr>
        <td>Belarus</td>
        <td>10.7</td>
        <td>13.36</td>
    </tr>
    <tr>
        <td>Bosnia and Herzegovina</td>
        <td>8.87</td>
        <td>9.75</td>
    </tr>
    <tr>
        <td>Bulgaria</td>
        <td>8.92</td>
        <td>14.44</td>
    </tr>
    <tr>
        <td>Croatia</td>
        <td>9.45</td>
        <td>12.18</td>
    </tr>
    <tr>
        <td>Czech Republic</td>
        <td>9.63</td>
        <td>10.34</td>
    </tr>
    <tr>
        <td>Estonia</td>
        <td>10.51</td>
        <td>12.4</td>
    </tr>
    <tr>
        <td>Germany</td>
        <td>8.47</td>
        <td>11.42</td>
    </tr>
    <tr>
        <td>Greece</td>
        <td>8.66</td>
        <td>11.09</td>
    </tr>
    <tr>
        <td>Hungary</td>
        <td>9.16</td>
        <td>12.73</td>
    </tr>
    <tr>
        <td>Italy</td>
        <td>8.74</td>
        <td>10.19</td>
    </tr>
    <tr>
        <td>Japan</td>
        <td>7.93</td>
        <td>9.51</td>
    </tr>
    <tr>
        <td>Latvia</td>
        <td>10.0</td>
        <td>14.31</td>
    </tr>
    <tr>
        <td>Lithuania</td>
        <td>10.1</td>
        <td>14.27</td>
    </tr>
    <tr>
        <td>Moldova</td>
        <td>12.0</td>
        <td>12.59</td>
    </tr>
    <tr>
        <td>Monaco</td>
        <td>6.65</td>
        <td>9.24</td>
    </tr>
    <tr>
        <td>Poland</td>
        <td>9.74</td>
        <td>10.19</td>
    </tr>
    <tr>
        <td>Portugal</td>
        <td>9.27</td>
        <td>11.02</td>
    </tr>
    <tr>
        <td>Romania</td>
        <td>9.14</td>
        <td>11.9</td>
    </tr>
    <tr>
        <td>Russia</td>
        <td>11.6</td>
        <td>13.69</td>
    </tr>
    <tr>
        <td>Serbia</td>
        <td>9.08</td>
        <td>13.66</td>
    </tr>
    <tr>
        <td>Slovenia</td>
        <td>8.42</td>
        <td>11.37</td>
    </tr>
    <tr>
        <td>Ukraine</td>
        <td>10.72</td>
        <td>14.46</td>
    </tr>
    <tr>
        <td>Saint Pierre and Miquelon</td>
        <td>7.42</td>
        <td>9.72</td>
    </tr>
</table>




```python
%%sql
--country have the highest population/area ratio  
SELECT name, population, area, population/area as pop_area_ratio
FROM facts
WHERE pop_area_ratio >= (SELECT MAX(population/area)
                         FROM facts
                         WHERE name <> 'World');
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
        <th>area</th>
        <th>pop_area_ratio</th>
    </tr>
    <tr>
        <td>Macau</td>
        <td>592731</td>
        <td>28</td>
        <td>21168</td>
    </tr>
</table>


