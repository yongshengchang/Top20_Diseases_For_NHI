# Top20_Diseases_For_NHI

國人全民健康保險就醫疾病資訊
----------------------------

存取、解析 國人全民健康保險就醫疾病資訊的 ODS 格式的檔案並製作成WebAPI

資料來源: 

* [國人全民健康保險就醫疾病資訊]](https://data.nhi.gov.tw/Datasets/DatasetDetail.aspx?id=531&Mid=A111068)

## Prpcess

1. 檢查Data資料夾底下是否有 [ 國人全民健康保險就醫疾病資訊.ods ] 的檔案。
2. 檢查Data資料夾底下是否有 [ 當年度的資料檔案 ]，沒有則自動去官網下載檔案。
3. 解析[ 國人全民健康保險就醫疾病資訊.ods ] 的檔案，擷取各年度ods檔案並下載。
4. 解析各年度ods檔案，回傳JSON格式的資料。

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
from urllib.request import urlretrieve
from flask import jsonify
from pyexcel_ods import get_data
import pyexcel_ods
import urllib
import os

#新增資料夾
os.makedirs( './data', exist_ok=True )

#將資料命名後儲存至資料夾底下
urlretrieve ("https://data.nhi.gov.tw/Datasets/Download.ashx?rid=A21030000I-D2001B-001&l=http://data.nhi.gov.tw/resource/OpenData/%E5%9C%8B%E4%BA%BA%E5%85%A8%E6%B0%91%E5%81%A5%E5%BA%B7%E4%BF%9D%E9%9A%AA%E5%B0%B1%E9%86%AB%E7%96%BE%E7%97%85%E8%B3%87%E8%A8%8A.ods", "./data/國人全民健康保險就醫疾病資訊.ods")

#不需解碼的字元
b = b'/:?='

data = get_data("./data/國人全民健康保險就醫疾病資訊.ods", start_row=1)

#解析並只下載ODS的檔案
for TABLE in data["工作表1"]:
  if TABLE != [] :
    if TABLE[0].find('ODS') > 0 :
      urlretrieve(urllib.parse.quote(TABLE[1],b), "./data/"+TABLE[0][0:TABLE[0].find('O')-1]+".ods")




#使用get_data讀取ODS檔案
data = get_data("./data/2017年國人全民健康保險就醫疾病資訊-1070906.ods", start_row=3, row_limit=20)

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