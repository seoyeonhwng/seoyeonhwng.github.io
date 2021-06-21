---
layout:     post
title:      "AWS Lambda 배포 패키징"
date:       2020-03-22 12:00:00
author:     "yellie"
header-style: text
tags:
    - AWS
---

AWS lambda에 코드를 올리는 방법은 3가지이다.

- 인라인 코드 (용량 제한 : 3MB)
- zip파일을 업로드 (용량 제한 : 50MB)
- 아마존 S3에서 파일 업로드

저번 람다 튜토리얼에서는 1번 방법으로 코드 업로드를 하였다. 이번 글에서는 별도의 프레임워크를 사용하지 않고 스크립트를 이용하여 용량이 큰 코드를 쉽고 간단하게 람다에 배포하는 방법을 다룬다.

## 배포 패키지
코드와 실행에 필요한 라이브러리가 포함되어 있는 **zip파일**을 뜻한다. 이러한 배포 패키지를 만드는 행위를 **배포 패키징** 이라 한다.

## 2번) zip 파일을 업로드
2번 방법은 코드와 라이브러리를 package.zip 으로 압축한 후 직접 람다에 업로드한다. 배포할때마다 zip파일을 직접 업로드 해야 한다는 단점이 있다.

1. 패키징 스크립트(build.sh) 를 작성한다.
<script src="https://gist.github.com/seoyeonhwng/be5919738fef9d05ae5828f0b6774121.js"></script>
> 패키지에서 제거해야하는 라이브러리에는 `numpy`, `boto3`와 같은 Lambda에서 제공하는 라이브러리 또는 `psycopg2`, `pillow`와 같은 C 기반 라이브러리 (따로 Layer를 생성해야함)가 있다.

2. 터미널에서 스크립트 파일(build.sh)에 실행 권한을 준다.
```
> chmod +x build.sh 
```

3. build.sh 파일을 실행하면 package.zip 파일이 생긴다.
```
> ./build.sh 
```

4. `.zip 파일 업로드`를 선택하고 3번에서 생긴 package.zip 파일을 업로드한다.
![image](https://user-images.githubusercontent.com/49056225/122340684-8c801d00-cf7d-11eb-9dc7-ca237962f986.png)

## 3번) S3에서 파일 업로드
만약 package.zip 용량이 50MB보다 크다면 2번 방법처럼 직접 .zip 파일 업로드가 불가능하다. 이럴땐 3번 방법을 이용해야 한다.

3번 방법은 S3 `lambda-function-support` 버킷에 배포 패키지를 업로드 하고 해당 패키지를 Lambda에 등록한다. 회사에서는 이 방법을 사용하고 있다.

AWS CLI가 설치되어 있지 않다면
```
> pip install --upgrade --user awscli
```

AWS 계정 세팅이 되어 있지 않다면 아래 명령어를 실행하여 ACCESS KEY, SECRET KEY 등을 설정한다.
```
> aws configure 
```

1. 배포 스크립트(deploy.sh) 를 작성한다.
<script src="https://gist.github.com/seoyeonhwng/2ae5e86a10379be8ae1de4ddf2d8c9ad.js"></script>
- 변경해야 할 값
        - test -> 본인의 람다 함수 이름
        - lambda_function_support -> .zip을 저장할 S3 버킷 이름
- aws lambda 명령어
        - update-function-code : 함수에 코드를 업로드하는 명령어
        - update-function-configuration : 함수의 환경변수 설정하는 명령어
[더 많은 명령어가 궁금하다면?](https://docs.aws.amazon.com/cli/latest/reference/lambda/index.html)

2. 터미널에서 스크립트 파일(deploy.sh)에 실행 권한을 준다.
```
> chmod +x deploy.sh
```

3. deploy.sh 실행하면 빌드 후 배포까지 한번에 실행된다.
```
> ./deploy.sh
```

## 마치며
이렇게 스크립트를 이용하면 따로 별도의 프레임워크를 설치할 필요 없이 쉽게 배포할 수 있으니 무척 편하고 간단하다!

다음 글에서는 S3에 대하여 자세하게 다뤄볼 예정이다.
