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

## 連接資料庫、建立資料表

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

# 建立資料表操作對象 sessionmaker
# 建立一個基礎類別，對應到這些表格的類別
session = db_session()
Base = declarative_base()       
```

建立

```python
# 建立資料表
class Information(Base):
    __tablename__ = 'information'
    year = Column(Integer(), primary_key=True)
    rank = Column(Integer(), primary_key=True)
    name = Column(String(50), nullable=True)
    rate = Column(Float, nullable=True)

# 通過Base.metadata找到所有繼承 Base 的資料表class
Base.metadata.create_all(bind=engine)
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

Step 2 製作Web API
----------------------------

```python app.py
from flask import Flask
from flask import jsonify, request
from database import db_session, init_db
from information import Information


app = Flask(__name__)

#在接受第一個request的時候，可以執行這個function，目的:初始化資料庫
@app.before_first_request
def init():
    init_db()

#在請求結束或應用程式關閉，可以正確的移除database的sessions
@app.teardown_appcontext
def shutdown_session(exception=None):
    db_session.remove()


@app.route('/avengers/all', methods=['GET'])
def avengers_all():
    return jsonify(avengers)


if __name__ == '__main__':
    #有修改才會自動更新
    app.jinja_env.auto_reload = True
    #Debug模式
    app.run(debug=True)
```