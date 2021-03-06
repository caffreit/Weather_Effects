[Homepage of blog-](https://caffreit.github.io/blog-/)

## Analysis of Weather effects

This is the second post in a series on the Malahide ParkRun. It will look at how the weather affects (if at all) attendence and the like.

The [first post](https://caffreit.github.io/ParkRun_Part_1/) looked at seasonality in attendence and how finish times vary with age etc.


```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import scipy.stats as stats
%matplotlib inline
```

#### Import ParkRun data


```python
path_to_file = 'C:\Users\Administrator\Documents\Python Scripts\examplepark.csv'
data = pd.read_csv(path_to_file)
```

#### Some data cleaning


```python
data['Time'] = ((pd.to_numeric(data['Time'].str.slice(0,2)))*60)+(pd.to_numeric(data['Time'].str.slice(3,5)))+((pd.to_numeric(data['Time'].str.slice(6,8)))/60)
data['Date'] = pd.to_datetime(data['Date'],errors='coerce', format='%d-%m-%Y')
data['Age_Cat'] = pd.to_numeric(data['Age_Cat'].str.slice(2,4),errors='coerce', downcast='signed')
data['Age_Grade'] = pd.to_numeric(data['Age_Grade'].str.slice(0,5),errors='coerce')
data['Club_Coded'] = data['Club'].isnull()
```

#### Adding a column which codes whether or not the runner was a member of a club. 
1 for member, 0 for non-member.


```python
def converter(Club):
    if Club==True:
        return 0
    else:
        return 1
```


```python
data['Club_Coded'] = data['Club_Coded'].apply(converter)
data.head(10)
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Pos</th>
      <th>Name</th>
      <th>Time</th>
      <th>Age_Cat</th>
      <th>Age_Grade</th>
      <th>Gender</th>
      <th>Gen_Pos</th>
      <th>Club</th>
      <th>Note</th>
      <th>Total_Runs</th>
      <th>Run_No.</th>
      <th>Club_Coded</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2012-11-10</td>
      <td>1</td>
      <td>Michael MCSWIGGAN</td>
      <td>18.316667</td>
      <td>35.0</td>
      <td>73.43</td>
      <td>M</td>
      <td>1.0</td>
      <td>Portmarnock Athletic Club</td>
      <td>First Timer!</td>
      <td>29.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2012-11-10</td>
      <td>2</td>
      <td>Alan FOLEY</td>
      <td>18.433333</td>
      <td>30.0</td>
      <td>71.16</td>
      <td>M</td>
      <td>2.0</td>
      <td>Raheny Shamrock AC</td>
      <td>First Timer!</td>
      <td>99.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2012-11-10</td>
      <td>3</td>
      <td>Matt SHIELDS</td>
      <td>18.533333</td>
      <td>55.0</td>
      <td>85.07</td>
      <td>M</td>
      <td>3.0</td>
      <td>North Belfast Harriers</td>
      <td>First Timer!</td>
      <td>274.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2012-11-10</td>
      <td>4</td>
      <td>David GARGAN</td>
      <td>18.650000</td>
      <td>40.0</td>
      <td>73.73</td>
      <td>M</td>
      <td>4.0</td>
      <td>Raheny Shamrock AC</td>
      <td>First Timer!</td>
      <td>107.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2012-11-10</td>
      <td>5</td>
      <td>Paul SINTON-HEWITT</td>
      <td>18.900000</td>
      <td>50.0</td>
      <td>79.28</td>
      <td>M</td>
      <td>5.0</td>
      <td>Ranelagh Harriers</td>
      <td>First Timer!</td>
      <td>369.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2012-11-10</td>
      <td>6</td>
      <td>John Gerard MURPHY</td>
      <td>20.250000</td>
      <td>40.0</td>
      <td>68.97</td>
      <td>M</td>
      <td>6.0</td>
      <td>North Belfast Harriers</td>
      <td>First Timer!</td>
      <td>342.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2012-11-10</td>
      <td>7</td>
      <td>Conor FITZPATRICK</td>
      <td>20.283333</td>
      <td>20.0</td>
      <td>64.26</td>
      <td>M</td>
      <td>7.0</td>
      <td>Portmarnock Athletic Club</td>
      <td>First Timer!</td>
      <td>40.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>2012-11-10</td>
      <td>8</td>
      <td>Rachael BECK</td>
      <td>20.450000</td>
      <td>40.0</td>
      <td>76.37</td>
      <td>F</td>
      <td>1.0</td>
      <td>Fingal Triathlon Club</td>
      <td>First Timer!</td>
      <td>9.0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>2012-11-10</td>
      <td>9</td>
      <td>Des HUSIN</td>
      <td>20.533333</td>
      <td>45.0</td>
      <td>69.07</td>
      <td>M</td>
      <td>8.0</td>
      <td>NaN</td>
      <td>First Timer!</td>
      <td>296.0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2012-11-10</td>
      <td>10</td>
      <td>John COLEMAN</td>
      <td>20.816667</td>
      <td>30.0</td>
      <td>63.01</td>
      <td>M</td>
      <td>9.0</td>
      <td>NaN</td>
      <td>First Timer!</td>
      <td>87.0</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



#### Import weather data. 
I found the data at http://www.met.ie/climate-request/
The data was measured at Dublin Airport, many variables are recorded but I kept (what I think are) the important ones. The max and min temps as well as the amount of rain measured in millimetres.


```python
path_to_file = 'C:\Users\Administrator\Documents\Python Scripts\dly532.csv'
weatherdata = pd.read_csv(path_to_file)
weatherdata.head()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>maxtp</th>
      <th>mintp</th>
      <th>rain</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10-Nov-12</td>
      <td>8.2</td>
      <td>1.9</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>17-Nov-12</td>
      <td>7.9</td>
      <td>2.8</td>
      <td>0.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>24-Nov-12</td>
      <td>6.0</td>
      <td>-2.4</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01-Dec-12</td>
      <td>5.5</td>
      <td>-0.2</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>08-Dec-12</td>
      <td>7.3</td>
      <td>-0.2</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



#### Creating a dataframe containing several statistics for each date.


```python
dd = {'Runner_Count': data.groupby('Date').size(), \
     'Min_Time': data.groupby('Date').min()['Time'], \
    'Mean_Time': data.groupby('Date').mean()['Time'], \
    'Max_Time': data.groupby('Date').max()['Time'], \
    'STD_Time': data.groupby('Date').std()['Time']}
dfdate = pd.DataFrame(data=dd)
dfdate['Date'] = dfdate.index
dfdate.index = range(275)
```

#### Addition of New Years Resolutions
We saw in the previous post that there is a peak in the attendence just after New Years Day. So for later analysis I want to be able to distinguish between dates which are soon after (within two months) of NYDay.


```python
dfdate.loc[(dfdate['Date'].dt.month==1), 'NY_Res?'] = 1
dfdate.loc[(dfdate['Date'].dt.month==2), 'NY_Res?'] = 1
dfdate.loc[(dfdate['Date'].dt.month==3), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==4), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==5), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==6), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==7), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==8), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==9), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==10), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==11), 'NY_Res?'] = 0
dfdate.loc[(dfdate['Date'].dt.month==12), 'NY_Res?'] = 0
dfdate.head(10)
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Max_Time</th>
      <th>Mean_Time</th>
      <th>Min_Time</th>
      <th>Runner_Count</th>
      <th>STD_Time</th>
      <th>Date</th>
      <th>NY_Res?</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>44.783333</td>
      <td>27.830778</td>
      <td>18.316667</td>
      <td>159</td>
      <td>5.495270</td>
      <td>2012-11-10</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>48.000000</td>
      <td>26.322277</td>
      <td>16.050000</td>
      <td>216</td>
      <td>5.815160</td>
      <td>2012-11-17</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>51.150000</td>
      <td>26.752964</td>
      <td>16.400000</td>
      <td>268</td>
      <td>5.177254</td>
      <td>2012-11-24</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>44.516667</td>
      <td>25.932583</td>
      <td>16.716667</td>
      <td>236</td>
      <td>5.258924</td>
      <td>2012-12-01</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>52.150000</td>
      <td>25.786577</td>
      <td>17.233333</td>
      <td>162</td>
      <td>6.002006</td>
      <td>2012-12-08</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>46.633333</td>
      <td>24.957809</td>
      <td>17.050000</td>
      <td>155</td>
      <td>5.008983</td>
      <td>2012-12-15</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>36.266667</td>
      <td>25.213864</td>
      <td>17.466667</td>
      <td>125</td>
      <td>4.615561</td>
      <td>2012-12-22</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>36.916667</td>
      <td>25.257328</td>
      <td>17.716667</td>
      <td>124</td>
      <td>4.371382</td>
      <td>2012-12-29</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>39.366667</td>
      <td>26.410031</td>
      <td>16.250000</td>
      <td>227</td>
      <td>4.850019</td>
      <td>2013-01-05</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>41.716667</td>
      <td>26.297910</td>
      <td>16.016667</td>
      <td>377</td>
      <td>4.929012</td>
      <td>2013-01-12</td>
      <td>1.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df1 = dfdate.drop(['Date','Runner_Count'],axis=1)
df1 = pd.melt(df1, "NY_Res?", var_name="measurement")
df1.tail()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>NY_Res?</th>
      <th>measurement</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1095</th>
      <td>1.0</td>
      <td>STD_Time</td>
      <td>5.982540</td>
    </tr>
    <tr>
      <th>1096</th>
      <td>1.0</td>
      <td>STD_Time</td>
      <td>5.872022</td>
    </tr>
    <tr>
      <th>1097</th>
      <td>1.0</td>
      <td>STD_Time</td>
      <td>6.851054</td>
    </tr>
    <tr>
      <th>1098</th>
      <td>1.0</td>
      <td>STD_Time</td>
      <td>5.377526</td>
    </tr>
    <tr>
      <th>1099</th>
      <td>1.0</td>
      <td>STD_Time</td>
      <td>6.163651</td>
    </tr>
  </tbody>
</table>
</div>



### Swarmplot


```python
a4_dims = (11.7, 8.27)
fig, ax = plt.subplots(figsize=a4_dims)

sns.swarmplot(ax=ax, data=df1, x="measurement", y="value", hue='NY_Res?')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11444c88>




![png](/img/output_16_1.png)


Above is a swarmplot where each point corresponds to a date. The colour of the point referes to whether or not that date was within two months of New Years Day. (1 = yes, 0 = no). You might make the assumption that everyone who attends as part of a resolution would be slow, but the mean times of January and February do not seem to be any slower than the rest of the year.

It also tells us that compared to the mean and minimum values of Finish times, the maximum time is quite varied. This makes sense as the average time a person can run 5 km is ~30 mins. The mean is not likely to shift much from this value unless a troupe of aul nuns or ethiopians decide to compete. Whereas it only takes one person send the maximum time sky high.

#### Ratio of Male to Female athletes


```python
df2 = data.groupby(['Date','Gender']).count()['Pos']
df2 = df2.unstack()
df2['Gen_Ratio'] = df2['M']/df2['F']
df2['Date'] = df2.index
df2.index = range(275)
df2.head()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Gender</th>
      <th>F</th>
      <th>M</th>
      <th>Gen_Ratio</th>
      <th>Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>81</td>
      <td>69</td>
      <td>0.851852</td>
      <td>2012-11-10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>85</td>
      <td>117</td>
      <td>1.376471</td>
      <td>2012-11-17</td>
    </tr>
    <tr>
      <th>2</th>
      <td>122</td>
      <td>131</td>
      <td>1.073770</td>
      <td>2012-11-24</td>
    </tr>
    <tr>
      <th>3</th>
      <td>90</td>
      <td>132</td>
      <td>1.466667</td>
      <td>2012-12-01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>57</td>
      <td>92</td>
      <td>1.614035</td>
      <td>2012-12-08</td>
    </tr>
  </tbody>
</table>
</div>



#### Ratio of club members to non-members


```python
df3 = data.groupby(['Date','Club_Coded']).count()['Pos']
df3 = df3.unstack()
df3['Club_Ratio'] = df3[0]/df3[1]
df3['Date'] = df3.index
df3.index = range(275)
df3.head()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Club_Coded</th>
      <th>0</th>
      <th>1</th>
      <th>Club_Ratio</th>
      <th>Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>111</td>
      <td>48</td>
      <td>2.312500</td>
      <td>2012-11-10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>164</td>
      <td>52</td>
      <td>3.153846</td>
      <td>2012-11-17</td>
    </tr>
    <tr>
      <th>2</th>
      <td>194</td>
      <td>74</td>
      <td>2.621622</td>
      <td>2012-11-24</td>
    </tr>
    <tr>
      <th>3</th>
      <td>174</td>
      <td>62</td>
      <td>2.806452</td>
      <td>2012-12-01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>115</td>
      <td>47</td>
      <td>2.446809</td>
      <td>2012-12-08</td>
    </tr>
  </tbody>
</table>
</div>



#### Addition of weather data and the Ratio data


```python
dfdate['MaxTemp'] = weatherdata['maxtp']
dfdate['MinTemp'] = weatherdata['mintp']
dfdate['Rain(mm)'] = weatherdata['rain']
dfdate['Gen_Ratio'] = df2['Gen_Ratio']
dfdate['M_count'] = df2['M']
dfdate['F_count'] = df2['F']
dfdate['Club_Ratio'] = df3['Club_Ratio']
dfdate['No_Club_Count'] = df3[0]
dfdate['In_Club_Count'] = df3[1]
dfdate.head()
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Max_Time</th>
      <th>Mean_Time</th>
      <th>Min_Time</th>
      <th>Runner_Count</th>
      <th>STD_Time</th>
      <th>Date</th>
      <th>NY_Res?</th>
      <th>MaxTemp</th>
      <th>MinTemp</th>
      <th>Rain(mm)</th>
      <th>Gen_Ratio</th>
      <th>M_count</th>
      <th>F_count</th>
      <th>Club_Ratio</th>
      <th>No_Club_Count</th>
      <th>In_Club_Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>44.783333</td>
      <td>27.830778</td>
      <td>18.316667</td>
      <td>159</td>
      <td>5.495270</td>
      <td>2012-11-10</td>
      <td>0.0</td>
      <td>8.2</td>
      <td>1.9</td>
      <td>0.0</td>
      <td>0.851852</td>
      <td>69</td>
      <td>81</td>
      <td>2.312500</td>
      <td>111</td>
      <td>48</td>
    </tr>
    <tr>
      <th>1</th>
      <td>48.000000</td>
      <td>26.322277</td>
      <td>16.050000</td>
      <td>216</td>
      <td>5.815160</td>
      <td>2012-11-17</td>
      <td>0.0</td>
      <td>7.9</td>
      <td>2.8</td>
      <td>0.1</td>
      <td>1.376471</td>
      <td>117</td>
      <td>85</td>
      <td>3.153846</td>
      <td>164</td>
      <td>52</td>
    </tr>
    <tr>
      <th>2</th>
      <td>51.150000</td>
      <td>26.752964</td>
      <td>16.400000</td>
      <td>268</td>
      <td>5.177254</td>
      <td>2012-11-24</td>
      <td>0.0</td>
      <td>6.0</td>
      <td>-2.4</td>
      <td>8.0</td>
      <td>1.073770</td>
      <td>131</td>
      <td>122</td>
      <td>2.621622</td>
      <td>194</td>
      <td>74</td>
    </tr>
    <tr>
      <th>3</th>
      <td>44.516667</td>
      <td>25.932583</td>
      <td>16.716667</td>
      <td>236</td>
      <td>5.258924</td>
      <td>2012-12-01</td>
      <td>0.0</td>
      <td>5.5</td>
      <td>-0.2</td>
      <td>0.0</td>
      <td>1.466667</td>
      <td>132</td>
      <td>90</td>
      <td>2.806452</td>
      <td>174</td>
      <td>62</td>
    </tr>
    <tr>
      <th>4</th>
      <td>52.150000</td>
      <td>25.786577</td>
      <td>17.233333</td>
      <td>162</td>
      <td>6.002006</td>
      <td>2012-12-08</td>
      <td>0.0</td>
      <td>7.3</td>
      <td>-0.2</td>
      <td>0.0</td>
      <td>1.614035</td>
      <td>92</td>
      <td>57</td>
      <td>2.446809</td>
      <td>115</td>
      <td>47</td>
    </tr>
  </tbody>
</table>
</div>



### Finally we get to looking at weather effects
I've plotted several of the stats (Runner Count, Max and Min Times) versus the amount of rain and the minimum temperature.

I've also highlighted the dates which were within two months of New years day. (1 __(blue)__ = yes, 0 __(red)__ = no) This was to see if adverse weather would have a larger effect on the people who were there as a resolution. 

The reason I plotted the max and min times is because they are indicative of the type of person running, i.e. if max time drops then it would indicate that the fair weather athletes stayed at home.

To summarise the plots below, the weather has a minimal effect on the ParkRun data. The rain has only a slight effect on attendence, and min and max times. Minimum temperature has no effect on any of the stats I have here.

#### Runner Count versus Rain and Min Temperature


```python
fig, ax = plt.subplots()

colors = {0.0:'red', 1.0:'blue'}

grouped = dfdate.groupby('NY_Res?')
for key, group in grouped:
    group.plot(ax=ax, kind='scatter', x='Rain(mm)', y='Runner_Count', label=key, color=colors[key], figsize=(12,4))
ax.set_xlim(xmax=30)

fig, ax = plt.subplots()


for key, group in grouped:
    group.plot(ax=ax, kind='scatter', x='MinTemp', y='Runner_Count', label=key, color=colors[key], figsize=(12,4))
ax.set_xlim(xmax=20)

#plt.tight_layout()
plt.show()
```


![png](/img/output_26_0.png)



![png](/img/output_26_1.png)


There seems to a weak relationship between attendence and the amount of rain. The minimm doesn't change much but it seems to reduce the maximum amount. So the rain may not scare a hardcore set of people, but the more rain the less likely someone is to attend.

The minimum temperature seems to have no effect at all.

In blue are runs that took place in either January or February. I'm counting these two months as part of the New Years Resolution effect. I wanted to see if there was any way to distinguish the days with resolution athletes, if perhaps they were easily put off by bad weather. But it doensn't seem to be the case, once people have made their resolution the bad weather doesn't put them off.

#### Max and Min finish times versus Rain in (mm)


```python
fig, ax = plt.subplots()

colors = {0.0:'red', 1.0:'blue'}

grouped = dfdate.groupby('NY_Res?')
for key, group in grouped:
    group.plot(ax=ax, kind='scatter', x='Rain(mm)', y='Max_Time', label=key, color=colors[key], figsize=(12,4))
ax.set_xlim(xmax=30)

fig, ax = plt.subplots()

for key, group in grouped:
    group.plot(ax=ax, kind='scatter', x='Rain(mm)', y='Mean_Time', label=key, color=colors[key], figsize=(12,4))
ax.set_xlim(xmax=30)

fig, ax = plt.subplots()

for key, group in grouped:
    group.plot(ax=ax, kind='scatter', x='Rain(mm)', y='Min_Time', label=key, color=colors[key], figsize=(12,4))
ax.set_xlim(xmax=30)

plt.show()
```


![png](/img/output_29_0.png)



![png](/img/output_29_1.png)



![png](/img/output_29_2.png)


As we showed in the previous post, The min and and max times show a dependence on the Runner Count. So we expect to see a slight change in the min and max times with amount of Rain. The max and mins show a slight decrease and increase respectively. 

The mean time sees no change. If the mean were to drop with rain, we could probaly say that the enthusiasts still attend while the rest of us curl up with a lovely cupÃ¡n tae.

So from these plots, we can draw the not-so unexpected conclusion that no-one, neither enthusiasts or couch potatoes like running in the rain and both are equally less likely to run in the rain.

_A note on the plots versus Rain, I also plotted them on a log scale. Unfortunately this removes all points with no rain and the trend is not much clearer. So for ease of interpretaion I have left them all on a linear scale._

#### Max and Min finish times versus minmum temperature

```python
fig, ax = plt.subplots()

colors = {0.0:'red', 1.0:'blue'}

grouped = dfdate.groupby('NY_Res?')
for key, group in grouped:
    group.plot(ax=ax, kind='scatter', x='MinTemp', y='Max_Time', label=key, color=colors[key], figsize=(12,4))
ax.set_xlim(xmax=20)

fig, ax = plt.subplots()


for key, group in grouped:
    group.plot(ax=ax, kind='scatter', x='MinTemp', y='Min_Time', label=key, color=colors[key], figsize=(12,4))
ax.set_xlim(xmax=20)

plt.show()
```


![png](/img/output_31_0.png)



![png](/img/output_31_1.png)


Again, temperature has no effect on either attendence or the max or min times. __Conclusion:__ people don't care how cold it is.

### Summary:
Nobody likes running in the rain, not even enthusiasts.

Nobody cares how cold it is.

People who make a resolution to go running still go running despite bad weather. They also don't appear to be all slow mammys.

### Addendum

While looking at the pairplot of the dataframe (see bottom), I saw that the Ratio of Males to Females has strong dependence on attendence. Given this strong correlation I wanted to look into the differences between male and female athletes in more detail. As such the next two posts are about the differences between the two and whether or not we can predict the gender of the athlete.


```python
ax = dfdate.plot.scatter(x='Runner_Count',y='Gen_Ratio', figsize=(8,6))

ax.set_xlabel("Runner Count", fontsize=20)
ax.set_ylabel("Gender Ratio", fontsize=20)
ax.grid('on', which='major', axis='x')
ax.grid('on', which='major', axis='y')
ax.grid('on', which='minor', axis='x')
ax.grid('on', which='minor', axis='y')
ax.set_xscale('log')
#ax.set_yscale('log')
#ax.set_xlim(xmax=30)
```


![png](/img/output_36_0.png)



```python
sns.pairplot(dfdate)
```




    <seaborn.axisgrid.PairGrid at 0x11b12630>




![png](/img/output_37_1.png)



```python

```
