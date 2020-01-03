---
title: "Placement Network of Economics PhD Programs"
date: 2020-01-03
tags: [social network, networkx, data wrangling]
header:
  image: "/images/perceptron/percept.jpg"
excerpt: "social network, networkx, data wrangling"
mathjax: "true"
---

```python
import pandas as pd
import numpy as np
import csv
import networkx as nx
from operator import itemgetter
import community
import matplotlib.pyplot as plt
import seaborn as sns; sns.set()
plt.style.use('seaborn-bright')
```

### Preparing Data 

#### US Programs Metadata


```python
meta = pd.read_csv('programs_meta.csv')
meta['Name'] = meta['Name'].str.lower()
#Rank: average of R-Rank: 
#Programs are ranked highly if they have similar features to programs viewed by faculty as top-notch.
#meta['Rank'] = meta['Rank']).round(0)
meta['Rank'] = len(meta)+1-meta['Rank'].rank(method='max')
meta.sort_values(by=['Rank'], ascending=False).head(10)
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
      <th>ID</th>
      <th>Name</th>
      <th>Rank</th>
      <th>Private</th>
      <th>Size</th>
      <th>City</th>
      <th>Space</th>
      <th>FS</th>
      <th>International</th>
      <th>Citations</th>
      <th>FacultySize</th>
      <th>Time</th>
      <th>Aca%</th>
      <th>NonAca%</th>
      <th>GradNum</th>
      <th>TolIndex</th>
      <th>AvgIndex</th>
      <th>Tier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>21</th>
      <td>24</td>
      <td>harvard university</td>
      <td>104.0</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>1.00</td>
      <td>0.51</td>
      <td>4.21</td>
      <td>66</td>
      <td>5.00</td>
      <td>0.67</td>
      <td>0.33</td>
      <td>219</td>
      <td>46078.2</td>
      <td>210.40</td>
      <td>1</td>
    </tr>
    <tr>
      <th>59</th>
      <td>65</td>
      <td>university of chicago</td>
      <td>103.0</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>0.55</td>
      <td>0.73</td>
      <td>3.34</td>
      <td>70</td>
      <td>5.50</td>
      <td>0.60</td>
      <td>0.40</td>
      <td>121</td>
      <td>13319.6</td>
      <td>110.08</td>
      <td>1</td>
    </tr>
    <tr>
      <th>28</th>
      <td>31</td>
      <td>massachusetts institute of technology</td>
      <td>102.0</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>0.96</td>
      <td>0.58</td>
      <td>3.25</td>
      <td>46</td>
      <td>4.80</td>
      <td>0.74</td>
      <td>0.26</td>
      <td>136</td>
      <td>32221.0</td>
      <td>236.92</td>
      <td>1</td>
    </tr>
    <tr>
      <th>51</th>
      <td>57</td>
      <td>university of california-berkeley</td>
      <td>101.0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>0.50</td>
      <td>0.34</td>
      <td>2.83</td>
      <td>60</td>
      <td>5.70</td>
      <td>0.70</td>
      <td>0.30</td>
      <td>149</td>
      <td>19732.6</td>
      <td>132.43</td>
      <td>1</td>
    </tr>
    <tr>
      <th>37</th>
      <td>41</td>
      <td>princeton university</td>
      <td>100.0</td>
      <td>1</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>1.00</td>
      <td>0.71</td>
      <td>3.04</td>
      <td>47</td>
      <td>5.20</td>
      <td>0.62</td>
      <td>0.38</td>
      <td>156</td>
      <td>16353.8</td>
      <td>104.83</td>
      <td>1</td>
    </tr>
    <tr>
      <th>30</th>
      <td>33</td>
      <td>new york university</td>
      <td>98.0</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>1.00</td>
      <td>0.86</td>
      <td>2.24</td>
      <td>59</td>
      <td>5.67</td>
      <td>0.73</td>
      <td>0.27</td>
      <td>114</td>
      <td>11618.0</td>
      <td>101.91</td>
      <td>1</td>
    </tr>
    <tr>
      <th>43</th>
      <td>48</td>
      <td>stanford university</td>
      <td>98.0</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>0.83</td>
      <td>0.61</td>
      <td>2.81</td>
      <td>73</td>
      <td>6.00</td>
      <td>0.63</td>
      <td>0.37</td>
      <td>147</td>
      <td>23036.0</td>
      <td>156.71</td>
      <td>1</td>
    </tr>
    <tr>
      <th>103</th>
      <td>110</td>
      <td>yale university</td>
      <td>97.0</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
      <td>1.00</td>
      <td>0.67</td>
      <td>1.68</td>
      <td>43</td>
      <td>5.20</td>
      <td>0.78</td>
      <td>0.22</td>
      <td>139</td>
      <td>19201.8</td>
      <td>138.14</td>
      <td>1</td>
    </tr>
    <tr>
      <th>72</th>
      <td>78</td>
      <td>university of maryland college park</td>
      <td>96.0</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>1</td>
      <td>0.95</td>
      <td>0.71</td>
      <td>1.95</td>
      <td>52</td>
      <td>5.30</td>
      <td>0.39</td>
      <td>0.61</td>
      <td>122</td>
      <td>3291.2</td>
      <td>26.98</td>
      <td>2</td>
    </tr>
    <tr>
      <th>85</th>
      <td>91</td>
      <td>university of pennsylvania</td>
      <td>94.0</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
      <td>1</td>
      <td>0.23</td>
      <td>0.77</td>
      <td>2.21</td>
      <td>46</td>
      <td>4.90</td>
      <td>0.66</td>
      <td>0.34</td>
      <td>128</td>
      <td>8138.2</td>
      <td>63.58</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
meta.describe()
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
      <th>ID</th>
      <th>Rank</th>
      <th>Private</th>
      <th>Size</th>
      <th>City</th>
      <th>Space</th>
      <th>FS</th>
      <th>International</th>
      <th>Citations</th>
      <th>FacultySize</th>
      <th>Time</th>
      <th>Aca%</th>
      <th>NonAca%</th>
      <th>GradNum</th>
      <th>TolIndex</th>
      <th>AvgIndex</th>
      <th>Tier</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
      <td>104.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>56.423077</td>
      <td>52.259615</td>
      <td>0.326923</td>
      <td>2.605769</td>
      <td>0.576923</td>
      <td>0.375000</td>
      <td>0.700673</td>
      <td>0.638173</td>
      <td>1.198558</td>
      <td>31.625000</td>
      <td>5.515000</td>
      <td>0.652500</td>
      <td>0.347596</td>
      <td>59.394231</td>
      <td>3576.419231</td>
      <td>40.440385</td>
      <td>3.461538</td>
    </tr>
    <tr>
      <th>std</th>
      <td>31.491004</td>
      <td>30.169745</td>
      <td>0.471361</td>
      <td>1.092057</td>
      <td>0.496440</td>
      <td>0.486467</td>
      <td>0.279703</td>
      <td>0.196636</td>
      <td>0.792733</td>
      <td>16.980393</td>
      <td>0.776348</td>
      <td>0.158186</td>
      <td>0.158049</td>
      <td>44.822285</td>
      <td>6847.360509</td>
      <td>42.971767</td>
      <td>1.371861</td>
    </tr>
    <tr>
      <th>min</th>
      <td>2.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.150000</td>
      <td>10.000000</td>
      <td>3.000000</td>
      <td>0.290000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>28.750000</td>
      <td>26.000000</td>
      <td>0.000000</td>
      <td>2.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.495000</td>
      <td>0.527500</td>
      <td>0.657500</td>
      <td>18.000000</td>
      <td>5.000000</td>
      <td>0.570000</td>
      <td>0.247500</td>
      <td>28.000000</td>
      <td>487.800000</td>
      <td>15.482500</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>57.500000</td>
      <td>52.500000</td>
      <td>0.000000</td>
      <td>3.000000</td>
      <td>1.000000</td>
      <td>0.000000</td>
      <td>0.770000</td>
      <td>0.690000</td>
      <td>0.995000</td>
      <td>28.000000</td>
      <td>5.500000</td>
      <td>0.645000</td>
      <td>0.355000</td>
      <td>46.000000</td>
      <td>1060.700000</td>
      <td>27.295000</td>
      <td>4.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>83.250000</td>
      <td>78.250000</td>
      <td>1.000000</td>
      <td>4.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.770000</td>
      <td>1.470000</td>
      <td>39.000000</td>
      <td>6.000000</td>
      <td>0.752500</td>
      <td>0.430000</td>
      <td>80.250000</td>
      <td>2892.700000</td>
      <td>46.370000</td>
      <td>5.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>110.000000</td>
      <td>104.000000</td>
      <td>1.000000</td>
      <td>4.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>1.000000</td>
      <td>0.980000</td>
      <td>4.210000</td>
      <td>88.000000</td>
      <td>8.000000</td>
      <td>1.000000</td>
      <td>0.710000</td>
      <td>219.000000</td>
      <td>46078.200000</td>
      <td>236.920000</td>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### US Top 100 Placement Data 


```python
df = pd.read_csv('usdata.csv')
#df['Weight'] = df['Target'].map(meta.set_index('Name')['Rank'])
df.rename(columns={'Scores':'Weight'}, inplace=True)
df['Year'] = pd.to_datetime(df['Year'], format = '%Y').dt.year
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
      <th>Unnamed: 0</th>
      <th>ID</th>
      <th>Source</th>
      <th>Year</th>
      <th>Target</th>
      <th>Weight</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>11</td>
      <td>2</td>
      <td>arizona state university</td>
      <td>2011.0</td>
      <td>university of california-santa barbara</td>
      <td>88.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14</td>
      <td>2</td>
      <td>arizona state university</td>
      <td>2014.0</td>
      <td>washington university in st. louis</td>
      <td>110.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20</td>
      <td>4</td>
      <td>boston college</td>
      <td>2011.0</td>
      <td>clark university</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>24</td>
      <td>4</td>
      <td>boston college</td>
      <td>2017.0</td>
      <td>wayne state university</td>
      <td>10.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>27</td>
      <td>4</td>
      <td>boston college</td>
      <td>2012.0</td>
      <td>washington state university</td>
      <td>17.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df.shape
