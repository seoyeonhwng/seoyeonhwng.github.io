---
layout:     post
title:      "AWS Lambda란 무엇인가"
date:       2020-03-15 12:00:00
author:     "yellie"
header-style: text
tags:
    - AWS
    - Lambda
---

AWS 뿌시기 1탄은 Lambda다.

회사에서 주로 Lambda를 이용하여 개발하기 때문에 Lambda는 내가 가장 많이 써본 AWS 서비스이다. 앞으로 정리할 AWS 글에서 람다가 자주 등장할 것 같아 가장 먼저 정리해보았다. 렛츠 고!

## Lambda란?
Lambda는 AWS에서 제공하는 **서버리스 컴퓨팅 플랫폼**이다.

서버리스란, 서버가 없다는 뜻이 아니고 **개발자가 서버의 존재를 신경쓸 필요가 없다**는 뜻이다. 서버가 잘 돌아가고 있는지, 
개수와 사양한 적당한지 등등 신경쓸 필요없이 사용자는 오직 코드에만 집중할 수 있으니 무척 편하다.

이때 사용한 컴퓨팅 시간, 용량에 대해서만 AWS에게 비용을 지불하면 된다.

## 언제 쓰면 좋을까?
특정 시기에만 코드를 실행시키거나 특정 경우에만 많은 리소스가 필요한 경우 Lambda를 사용하면 유용하다.

예를 들면
- 서버 띄우지 않고 코드를 실행하고 싶은 경우
- 평소에는 로그인 + 글쓰기 정도인 웹사이트에서 썸네일을 생성하는 경우 *(썸네일 생성은 많은 리소스를 요구함)*
- 일주일 동안 이벤트를 진행하여 트래픽이 몰리는 경우

하지만 람다의 단점도 존재한다.
- 코드 용량이 최대 250MB 이다.
- 함수 실행 시간은 최대 15분이다.
- 처음 함수 호출시 Cold Start를 하게되고 초기 지연시간이 발생한다.
- 비싸다.

등등이 있지만 그래도 서버 걱정 없이 오직 코드에만 집중 할 수 있다는 것은 굉장히 편하고 합리적인 것은 분명하다.

## 튜토리얼
python을 이용하여 ‘hello world’를 출력하는 람다 함수를 만들어보자.

1. AWS Lambda로 들어간다.
![image](https://user-images.githubusercontent.com/49056225/121773012-2d548e00-cbb4-11eb-89da-0c05a81d1a8d.png)

2. 오른쪽 상단에 `함수 생성` 버튼을 누른다.
![image](https://user-images.githubusercontent.com/49056225/121773021-3c3b4080-cbb4-11eb-9bc8-03d5fd5b5fc1.png)

3. 만들고자 하는 람다 함수 이름과 사용 언어를 선택한 후 `함수 생성` 버튼을 누른다.
![image](https://user-images.githubusercontent.com/49056225/121773045-4d844d00-cbb4-11eb-8abc-820d010fbd71.png)

함수 이름은 hello_world, 사용 언어는 python 3.7로 선택했다.
![image](https://user-images.githubusercontent.com/49056225/121773055-5c6aff80-cbb4-11eb-8aca-ba607abdce73.png)

4. 코드 인라인에 원하는 코드를 추가한다.
![image](https://user-images.githubusercontent.com/49056225/121773064-68ef5800-cbb4-11eb-977b-288ae7f7b158.png)

lambda_handler 함수에 hello world 출력문을 추가해주었다.

### 핸들러
람다 함수에게 어디서부터 코드를 실행해! 라고 알려주는 부분이다.

예를 들어, 핸들러가 `lambda_function.lambda_handler`라면 이 람다 함수가 실행될때 lambda_function.py라는 파일 안에 정의된 lambda_handler 함수를 실행한다는 의미이다. 
`filename.handler-method` 형식으로 작성한다.

5. 아래에 있는 기본 설정에서 실행에 필요한 메모리와 시간을 설정한다.
![image](https://user-images.githubusercontent.com/49056225/121773085-8ae8da80-cbb4-11eb-8987-796225fa3a1a.png)
![image](https://user-images.githubusercontent.com/49056225/121773092-976d3300-cbb4-11eb-9df7-6af162df5d68.png)

메모리는 256MB, 실행 시간은 최대 5분으로 설정했다.

6. 상단에 `저장` 버튼을 누른다.
![image](https://user-images.githubusercontent.com/49056225/121773108-af44b700-cbb4-11eb-8cef-2a8fc8502d66.png)

7. 테스트 이벤트를 생성하여 람다 함수를 실행한다.

테스트 이벤트 구성 버튼을 클릭하여 이벤트를 만들어보자
![image](https://user-images.githubusercontent.com/49056225/121773123-bec40000-cbb4-11eb-83a5-db866688a77b.png)
![image](https://user-images.githubusercontent.com/49056225/121773164-02b70500-cbb5-11eb-805c-d9a44e9ca8b6.png)
![image](https://user-images.githubusercontent.com/49056225/121773173-0d719a00-cbb5-11eb-9177-6e34dd5bfdf1.png)

방금 만든 이벤트를 선택하고 `테스트 버튼`을 누르면 로그에 hello world가 출력된 것을 볼 수 있다. 잘 만들었다!

![image](https://user-images.githubusercontent.com/49056225/121773183-1a8e8900-cbb5-11eb-8dea-12f2e0b5668a.png)


## 마치며
AWS lambda에 코드를 올리는 방법은 3가지이다.

- 인라인 코드
- zip파일을 업로드
- 아마존 S3에서 파일 업로드

오늘은 1번 방법으로 코드를 업로드하였다. 다음 시간에는 2번, 3번 방법을 이용하여 스크립트로 간단하게 업로드하는 방법에 대해 다룰 예정이다.




