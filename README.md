# Top20_Diseases_For_NHI

國人全民健康保險就醫疾病資訊
----------------------------

存取、解析 國人全民健康保險就醫疾病資訊的 ODS 格式的檔案並製作成WebAPI

資料來源: 

* [國人全民健康保險就醫疾病資訊]](https://data.nhi.gov.tw/Datasets/DatasetDetail.aspx?id=531&Mid=A111068)

## Tool

* PuCharm

## Install

By PyPi

```
$ pip install Flask
$ pip inStall pyexcel_ods
```

解析 ODS 格式的檔案並回傳JSON格式資料
----------------------------

```python
from flask import jsonify
from pyexcel_ods import get_data

#使用get_data讀取ODS檔案
data = get_data("2017年國人全民健康保險就醫疾病資訊-1070906.ods", start_row=3, row_limit=20)

results = []
d = {} 
d["表1"] = []  
d["表2"] = [] 
d["表3"] = []  
d["表4"] = [] 
d["表5"] = [] 

#組合表1~表5的資料
def func(i):
  i+=1
  if i< 4 :
    for TABLE in data["表"+str(i)]:
      d["表"+str(i)].append({  
        'list': i,
        'year': 2017,
        'rank': TABLE[0],
        'name': TABLE[2],
        'rate': TABLE[9],
      })
  elif i== 4:
    for TABLE in data["表"+str(i)]:
      d["表"+str(i)].append({  
        'list': i,
        'year': 2017,
        'rank': TABLE[0],
        'name': TABLE[2],
        'rate': TABLE[5],
      })
  else: 
    for TABLE in data["表"+str(i)]:
    d["表"+str(i)].append({  
    'list': i,
    'year': 2017,
    'rank': TABLE[0],
    'name': TABLE[2],
    'rate': TABLE[6],
    })       
  return d["表"+str(i)]

@app.route('/')
def start():
    #Run迴圈執行Function
    for i in range(5):
        results.append(func(i))

    return jsonify(results)
```