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
from pyexcel_ods import get_data
from urllib.request import urlretrieve
from flask import jsonify
import urllib
import pyexcel_ods
import json
import os

#新增所需陣列
results = []
d = {} 
d["表1"] = []  
d["表2"] = [] 
d["表3"] = []  
d["表4"] = [] 
d["表5"] = [] 

#新增資料夾
os.makedirs( './', exist_ok=True )

#不需解碼的字元
b = b'/:?='

#檢查檔案是否存在，否就去下載
if not os.path.isfile(filepath):
  os.makedirs( '.', exist_ok=True )
  urlretrieve ("https://data.nhi.gov.tw/Datasets/Download.ashx?rid=A21030000I-D2001B-001&l=http://data.nhi.gov.tw/resource/OpenData/%E5%9C%8B%E4%BA%BA%E5%85%A8%E6%B0%91%E5%81%A5%E5%BA%B7%E4%BF%9D%E9%9A%AA%E5%B0%B1%E9%86%AB%E7%96%BE%E7%97%85%E8%B3%87%E8%A8%8A.ods", "./國人全民健康保險就醫疾病資訊.ods")
  

#使用get_data取得[國人全民健康保險就醫疾病資訊.ods]檔案，並讀取檔案下載URL
Main_data = get_data("./國人全民健康保險就醫疾病資訊.ods", start_row=1)

#解析並只下載ODS的檔案
for TABLE in Main_data["工作表1"]:
  if TABLE != [] :
    if TABLE[0].find('ODS') > 0 :
      FileName = TABLE[0][0:TABLE[0].find('O')-1]+".ods"
      urlretrieve(urllib.parse.quote(TABLE[1],b), "./data/"+FileName)
      #使用get_data取得ODS檔案並讀取範圍內資料    
      data = get_data("./data/"+FileName, start_row=3, row_limit=20)
      #參數 例:西元年:2016
      startUp(int(FileName[0:4]))


#組合各年度<表1~表5>的資料
#2017年後檔案裡的儲存格有變動，因此另外做處理
#list : 檔案裡的SHEET
#year : 年度
#rank : 排名
#name : 疾病代碼列表群組
#rate : 表1~表4:占率 表5:平均每住院者住院日數(日/人)
def func(_year,_list):
  _list += 1
  if _list <  4 :
    if _year > 2017 :
      for TABLE in data["表"+str(_list)]:
        results.append({  
          'list': _list,
          'year': _year,
          'rank': TABLE[0],
          'name': TABLE[2],
          'rate': TABLE[8],
        })
    else:
      for TABLE in data["表"+str(_list)]:
        results.append({  
          'list': _list,
          'year': _year,
          'rank': TABLE[0],
          'name': TABLE[2],
          'rate': TABLE[9],
        })
  elif _list == 4:
    if _year > 2017 :
      for TABLE in data["表"+str(_list)]:
        results.append({  
          'list': _list,
          'year': _year,
          'rank': TABLE[0],
          'name': TABLE[2],
          'rate': TABLE[4],
        })
    else:
      for TABLE in data["表"+str(_list)]:
        results.append({  
          'list': _list,
          'year': _year,
          'rank': TABLE[0],
          'name': TABLE[2],
          'rate': TABLE[5],
        })
  else: 
    for TABLE in data["表"+str(_list)]:
      results.append({  
      'list': _list,
      'year': _year,
      'rank': TABLE[0],
      'name': TABLE[2],
      'rate': TABLE[6],
      })     


#Run迴圈執行Function
def startUp():    
    for i in range(5):
        func(Syear,i)


@app.route('/')
def start():
    return jsonify(results)

```