```




    (832, 6)



#### Global Placement Data


```python
glo = pd.read_csv('GlobalPlacement.csv').dropna()
glo.rename(columns={'Scores':'Weight', 'Placement':'Target'}, inplace=True)
glo['Year'] = pd.to_datetime(glo['Year'], format = '%Y').dt.year
glo.head()
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
      <th>Unnamed: 0</th>
      <th>ID</th>
      <th>Source</th>
      <th>Year</th>
      <th>Target</th>
      <th>Weight</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>2</td>
      <td>arizona state university</td>
      <td>2011</td>
      <td>hofstra university</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>2</td>
      <td>arizona state university</td>
      <td>2015</td>
      <td>university of winnipeg</td>
      <td>4.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>2</td>
      <td>arizona state university</td>
      <td>2014</td>
      <td>nanjing university school of business</td>
      <td>10.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>2</td>
      <td>arizona state university</td>
      <td>2011</td>
      <td>yeshiva university</td>
      <td>11.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>2</td>
      <td>arizona state university</td>
      <td>2016</td>
      <td>virginia commonwealth university</td>
      <td>14.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
glo.shape
```




    (3631, 6)




```python
## Subdata of different years
def subdata(df):
    df10 = df[df['Year']==2010]
    df11 = df[df['Year']==2011]
    df12 = df[df['Year']==2012]
    df13 = df[df['Year']==2013]
    df14 = df[df['Year']==2014]
    df15 = df[df['Year']==2015]
    df16 = df[df['Year']==2016]
    df17 = df[df['Year']==2017]

    subdf = [df10, df11, df12, df13, df14, df15, df16, df17]
    
    return subdf

