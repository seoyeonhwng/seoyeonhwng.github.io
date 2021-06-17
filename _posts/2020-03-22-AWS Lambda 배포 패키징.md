---
layout:     post
title:      "AWS Lambda 배포 패키징"
date:       2020-03-22 12:00:00
author:     "yellie"
header-style: text
tags:
    - AWS
    - Lambda
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
