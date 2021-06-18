---
layout:     post
title:      "Docker로 AWS Lambda Layer 만들기"
date:       2020-04-26 12:00:00
author:     "yellie"
header-style: text
tags:
    - AWS
    - Lambda
    - Docker
---

AWS Lambda에 코드를 업로드할때 만약 C 또는 C++을 기반으로 한 라이브러리가 포함되어 있다면 Lambda는 이 라이브러리를 컴파일 하지 못한다.

그 이유는 AWS Lambda는 Amazon linux 운영체제에서 실행되고 내 로컬은 Unix 기반의 macOS에서 실행되기 때문이다. 
즉, Lambda와 내 컴퓨터의 운영체제는 서로 달라서 Unix에서 컴파일한 파일을 Linux에서 컴파일 하지 못한다. 이러한 경우 어떻게 해야할까?

이럴때는 **도커로 Linux환경를 생성하여 해당 라이브러리를 컴파일한 후 Lambda layer를 생성**해줘야 한다. 이번 글에서는 이 과정을 다룬다 :)

## 튜토리얼
튜토리얼로 pillow layer를 만들어서 Lambda layer로 올리는 것 까지 해볼 예정이다.

1) Lambda 실행 환경과 동일한 amazon linux 컨테이너를 실행시킨다.
```
docker run -it amazonlinux
```
![image](https://user-images.githubusercontent.com/49056225/122502852-610b3a00-d032-11eb-9696-5a1b065f2fbb.png)

2) 컨테이너에 python 3.7을 설치하고 폴더 안에 가상환경을 만든다.

```
yum -y upgrade 
yum -y update 
yum groupinstall -y development

# python 3.7 설치 
yum -y install python37 
yum -y install python3-devel

# 폴더 생성
mkdir pillow-layer
cd pillow-layer

# 가상환경 설치 후 활성화
python3 -m venv env 
. env/bin/activate
```
![image](https://user-images.githubusercontent.com/49056225/122502920-797b5480-d032-11eb-824b-ec3d5b0f8d3d.png)

3) 가상환경에 pillow를 설치한다.
```
pip install pillow
```

pip으로 설치한 패키지는 site-packages 폴더 안에 저장되므로 site-packages 폴더 안으로 들어가서 pillow와 관련된 파일 이름을 확인한다.

```
find . -name 'site-packages'
cd [가상환경 패키지 경로]
```

![image](https://user-images.githubusercontent.com/49056225/122502974-957ef600-d032-11eb-83d7-90200fe0c8fd.png)

- PIL
- Pillow-7.1.1.dist-info
- Pillow.libs

pillow와 관련된 파일은 위의 3개이다. 이 파일만 들어있는 새로운 site-packages 폴더를 만든다.

```
# 원래 폴더 이름을 tmp으로 변경
cd .. 
mv site-packages tmp

# site-packages 폴더를 새로 생성 후 필요한 파일 복사
mkdir site-packages 
cp -r tmp/PIL site-packages 
cp -r tmp/Pillow* site-packages
```

4) pillow-layer 폴더로 이동한 후 새로 만든 site-packages를 복사한다. **(꼭 python/lib/python3.7/site-packages 경로로 만들어줘야 한다.)**
```
mkdir -p python/lib/python3.7 && cp -r env/lib/python3.7/site-packages $_
```

컨테이너에서 detach 후 python 폴더를 로컬에 복사한다.

```
docker cp [컨테이너ID]:/pillow-layer/python .
```

5) AWS Lambda 에서 layer를 생성하고, zip으로 압축한 python 폴더를 업로드한다.
![image](https://user-images.githubusercontent.com/49056225/122503150-e5f65380-d032-11eb-83b2-856cfd1a19f0.png)
![image](https://user-images.githubusercontent.com/49056225/122503168-edb5f800-d032-11eb-817c-ce0e83f9b8a7.png)
Lambda function에서 방금 만든 pillow layer를 추가해서 사용하면 된다. 끝!

## 마치며
이 과정을 스스로 하기 까지 2주 정도 삽질을 했다. 도커를 하나도 모르던 상태였기 때문에 도커를 처음부터 공부해야했고 site-packages 폴더도 처음으로 살펴보았다. 
리눅스 명령어도 친숙하지 않아서 삽질한 시간이 더 길어졌던 것 같다.

이렇게 정리하고 보니 별게 아닌 것 같아서 조금 속상하지만 이제 원하는 lambda layer를 뚝딱 만들어낼 수 있으니 뿌듯하다.