subdf = subdata(df)
subglo = subdata(glo)
```

### Macro Trend of Econ Job Market

#### US Market


```python
yrlabs = ['2010', '2011', '2012', '2013', '2014', '2015', '2016', '2017']
```


```python
yearagg = df.groupby('Year').agg({'Year':'count', 'Weight':'mean'})
```

#### Global Market


```python
yearaggglo = glo.groupby('Year').agg({'Year':'count', 'Weight':'mean'})[:8]
```

#### Plots


```python
plt.plot(yrlabs, "Weight", data=yearagg, label='US')
plt.plot(yrlabs, "Weight", data=yearaggglo, label='Global')

plt.suptitle('Harder to Get a Good Econ Acedemic Position')
plt.xlabel('Year')
plt.ylabel('Average Quality of Placement')
plt.legend()
plt.savefig('agg_year_quality.png')
```


![png](Network_Placement_files/Network_Placement_19_0.png)



```python
plt.plot(yrlabs, "Year", data=yearagg, label='US')
plt.plot(yrlabs, "Year", data=yearaggglo, label='Global')
#plt.suptitle('Number of New Econ Graduates Who Got Top 100 US Academic Placements')
plt.suptitle('Only around 25% Are Placed in Top 100 U.S Schools')
plt.xlabel('Year')
plt.ylabel('Number of Placements')
plt.legend()
plt.savefig('agg_year_number.png')
```


![png](Network_Placement_files/Network_Placement_20_0.png)


### Create a Directed Graph


```python
## weights: aggregate counts of placements across years
aggplacements = glo.groupby(['Source', 'Target']).agg({'ID':'count'})
aggplacements.rename(columns={'ID':'Counts'}, inplace=True)
aggplacements.reset_index(level=['Source', 'Target'], inplace=True)
aggplacements.sort_values(by=['Counts'], ascending=False).head(5)
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
      <th>Source</th>
      <th>Target</th>
      <th>Counts</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>592</th>
      <td>harvard university</td>
      <td>university of chicago</td>
      <td>12</td>
    </tr>
    <tr>
      <th>597</th>
      <td>harvard university</td>
      <td>university of pennsylvania</td>
      <td>10</td>
    </tr>
    <tr>
      <th>579</th>
      <td>harvard university</td>
      <td>stanford university</td>
      <td>7</td>
    </tr>
    <tr>
      <th>833</th>
      <td>massachusetts institute of technology</td>
      <td>university of illinois</td>
      <td>7</td>
    </tr>
    <tr>
      <th>96</th>
      <td>boston university</td>
      <td>shanghai university of finance and economics</td>
      <td>6</td>
    </tr>
  </tbody>
</table>
</div>




```python
def newnetwork(df, verbose=True):
    '''
    This function creates a new directed graph object by adding nodes, edges, and weights.
    '''
   
    #nodes
    node_names = list(set([i for i in df['Target'].unique()] + 
                          [i for i in df['Source'].unique()]))
    
    #edges
    edges = [(df.iloc[i]['Source'], df.iloc[i]['Target']) for i in range(len(df))]
    
    
    #initialize a Graph object
    G = nx.DiGraph(directed=True)

    #add lists of nodes and edges
    G.add_nodes_from(node_names)
    G.add_edges_from(edges)

    #basic information about your newly-created network
    if verbose==True:
        print('Number of Nodes:', len(node_names))
        print('Number of Edges:', len(edges))
        print(nx.info(G))
    
    #add edges weight (aggregate counts of placements)
    aggplacements = df.groupby(['Source', 'Target']).agg({'ID':'count'})
    aggplacements.rename(columns={'ID':'Counts'}, inplace=True)
    aggplacements.reset_index(level=['Source', 'Target'], inplace=True)
    
    for i in G.edges:
        weight = int(aggplacements[(aggplacements['Source']==i[0]) & 
                  (aggplacements['Target']==i[1])].Counts)
        G.edges[i[0], i[1]]['weight'] = weight
        
    #The set_node_attributes function takes three variables: 
    #1. the Graph to which you’re adding the attribute
    #2. the dictionary of id-attribute pairs
    #3. the name of the new attribute.

    for i in meta.columns[2:]:
        nx.set_node_attributes(G, meta.set_index('Name').to_dict()[i], i)
        
    return G
```


```python
G = newnetwork(df)
```

    Number of Nodes: 100
    Number of Edges: 832
    Name: 
    Type: DiGraph
    Number of nodes: 100
    Number of edges: 599
    Average in degree:   5.9900
    Average out degree:   5.9900
    


```python
GG = newnetwork(glo)
```

    Number of Nodes: 2096
    Number of Edges: 3631
    Name: 
    Type: DiGraph
    Number of nodes: 2096
    Number of edges: 3224
    Average in degree:   1.5382
    Average out degree:   1.5382
    


```python
print(G.nodes['university of chicago'])
```

    {'Rank': 103.0, 'Private': 1, 'Size': 4, 'City': 1, 'Space': 0, 'FS': 0.55, 'International': 0.73, 'Citations': 3.34, 'FacultySize': 70, 'Time': 5.5, 'Aca%': 0.6, 'NonAca%': 0.4, 'GradNum': 121, 'TolIndex': 13319.6, 'AvgIndex': 110.08, 'Tier': 1}
    


```python
subG = [newnetwork(i, False) for i in subdf]
subGglo = [newnetwork(i, False) for i in subglo]
```

### Network Statistics

#### Self-loop
Is it common to be hired by where they graduate from?


```python
def selfloop(df):
    sameratios=[]
    for i in subdf:
        sameratios.append(len(i[i['Source']==i['Target']])/len(df))
    plt.plot(yrlabs, sameratios)
    plt.xlabel('Year')
    plt.ylabel('Average Number of Graduates')
```


