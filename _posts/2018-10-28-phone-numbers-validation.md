---
title: "Fun Project: Phone Numbers Validation Tool using NumVerify API"
date: 2018-10-27
tags: [api, random digit dialing, validation]
header:
  image: "/images/random.png"
excerpt: "api, random digit dialing, validation "
mathjax: "true"
---





### 

Author: LI LIU (1st yr master's student at UChicago)

10/26/18: basic framework

10/27/18: add multiple conditions for verifying valid numbers


```python
import pandas as pd
```


```python
#Read in file
Num = pd.read_excel('PhoneSurvey.xlsx', 
                    names=['ID','Phone','Response','Party','Age'])
```

#### Example: Random-genrated Numbers in Philadelphia (Code 267)


```python
Num["Code"] = 267
Num["PN"] = Num["Code"].astype(str) + Num["Phone"].astype(str)
Num.head()
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
      <th>Phone</th>
      <th>Response</th>
      <th>Party</th>
      <th>Age</th>
      <th>Code</th>
      <th>PN</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>4310454</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2674310454</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>6920338</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2676920338</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>6918070</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2676918070</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>3605305</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2673605305</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>8892229</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2678892229</td>
    </tr>
  </tbody>
</table>
</div>



### Go to https://numverify.com/documentation
### Create a free account to get access key (up to 250 API requests)

#### Define the fucntion


```python
import urllib.request, json 
list=[]

#Replace with access key
key = "###"

def validtest(number):
    '''
    Return the indicator list of whether number is valid or not.
    '''
    link = "http://apilayer.net/api/validate?access_key="+key+"&number="+\
            number + "&country_code=US&format=1"
    with urllib.request.urlopen(link) as url:
         data = json.loads(url.read().decode())
            
         #Specify conditions for valid numbers
         if data["valid"] == True and data['carrier'] !='' and data['location'] !='' and data['line_type'] !='':
            list.append(1)
         else:
            list.append(0)
                
    return data

```

#### Run the function


```python
for num in Num["PN"]:
    validtest(num)
res = pd.Series(list)
Num["Label"] = res.values
```


```python
#Counts of valid numbers
sum(list)
```




    60




```python
#Test single number
#validtest("my number")
```


```python
Num.head()
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
      <th>Phone</th>
      <th>Response</th>
      <th>Party</th>
      <th>Age</th>
      <th>Code</th>
      <th>PN</th>
      <th>Label</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>4310454</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2674310454</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>6920338</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2676920338</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>6918070</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2676918070</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>3605305</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2673605305</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>8892229</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>267</td>
      <td>2678892229</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



#### Save the output


```python
Num.to_excel('data.xlsx')
```
