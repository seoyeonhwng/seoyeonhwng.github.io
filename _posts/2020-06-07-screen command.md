---
layout:     post
title:      "Unix(Linux, Mac) Screen Command"
date:       2021-05-01 12:00:00
author:     "yellie"
header-style: text
tags:
    - screen
---

이번 글은 screen에 대한 글이다.

요즘 회사에서 데이터 파이프 라인을 구축하고 있다. 새로 구축한 데이터 웨어 하우스에 최근 1년치 과거 데이터를 덤프해야 했다. EC2상에서 약 5시간 정도 소요 될 작업이었다.

EC2는 ip 주소가 바뀌면 ec2 연결이 끊어지기 때문에 이동도 못하고 코드는 잘 돌고있는지 걱정하면서 끙끙대고 있는 나에게 옆자리 사수분께서 screen을 알려주셨다. 정말 편하고 좋아서 이렇게 글로 정리하게 되었다.

## screen 이란?
screen이란 **하나의 터미널이 여러개의 프로세스를 사용할 수 있게 하는 매니저**다.

screen를 사용하면 Unix에서 여러개의 윈도우를 사용할 수 있다. 즉, screen 명령어를 이용해 터미널 세션에서 detach 할 수 있고 나중에 다시 attach 할 수 있다. 
가장 좋은 점은 터미널 세션에서 detach하여도, screen 세션에서 원래 시작된 프로세스는 screen에 의하여 관리 받기 때문에 여전히 동작하고 있고 있다는 점이다.

쉽게 말해 **screen을 사용하여 코드를 돌리면 마음 편히 잘 수 있다**는 뜻이다:)

## 언제 쓰면 좋을까?
- 하나의 터미널을 여러개로 만들고 터미널 마다 다른 명령어를 실행하고 싶은 경우
- 다른 일을 하면서 데몬이 되지 않고 프로세스를 계속 실행시키고 싶은 경우
> 데몬(daemon) 이란 사용자가 직접적으로 제어하지 않고, 백그라운드에서 돌면서 여러 작업을 하는 프로그램을 뜻한다.
- 세션에 비밀번호가 필요한 경우
- 여러명의 유저가 가능한 세션이 필요한 경우

## 일반적인 사용 순서
1. screen을 새로 생성한다.
2. 해당 screen에서 원하는 작업을 실행한다.
3. 해당 screen에서 detach한다.
4. (한참 뒤에 시간이 흐른 뒤에)
5. 다시 해당 screen에 attach해서 실행시킨 작업이 얼마나 진행되었는지 확인한다.

## 명령어
- screen 설치
```
(yum | apt-get) install screen
```

- 새로운 screen 생성
```
1. screen # default 세션 생성
2. screen -S name # 특정 이름을 가진 세션 생성
```

- screen에서 detach
```
1. Ctrl a d # 현재 세션 detach
2. screen -d SCREENID # 다른 터미널에서 실행되고 있는 세션 deatch
```

- screen 목록 확인
```
screen -ls
```

- screen에 다시 attach
```
1. screen -r # 세션이 하나인 경우 그 세션에 attach
2. screen -r <SCREENID> # 특정 ID의 세션에 attach
3. screen -r name # 특정 이름의 세션에 attach
```

- screen 삭제
```
1. CTRL-a k # attach 상태의 세션 삭제
2. screen -X -S name kill # detach 상태의 세션 삭제
```

- screen lock
```
CTRL-a x
```

## 스크롤 하는 방법
screen 안에서는 터미널처럼 스크롤 할 수 없다.

만약 스크롤을 하고 싶다면 **CTRL-a Esc을 누른 뒤** 상하 방향 키로 움직이면 된다. esc를 누르면 scroll 모드에서 빠져나올 수 있다.

## reference
- <https://dasunhegoda.com/unix-screen-command/263/>