```python
selfloop(df)
plt.title('US: Ratios of Graduated Hired By Their Alma Maters')
plt.savefig('selfloopus.png')
```


![png](Network_Placement_files/Network_Placement_31_0.png)



```python
selfloop(glo)
plt.title('Global: Ratios of Graduated Hired By Their Alma Maters')
plt.savefig('selfloopglobal.png')
```


![png](Network_Placement_files/Network_Placement_32_0.png)


#### Network density
The ratio of actual edges in the network to all possible edges in the network. (how closely knit your network is)

A 0 would mean that there are no connections at all, and a 1 would indicate that all possible edges are present (a perfectly connected network)


```python
density = nx.density(G)
print("Network density:", density)
```

    Network density: 0.060505050505050506
    


```python
nx.density(G)/nx.density(GG)
```




    82.40847933428579




```python
plt.plot(yrlabs, [nx.density(i)  for i in subG], label='US')
plt.plot(yrlabs, [nx.density(i)  for i in subGglo], label='Global')
plt.suptitle('US Market is 82 times densier than the global job market')
plt.xlabel('Year')
plt.ylabel('Densities')
plt.legend()
plt.savefig('densities.png')
```


![png](Network_Placement_files/Network_Placement_36_0.png)


#### Shortest path
calculates the shortest possible series of nodes and edges that stand between any two nodes


```python
path = nx.shortest_path(G, source="university of chicago", target="indiana university at bloomington")

print("Shortest path:", path)

print("Length of that path:", len(path)-1)
```

    Shortest path: ['university of chicago', 'duke university', 'georgia state university', 'indiana university at bloomington']
    Length of that path: 3
    

#### Transitivity
Transitivity is the ratio of all triangles over all possible triangles. 

Transitivity, like density, expresses how interconnected a graph is in terms of a ratio of actual over possible connections. 

Because the graph is not very dense, there are fewer possible triangles to begin with, which may result in slightly higher transitivity


```python
triadic_closure = nx.transitivity(G)
print("Triadic closure:", triadic_closure)
```

    Triadic closure: 0.16130057191273037
    


```python
nx.transitivity(G)/nx.transitivity(GG)
```




    23.289183681923667




```python
plt.plot(yrlabs, [nx.transitivity(i)  for i in subG], label='US')
plt.plot(yrlabs, [nx.transitivity(i)  for i in subGglo], label='Global')
plt.suptitle('US econ job market is 26x more transitive than the global one')
plt.xlabel('Year')
plt.ylabel('Transitivities')
plt.legend()
plt.savefig('transitivity.png')
```


![png](Network_Placement_files/Network_Placement_42_0.png)


#### Degrees
A node’s degree is the sum of its edges. 


```python
def adddegrees(G):
    degree_dict = dict(G.degree(G.nodes()))
    nx.set_node_attributes(G, degree_dict, 'degree')
    #Outward ties reflect the capacity of the sending department to place its own students
    #Inward ties reflect the capacity of the receiving department to hire and retain faculty. 
    outdegree = {}
    indegree = {}
    for i in df['Source'].unique():
        outdegree[i] = G.out_degree(i)
        indegree[i] = G.in_degree(i)

    nx.set_node_attributes(G, outdegree, 'outdegree')
    nx.set_node_attributes(G, indegree, 'indegree')
    
    #use the built-in function sorted() to sort a dictionary by its keys or values and find the top twenty nodes ranked by degree. 
    #sorted_degree = sorted(degree_dict.items(), key=itemgetter(1), reverse=True)
    #sorted_outdegree = sorted(outdegree.items(), key=itemgetter(1), reverse=True)
    #sorted_indegree = sorted(indegree.items(), key=itemgetter(1), reverse=True)
    #print("Top 10 nodes by degree:")
     #   for d in sorted_degree[:10]:
      #      print(d)

    return G
```


```python
G = adddegrees(G)
GG = adddegrees(GG)
```

##### Degrees Visualization


```python
inout = pd.DataFrame.from_dict(dict(G.nodes(data=True)), orient='index')[['degree','outdegree', 'indegree']]
inoutsorted = inout.sort_values(by=['degree'], ascending=False)[['outdegree', 'indegree']].head(10)
inoutsorted.plot.barh(rot=15, title="US: Schools' Capacities of Hiring Faculty and Placing PhD Students")
plt.savefig('inoutdegrees.png')
```


![png](Network_Placement_files/Network_Placement_47_0.png)



```python
inoutglo = pd.DataFrame.from_dict(dict(GG.nodes(data=True)), orient='index')[['degree','outdegree', 'indegree']]
inoutsortedglo = inoutglo.sort_values(by=['degree'], ascending=False)[['outdegree', 'indegree']].head(10)
inoutsortedglo.plot.barh(rot=15, title="Global: Schools' Capacities of Hiring Faculty and Placing PhD Students")
plt.savefig('inoutdegreesglo.png')
```


![png](Network_Placement_files/Network_Placement_48_0.png)



```python

```

### Centrality 

centrality refers to indicators which identify the most important vertices within a graph.

#### Eigenvector centrality 
is a kind of extension of degree—it looks at a combination of a node’s edges and the edges of that node’s neighbors. Eigenvector centrality cares if you are a hub, but it also cares how many hubs you are connected to. It’s calculated as a value from 0 to 1: the closer to one, the greater the centrality. Eigenvector centrality is useful for understanding which nodes can get information to many other nodes quickly. 
 
Eigenvector centrality is a measure of the influence of a node in a network. It assigns relative scores to all nodes in the network based on the concept that connections to high-scoring nodes contribute more to the score of the node in question than equal connections to low-scoring nodes.



