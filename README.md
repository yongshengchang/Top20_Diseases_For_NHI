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
$ pip install Flask-SQLAlchemy
$ pip inStall pyexcel_ods
$ pip install pyinstaller
```

Step 1 解析 ODS 格式的檔案並寫入資料庫
----------------------------

## 連接資料庫

連接

```python
from sqlalchemy import create_engine, Column, Integer, String, Float, PrimaryKeyConstraint
from sqlalchemy.orm import scoped_session, sessionmaker
from sqlalchemy.ext.declarative import declarative_base
import os


# 取得目前的路徑
current_dir = os.path.dirname(__file__)

# 建立資料庫鏈接
engine = create_engine('sqlite:///{}/Cnhimi.db'.format(current_dir), convert_unicode=True)
db_session = scoped_session(sessionmaker(autocommit=False,
                                         autoflush=False,
                                         bind=engine))

# 建立一個基礎類別，對應到這些表格的類別
Base.metadata.create_all(bind=engine)

# 建立資料表操作對象 sessionmaker
# 通過Base.metadata找到所有繼承 Base 的數據表class
session = db_session()
Base = declarative_base()                                                                       
```

## 存取、解析 ODS 格式的檔案

解析

```python
from pyexcel_ods import get_data

# 擷取所需資料範圍
data = get_data("2017年國人全民健康保險就醫疾病資訊-1070906.ods", start_row=3, row_limit=20)    

# 取得特定SHEET
TABLE_1 = json.dumps(data["表1"], ensure_ascii=False)                                         
```

存取

```python
from sqlalchemy.orm import scoped_session, sessionmaker
import os

 # 將解析完所需資料寫入資料庫
for TABLE in data["表1"]:
    information = Information(year=i, rank=TABLE[0], name=TABLE[2], rate=TABLE[9])
    session.add(information)     

#寫進資料庫
session.commit()

#關閉資料庫
session.close()
```