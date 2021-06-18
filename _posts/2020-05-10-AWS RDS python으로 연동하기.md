---
layout:     post
title:      "AWS RDS(PostgreSQL) python으로 연동하기"
date:       2020-05-10 12:00:00
author:     "yellie"
header-style: text
tags:
    - AWS
    - RDS
---

AWS 뿌시기 3탄은 **RDS**이다.

AWS를 이용해서 개발하다보면 DB 연결은 필수라고 생각하여 3번째 주제로 정해보았다. 이번 글에서는 AWS RDS가 무엇인지 살펴본 후, python으로 AWS RDS의 PostgreSQL과 연동해본다.

## RDS?
RDS는 AWS에서 제공하는 **관리형 RDB(Relational Database) 서비스**이다. 지원하는 엔진은 MySQL, PostgreSQL, MariaDB, Oracle, SQL Server가 있다.

그러나 이런 엔진들이 클라우드 환경에 최적화 되어있지않다. 이러한 문제를 해결하기 위해 AWS는 자사의 클라우드 환경에 최적화된 새로운 RDB를 만들었다. 
그것이 바로 **Aurora DB** 이다. Aurora DB는 MySQL 및 PostgreSQL과 호환 가능하다.

회사에서는 Amazon Aurora에 PostgreSQL를 호환하여 사용하고 있다.

## 튜토리얼
python으로 Amazon Aurora PostgreSQL와 연동한 후 쿼리 실행까지 해 볼 예정이다.

1) 가상환경에 psycopg2를 설치한다.
```
pipenv install psycopg2-binary
```
- psycopg는 python과 PostgreSQL을 연결하는 모듈이다.

2) PostgreSQL DB에 접근하여 connection과 cursor를 생성한다.
```
import traceback
import psycopg2
from psycopg2.extras import RealDictCursor

try:
    conn = psycopg2.connect(host=HOST, dbname=DBNAME, user=USER, password=PWD)
    cur = conn.cursor(cursor_factory=RealDictCursor)
    
except Exception:
    print('Fail to connect DB')
```
- **RealDictCursor**를 이용하여 cursor를 생성하면 쿼리를 날렸을때 결과물을 json 형식으로 이쁘게 보여준다.
- DB 정보인 HOST, DBNAME, USER, PWD는 중요한 정보이기 때문에 config.json으로 따로 관리한다.

### SELECT
![image](https://user-images.githubusercontent.com/49056225/122504573-89e0fe80-d035-11eb-836e-f53e5e212460.png)
```
query = 'SELECT * FROM employees WHERE id = 1'

try:            
    cur.execute(query)            
    result = cur.fetchall()        
except Exception as e:            
    msg = traceback.format_exc()            
    msg += '\n\n Query: \n' + query            
    print(msg)            
    cur.close()
            
cur.close()
print(result)
```
![image](https://user-images.githubusercontent.com/49056225/122504606-99f8de00-d035-11eb-8a9c-7bf5416464aa.png)

### INSERT
```
query = "INSERT INTO employees (id, name, gender, birth_date) VALUES (4, 'cosmo Kramer', 'female', '1988-04-03')"

try:
    cur.execute(query)
except Exception as e:
    msg = traceback.format_exc()
    msg += '\n\n Query: \n' + query
    print(msg)
    
    cur.close()
    conn.rollback()
    
cur.close()        
conn.commit()
```
- INSERT는 성공시에는 commit을, 실패한 경우에는 rollback를 해준다.
![image](https://user-images.githubusercontent.com/49056225/122504727-d298b780-d035-11eb-80c4-08e832a66870.png)

## DBManager
S3Manager처럼 DBManager 클래스를 만들어 필요한 곳에 임포트해서 사용하고 있다.
<script src="https://gist.github.com/seoyeonhwng/1b6b671ef2b83e09ba9fbbf2f4798d5b.js"></script>