#### Betweenness centrality 
looks at all the shortest paths that pass through a particular node. it’s a quick way of giving you a sense of which nodes are important not because they have lots of connections themselves but because they stand between groups, giving the network connectivity and cohesion.

Betweenness is a centrality measure of a vertex within a graph. Betweenness centrality quantifies the number of times a node acts as a bridge along the shortest path between two other nodes.

#### Closeness centrality 

In connected graphs there is a natural distance metric between all pairs of nodes, defined by the length of their shortest paths. The farness of a node is defined as the sum of its distances to all other nodes, and its closeness is defined as the reciprocal of the farness. Thus, the more central a node is the lower its total distance to all other nodes.


Source: https://sites.google.com/a/umn.edu/social-network-analysis/terminology


```python
def addcentralities(G):
    '''
    The function adds three centralities to the nodes attributes of a network
    '''
    betweenness_dict = nx.betweenness_centrality(G) # Run betweenness centrality
    eigenvector_dict = nx.eigenvector_centrality(G) # Run eigenvector centrality
    closeness_dict = nx.closeness_centrality(G) # Run eigenvector centrality

    # Assign each to an attribute in your network
    nx.set_node_attributes(G, betweenness_dict, 'betweenness')
    nx.set_node_attributes(G, eigenvector_dict, 'eigenvector')
    nx.set_node_attributes(G, closeness_dict, 'closeness')  
    
    sorted_eigenvector = sorted(eigenvector_dict.items(), key=itemgetter(1), reverse=True)
    sorted_betweenness = sorted(betweenness_dict.items(), key=itemgetter(1), reverse=True)
    sorted_closeness = sorted(closeness_dict.items(), key=itemgetter(1), reverse=True)
    
    
    #print("Top 10 nodes by eigenvector centrality:")
    #for b in sorted_eigenvector[:10]:
    #    print(b[0],round(b[1],2))

    return G
```


```python
G = addcentralities(G)
GG = addcentralities(GG)
```

 which of the high betweenness centrality nodes had low degree? That is to say: which high-betweenness nodes are unexpected? 




```python
#
#First get the top 20 nodes by betweenness as a list
#top_eigenvector = sorted_eigenvector[:30]

#Then find and print their degree
#for tb in top_eigenvector: # Loop through top_betweenness
#    degree = degree_dict[tb[0]] 
#    if degree<10:
#        print("Name:", tb[0], "| Eigenvector Centrality:", tb[1], "| Degree:", degree)
```


```python
cent = pd.DataFrame.from_dict(dict(G.nodes(data=True)), orient='index')[['betweenness', 'eigenvector', 'closeness']]
centcomp=cent[(cent.index == 'university of chicago') | 
    (cent.index == 'harvard university') |
    (cent.index == 'massachusetts institute of technology')]
centcomp.round(4)
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
      <th>betweenness</th>
      <th>eigenvector</th>
      <th>closeness</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>university of chicago</th>
      <td>0.0259</td>
      <td>0.1804</td>
      <td>0.2391</td>
    </tr>
    <tr>
      <th>massachusetts institute of technology</th>
      <td>0.2279</td>
      <td>0.1993</td>
      <td>0.2953</td>
    </tr>
    <tr>
      <th>harvard university</th>
      <td>0.0104</td>
      <td>0.0569</td>
      <td>0.2126</td>
    </tr>
  </tbody>
</table>
</div>




```python
centcomp.plot.barh(rot=15, title="US Placement: MIT wins!")
plt.savefig('centralities.png')
```


![png](Network_Placement_files/Network_Placement_60_0.png)



```python
centglo = pd.DataFrame.from_dict(dict(GG.nodes(data=True)), orient='index')[['betweenness', 'eigenvector', 'closeness']]
centcompglo = centglo[(centglo.index == 'university of chicago') | 
    (centglo.index == 'harvard university') |
    (centglo.index == 'massachusetts institute of technology')]
centcompglo.round(4)
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
      <th>betweenness</th>
      <th>eigenvector</th>
      <th>closeness</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>harvard university</th>
      <td>0.0008</td>
      <td>0.0614</td>
      <td>0.0047</td>
    </tr>
    <tr>
      <th>massachusetts institute of technology</th>
      <td>0.0008</td>
      <td>0.0769</td>
      <td>0.0053</td>
    </tr>
    <tr>
      <th>university of chicago</th>
      <td>0.0012</td>
      <td>0.0974</td>
      <td>0.0056</td>
    </tr>
  </tbody>
</table>
</div>




```python
centcompglo.plot.barh(rot=15, title="Global Placement: UChicago wins")
#plt.show()
plt.savefig('centralitiesglobal.png')
```


![png](Network_Placement_files/Network_Placement_62_0.png)


### Spatial Location/Cluster

#### A strongly connected component 
is called attracting if it has no outgoing edges whatsoever. 


```python
#In other words, no placement in other universities
att = sorted(nx.attracting_components(G), key=len, reverse=True)
```

#### Strongly connected components


```python
connected = sorted(nx.strongly_connected_components(G), key=len, reverse=True)
```


```python
len(connected[0])/len(G.nodes)
```




    0.62



38% of the schools are not strongly connected with others

### Saltwater and freshwater economics

They were called the "freshwater school" because Chicago, Pittsburgh, Rochester, Minneapolis, etc. are located closer to the North American Great Lakes.

The established methodological approach to macroeconomic research was primarily defended by economists at the universities and other institutions located near the east and west coast of the United States. 

https://en.wikipedia.org/wiki/Saltwater_and_freshwater_economics

#### Water ratios


```python
#6
freshwater = ['university of chicago', 'carnegie mellon university', 'northwestern university', 'cornell university', 
              'university of minnesota-twin cities', 'university of rochester']
```


```python
#7
saltwater = ['university of california-berkeley', 'brown university', 
             'harvard university', 'university of pennsylvania', 'princeton university', 
             'columbia university in the city of new york', 'yale university']
```


```python
#check list names are the same in the data
for i in saltwater:
    if i not in df['Source'].unique():
        print(i)
        
for i in freshwater:
    if i not in df['Source'].unique():
        print(i)
```


```python
def freshsalt(df):
    ### fresh/salt graduates
    fnum = len(df[df['Source'].isin(freshwater)])
    snum = len(df[df['Source'].isin(saltwater)])
    print('Number of Graduates:')
    print('fresh:',fnum, 'salt:', snum)
    print('salt/fresh', snum/fnum)    
    
    print('fresh-fresh')
    ff = df[(df['Source'].isin(freshwater)) & df['Target'].isin(freshwater)]
    print("Count:", len(ff), "   ", "Percentage:", round(len(ff)/fnum,2))
    
    print('fresh-salt')
    fs = df[(df['Source'].isin(freshwater)) & df['Target'].isin(saltwater)]
    print("Count:", len(fs), "   ", "Percentage:", round(len(fs)/fnum,2))
    
    print('salt-salt')
    ss = df[(df['Source'].isin(saltwater)) & df['Target'].isin(saltwater)]
    print("Count:", len(ss), "   ", "Percentage:", round(len(ss)/snum,2))
    
    print('salt-fresh')
    sf = df[(df['Source'].isin(saltwater)) & df['Target'].isin(freshwater)]
    print("Count:", len(sf), "   ", "Percentage:", round(len(sf)/snum,2))
```


```python
freshsalt(df)
```

    Number of Graduates:
    fresh: 90 salt: 273
    salt/fresh 3.033333333333333
    fresh-fresh
    Count: 10     Percentage: 0.11
    fresh-salt
    Count: 14     Percentage: 0.16
    salt-salt
    Count: 62     Percentage: 0.23
    salt-fresh
    Count: 48     Percentage: 0.18
    


```python
freshsalt(glo)
```

    Number of Graduates:
    fresh: 266 salt: 637
    salt/fresh 2.3947368421052633
    fresh-fresh
    Count: 7     Percentage: 0.03
    fresh-salt
    Count: 9     Percentage: 0.03
    salt-salt
    Count: 49     Percentage: 0.08
    salt-fresh
    Count: 38     Percentage: 0.06
    

#### Water networks


```python
waterlist = freshwater + saltwater
water = df[(df['Source'].isin(waterlist)) & (df['Target'].isin(waterlist))]
len(water)
```




    134




```python
water.Source.unique()
```




    array(['brown university', 'columbia university in the city of new york',
           'cornell university', 'harvard university',
           'northwestern university', 'princeton university',
           'university of california-berkeley', 'university of chicago',
           'university of minnesota-twin cities',
           'university of pennsylvania', 'yale university'], dtype=object)




```python
#two schools don't have placements in other water schools
for i in waterlist:
    if i not in water.Source.unique():
        print(i)
```

    carnegie mellon university
    university of rochester
    


```python
waterG = newnetwork(water)
```

    Number of Nodes: 13
    Number of Edges: 134
    Name: 
    Type: DiGraph
    Number of nodes: 13
    Number of edges: 63
    Average in degree:   4.8462
    Average out degree:   4.8462
    


```python
pos=nx.random_layout(G) # positions for all nodes
plt.figure(figsize = (20,20))

# nodes
nx.draw_networkx_nodes(waterG,pos,
                       nodelist=freshwater,
                       node_color='r',
                       node_size=600,
                       alpha=0.8)

nx.draw_networkx_nodes(waterG,pos,
                       nodelist=saltwater,
                       node_color='b',
                       node_size=500,
                       alpha=0.8)

nx.draw_networkx_labels(waterG, pos, font_size=15)


# edges
edgewidth = [d['weight'] for (u,v,d) in waterG.edges(data=True)]

nx.draw_networkx_edges(waterG,pos,width=edgewidth, alpha=0.5)

plt.savefig("water.png", format = "png")
```


![png](Network_Placement_files/Network_Placement_83_0.png)


### Network Visualization

core–periphery structure
https://en.wikipedia.org/wiki/Core%E2%80%93periphery_structure#Uses_in_economics


```python
pos = nx.spring_layout(G) 
plt.figure(figsize = (40,30))
nx.draw_networkx(G, pos)
plt.savefig("network.png", format = "png", dpi = 500)
```


![png](Network_Placement_files/Network_Placement_86_0.png)



```python
pos = nx.spring_layout(GG) 
plt.figure(figsize = (40,30))
nx.draw_networkx(GG, pos, node_size=100, alpha=0.3, font_size=10)
plt.savefig("networkglobal.png", format = "png", dpi = 500)
```


![png](Network_Placement_files/Network_Placement_87_0.png)



```python
#create network plots for all years
for i in range(8):
    plt.figure(figsize = (30,30))
    nx.draw_networkx(subG[i])
    name = "network"+"201"+str(i)+'.png'
    plt.savefig(name, format = "png", dpi=500)
    plt.clf()
```


    <Figure size 2160x2160 with 0 Axes>



    <Figure size 2160x2160 with 0 Axes>



    <Figure size 2160x2160 with 0 Axes>



    <Figure size 2160x2160 with 0 Axes>



    <Figure size 2160x2160 with 0 Axes>



    <Figure size 2160x2160 with 0 Axes>



    <Figure size 2160x2160 with 0 Axes>



    <Figure size 2160x2160 with 0 Axes>



```python
#pos=nx.get_node_attributes(G,'pos')
pos = nx.spring_layout(G) 
plt.figure(figsize = (50,40))
#nx.draw(G,pos)
nx.draw_networkx_nodes(G, pos, nodes = G.nodes())
edgewidthG = [d['weight'] for (u,v,d) in G.edges(data=True)]
labels = nx.get_edge_attributes(G,'weight')
nx.draw_networkx_edges(G,pos,width=edgewidthG, alpha=0.5)
plt.savefig("network_label.png", format = "png", dpi = 500)
```


![png](Network_Placement_files/Network_Placement_89_0.png)



```python
##export data
nx.write_gexf(G, 'econ_network.gexf')
nx.write_gexf(GG, 'econ_network_global.gexf')
```

###  Rise/Decline


#### Placement in Top 100 US


```python
def getdelta(df, yrnum = 3):
    '''
    This function 
    1. takes in the dataframe of all placement and yrnum(number of years with quality placement)
    2. aggregate by Source and Year
    3. Reformat the table by breaking down years
    4. Calculate the delta (changes) of placement quality 
    '''
    global rd
    rd = df.groupby(['Source', 'Year'], as_index=False).agg({'Weight':'mean'})
    rd = rd.pivot(index=rd.Source, columns='Year')['Weight'].reset_index()
    rd.set_index('Source', inplace=True)   
    
    #Calculate the delta (changes) of placement quality 
    deltalist = []
    for s in rd.index:
        l = [i for i in rd.loc[s].dropna()]
        delta = 0
        if len(l)>yrnum:
            for i in range(len(l)-1):
                delta += l[i+1]-np.mean(l[:i+1])
 
            deltalist.append(delta/len(l))
        else:
            deltalist.append(np.nan)

    rd['delta'] = deltalist.copy()
        
    rd['color'] = ['red' if x < 0 else 'green' for x in rd['delta']]            
            
            
    return rd
```


```python
rd1 = getdelta(df)
```


```python
#Rise
decline1 = rd1.sort_values(by=['delta'], ascending=False).head(5)
decline1
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
      <th>Year</th>
      <th>2010.0</th>
      <th>2011.0</th>
      <th>2012.0</th>
      <th>2013.0</th>
      <th>2014.0</th>
      <th>2015.0</th>
      <th>2016.0</th>
      <th>2017.0</th>
      <th>delta</th>
      <th>color</th>
    </tr>
    <tr>
      <th>Source</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>state university of new york at binghamton</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>57.0</td>
      <td>6.500000</td>
      <td>43.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>435.0</td>
      <td>90.062500</td>
      <td>green</td>
    </tr>
    <tr>
      <th>washington university in st. louis</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>127.0</td>
      <td>88.000000</td>
      <td>244.0</td>
      <td>NaN</td>
      <td>389.000000</td>
      <td>NaN</td>
      <td>83.375000</td>
      <td>green</td>
    </tr>
    <tr>
      <th>university of texas at austin</th>
      <td>43.0</td>
      <td>74.50</td>
      <td>NaN</td>
      <td>91.666667</td>
      <td>149.5</td>
      <td>NaN</td>
      <td>190.000000</td>
      <td>233.0</td>
      <td>61.299074</td>
      <td>green</td>
    </tr>
    <tr>
      <th>brown university</th>
      <td>70.0</td>
      <td>292.50</td>
      <td>98.0</td>
      <td>NaN</td>
      <td>325.5</td>
      <td>172.000</td>
      <td>404.500000</td>
      <td>150.8</td>
      <td>60.480952</td>
      <td>green</td>
    </tr>
    <tr>
      <th>princeton university</th>
      <td>96.0</td>
      <td>324.75</td>
      <td>247.0</td>
      <td>435.000000</td>
      <td>239.0</td>
      <td>285.125</td>
      <td>289.333333</td>
      <td>234.5</td>
      <td>54.602827</td>
      <td>green</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Decline
rise1 = rd1.sort_values(by=['delta'], ascending=True).head(5)
rise1.head()
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
      <th>Year</th>
      <th>2010.0</th>
      <th>2011.0</th>
      <th>2012.0</th>
      <th>2013.0</th>
      <th>2014.0</th>
      <th>2015.0</th>
      <th>2016.0</th>
      <th>2017.0</th>
      <th>delta</th>
      <th>color</th>
    </tr>
    <tr>
      <th>Source</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>penn state university</th>
      <td>566.0</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>566.000000</td>
      <td>143.0</td>
      <td>74.0</td>
      <td>NaN</td>
      <td>146.000000</td>
      <td>-193.050000</td>
      <td>red</td>
    </tr>
    <tr>
      <th>university of virginia</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>435.00</td>
      <td>64.000000</td>
      <td>44.0</td>
      <td>435.0</td>
      <td>NaN</td>
      <td>76.000000</td>
      <td>-98.200000</td>
      <td>red</td>
    </tr>
    <tr>
      <th>university of arizona</th>
      <td>NaN</td>
      <td>291.000000</td>
      <td>222.50</td>
      <td>187.333333</td>
      <td>95.0</td>
      <td>98.5</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>-75.397222</td>
      <td>red</td>
    </tr>
    <tr>
      <th>duke university</th>
      <td>NaN</td>
      <td>231.666667</td>
      <td>113.75</td>
      <td>123.000000</td>
      <td>83.0</td>
      <td>63.5</td>
      <td>146.0</td>
      <td>55.666667</td>
      <td>-51.893452</td>
      <td>red</td>
    </tr>
    <tr>
      <th>university of california-davis</th>
      <td>NaN</td>
      <td>142.000000</td>
      <td>106.50</td>
      <td>43.000000</td>
      <td>41.0</td>
      <td>50.5</td>
      <td>50.0</td>
      <td>13.000000</td>
      <td>-41.615476</td>
      <td>red</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Issue
plt.hist(rd1.count(axis=1), density=True)
plt.xlabel('Number of Years with Top 100 US Placements')
plt.ylabel('Ratios')
plt.title('Most of the universities have top placements only in few years')
```




    Text(0.5, 1.0, 'Most of the universities have top placements only in few years')




![png](Network_Placement_files/Network_Placement_97_1.png)



```python
extreme1 = pd.concat([rise1, decline1])
plt.figure(figsize=(30,30), dpi= 100, facecolor='w', edgecolor='k')
#plt.style.use('fivethirtyeight')
plt.style.use('seaborn-white')

extreme1.sort_values('delta', inplace=True)
plt.hlines(y=extreme1.index, xmin=0, xmax=extreme1.delta, color=extreme1.color, alpha=0.4, linewidth=30)
plt.gca().set(ylabel='$Programs$', xlabel='$Changes$')
plt.yticks(extreme1.index, extreme1.index, fontsize=40)
plt.title('Top Rise & Decline of US Top100 Placement Quality', fontdict={'size':60})
plt.grid(linestyle='--', alpha=0.5)
plt.savefig('usrisedecline.png')
```


![png](Network_Placement_files/Network_Placement_98_0.png)


#### Placement in Global Universities

Previous problem: Most of the universities have top placements only in few years

* Consider all academic placements
* Use Tilburg Publication Number As Proxy
* Use discountfactors for non-tenure track positions


```python
#Apply getdelta function
rd2 = getdelta(glo)
```


```python
#Rise
rise2 = rd2.sort_values(by=['delta'], ascending=False).head(5)
rise2.head()
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
      <th>Year</th>
      <th>2010</th>
      <th>2011</th>
      <th>2012</th>
      <th>2013</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
      <th>2017</th>
      <th>delta</th>
      <th>color</th>
    </tr>
    <tr>
      <th>Source</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>carnegie mellon university</th>
      <td>7.000000</td>
      <td>8.000000</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>3.333333</td>
      <td>226.866667</td>
      <td>58.900000</td>
      <td>151.000000</td>
      <td>50.894815</td>
      <td>green</td>
    </tr>
    <tr>
      <th>university of connecticut</th>
      <td>1.500000</td>
      <td>21.000000</td>
      <td>0.000000</td>
      <td>213.000000</td>
      <td>26.000000</td>
      <td>157.600000</td>
      <td>49.000000</td>
      <td>NaN</td>
      <td>37.903571</td>
      <td>green</td>
    </tr>
    <tr>
      <th>columbia university in the city of new york</th>
      <td>NaN</td>
      <td>44.400000</td>
      <td>192.500000</td>
      <td>169.076923</td>
      <td>237.100000</td>
      <td>166.080000</td>
      <td>106.666667</td>
      <td>160.977778</td>
      <td>36.998264</td>
      <td>green</td>
    </tr>
    <tr>
      <th>university of rochester</th>
      <td>5.500000</td>
      <td>75.000000</td>
      <td>125.500000</td>
      <td>NaN</td>
      <td>8.500000</td>
      <td>82.571429</td>
      <td>83.400000</td>
      <td>146.200000</td>
      <td>32.900510</td>
      <td>green</td>
    </tr>
    <tr>
      <th>brown university</th>
      <td>81.166667</td>
      <td>164.733333</td>
      <td>96.088889</td>
      <td>83.657143</td>
      <td>358.500000</td>
      <td>92.800000</td>
      <td>188.533333</td>
      <td>133.833333</td>
      <td>29.802918</td>
      <td>green</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Decline
decline2 = rd2.sort_values(by=['delta'], ascending=True).head(5)
decline2.head()
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
      <th>Year</th>
      <th>2010</th>
      <th>2011</th>
      <th>2012</th>
      <th>2013</th>
      <th>2014</th>
      <th>2015</th>
      <th>2016</th>
      <th>2017</th>
      <th>delta</th>
      <th>color</th>
    </tr>
    <tr>
      <th>Source</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>rice university</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>215.333333</td>
      <td>128.000000</td>
      <td>0.000000</td>
      <td>NaN</td>
      <td>24.000000</td>
      <td>NaN</td>
      <td>-87.361111</td>
      <td>red</td>
    </tr>
    <tr>
      <th>penn state university</th>
      <td>194.375000</td>
      <td>85.4</td>
      <td>56.633333</td>
      <td>165.914286</td>
      <td>35.953846</td>
      <td>41.142857</td>
      <td>56.836364</td>
      <td>48.142857</td>
      <td>-47.134322</td>
      <td>red</td>
    </tr>
    <tr>
      <th>cornell university</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>121.500000</td>
      <td>112.700000</td>
      <td>12.750000</td>
      <td>45.888889</td>
      <td>-37.394444</td>
      <td>red</td>
    </tr>
    <tr>
      <th>yale university</th>
      <td>235.161905</td>
      <td>206.9</td>
      <td>198.000000</td>
      <td>163.640000</td>
      <td>140.325000</td>
      <td>126.225000</td>
      <td>134.105882</td>
      <td>173.200000</td>
      <td>-33.413529</td>
      <td>red</td>
    </tr>
    <tr>
      <th>california institute of technology</th>
      <td>150.400000</td>
      <td>30.0</td>
      <td>226.120000</td>
      <td>73.300000</td>
      <td>102.000000</td>
      <td>86.833333</td>
      <td>79.000000</td>
      <td>3.500000</td>
      <td>-28.740272</td>
      <td>red</td>
    </tr>
  </tbody>
</table>
</div>




```python
extreme2 = pd.concat([rise2, decline2])
```


```python
plt.figure(figsize=(30,30), dpi= 100, facecolor='w', edgecolor='k')
#plt.style.use('fivethirtyeight')
plt.style.use('seaborn-bright')

extreme2.sort_values('delta', inplace=True)
plt.hlines(y=extreme2.index, xmin=0, xmax=extreme2.delta, color=extreme2.color, alpha=0.4, linewidth=30)
plt.gca().set(ylabel='$Programs$', xlabel='$Changes$')
plt.yticks(extreme2.index, extreme2.index, fontsize=40)
plt.title('The Rise & Decline of Global Placement Quality', fontdict={'size':60})
plt.grid(linestyle='--', alpha=0.5)
plt.savefig('globalrisedecline.png')
```


![png](Network_Placement_files/Network_Placement_104_0.png)

