---
layout:     post
title:      "Python Global Interpreter Lock(GIL)이란?"
date:       2021-02-19 12:00:00
author:     "yellie"
header-style: text
tags:
    - python
---

파이썬 언어 자체에 대해 공부하다 보면 GIL을 마주치게 된다. GIL이 무엇인지 대략 알고 있지만, 조금 더 깊게 이해하고 싶은 마음에 [관련 글](https://realpython.com/python-gil/)을 읽고 이해한 바를 정리해보았다.

## multi-threading, 왜 쓸까?
multi-threading는 하나의 프로그램을 더 빠르게 실행시키기 위해 사용한다.독립적인 주소 공간을 할당받은 프로세스와 달리 thread들은 자원을 공유하기 때문에 프로세스간 통신보다 간단하고, 오버헤드가 적다. 
그래서 일반적으로 프로그램을 multi-processing보다 multi-threading으로 구현할 때 성능이 더욱 향상된다.
하지만 파이썬으로 구현한 프로그램을 multi-threading으로 구현하여도 성능이 향상되지 않거나 더 느려질 수도 있다. GIL때문에 여러 개의 thread들이 병렬적으로 실행되지 않고, 한 시점에 하나의 thread만이 실행되기 때문이다.

## GIL이란?
GIL은 오직 하나의 thread가 파이썬 interpreter의 제어권을 갖도록 하는 mutex(lock)이다. thread가 파이썬 bytecode를 실행하기 위해서는 GIL을 획득해야 하고, GIL을 반납할 때까지 다른 thread는 파이썬 bytecode를 실행할 수 없다. 
그렇기 때문에 thread가 여러 개 있어도 한 시점에는 오직 하나의 thread만이 실행될 수 있다.

<br><img src="https://user-images.githubusercontent.com/49056225/120757970-373f1700-c54c-11eb-8501-31a4c2b3552d.png" width="600" height="400"><br>


## GIL은 Python의 어떤 문제를 해결해주었을까?
### reference counting
파이썬은 메모리 관리법으로 reference counting을 사용한다. 파이썬에서 생성된 객체들은 그 객체가 참조된 횟수를 저장하는 reference count 변수를 가지고 있다. 
만약 reference count 변수의 값이 0이 되면, 그 객체에게 할당된 메모리는 반납된다.

```
>>> import sys
>>> a = []
>>> b = a
>>> sys.getrefcount(a) # 출력값 : 3
```

위의 예시에서, 빈 리스트 객체 []는 a, b 그리고 `sys.getrefcount()` 의 인자로 3번 참조되었기 때문에 reference count는 3이다.

### 다시 GIL
문제는 두개의 thread가 reference count 변수의 값을 동시에 변경할 수 있다는 것이다. 그렇게 되면 아직 존재하는 객체의 메모리가 반납되는 등 심각한 버그가 발생할 수 있다.

따라서 thread간 공유되는 모든 자료구조에 lock을 추가하여 reference count 변수가 안전하게 보호되도록 해야한다. 하지만 모든 객체마다 lock을 추가하게 되면 Deadlock이 발생할 수 있고, 
반복되는 lock 할당/해제로 인해 성능이 안좋아질 수 있다.

이에 대한 해결책으로 파이썬에서는 모든 객체마다 lock을 할당하지 않고, interpreter 자체에 lock을 할당하였다. 한 개의 lock이 존재하기 때문에 Deadlock이 발생하지 않고, 오버헤드도 크지 않다.

## GIL을 선택한 이유
파이썬은 thread라는 개념이 생기기 전부터 존재했고, 파이썬에 필요한 기능을 기존 C 라이브러리를 확장하여 구현한 extensions들이 이미 많이 존재했다. 
이미 존재하는 C extensions에게 thread-safe한 메모리 관리법이 필요했고, 구현하기 쉬운 GIL이 실용적인 해결책이었다.

또한, 하나의 lock만 관리하기 때문에 single-threaded 프로그램은 성능이 향상되었다. 이러한 장점때문에 GIL은 아직도 제거되지 않고 있다.

## CPU-bound 프로그램과 GIL
CPU-bound 프로그램은 CPU를 많이 사용하는 프로그램을 뜻한다. 행렬곱, image processing과 같이 수학적 계산을 많이 하는 작업이 해당된다. 
CPU-bound 프로그램에서 GIL은 어떤 영향을 끼치는지 코드를 통해 살펴보자.

### CPU-bound single-threaded
```
import time
from threading import Thread

COUNT = 50000000
def countdown(n):
    while n > 0:
        n -= 1
        
start = time.time()
countdown(COUNT) # thread 1개로 처리
end = time.time()

print('Time taken in seconds -', end - start)
```
<img src="https://user-images.githubusercontent.com/49056225/120758446-d06e2d80-c54c-11eb-87e7-8550ca32539d.png" width="600" height="300"><br>

### CPU-bound multi-threaded
```
import time
from threading import Thread

COUNT = 50000000
def countdown(n):
    while n > 0:
        n -= 1
        
# thread 2개로 나눠서 실행
t1 = Thread(target=countdown, args=(COUNT//2,))
t2 = Thread(target=countdown, args=(COUNT//2,))

start = time.time()
t1.start()
t2.start()
t1.join()
t2.join()
end = time.time()

print('Time taken in seconds -', end - start)
```
<img src="https://user-images.githubusercontent.com/49056225/120758571-f7c4fa80-c54c-11eb-8437-b52ce3d77d83.png" width="600" height="300"><br>


CPU-bound 프로그램은 single-threaded보다 multi-threaded로 구현했을때 실행 속도가 (비슷하거나) **더 오래 걸린다**. 
GIL으로 인해 multi-threaded가 single-threaded가 될 뿐만 아니라, lock 할당/해제로 인해 오버헤드가 발생하기 때문이다.

## I/O-bound 프로그램과 GIL
I/O-bound 프로그램은 대부분의 시간을 Input/Output 기다리는데 사용하는 프로그램을 뜻한다. 우리가 평소에 실행하는 대부분의 프로그램은 I/O-bound에 해당된다.

I/O 작업을 수행하는 동안 thread는 Python interpreter를 사용할 필요가 없기 때문에 GIL을 해제한다. 
따라서 CPU-bound 프로그램과 달리 I/O-bound 프로그램은 multi-threaded로 구현하는 경우 thread들이 동시에 실행되기 때문에 성능이 향상된다.

<img src="https://user-images.githubusercontent.com/49056225/120758719-22af4e80-c54d-11eb-9d3b-86521a17bbbd.png" height="600" height="300"><br>

## GIL 피하기
### multi-processing
가장 유명한 방법은 multi-threading 대신 multi-processing을 사용하는 것이다. 
프로세스는 자신만의 interpreter와 메모리 공간을 가지기 때문에 GIL이 문제가 되지 않는다. 파이썬에는 multiprocessing 모듈을 사용하여 프로세스를 쉽게 생성할 수 있다.

```
import time
from multiprocessing import Pool

COUNT = 50000000
def countdown(n):
    while n > 0:
    n -= 1
    
if __name__ == '__main__':
    pool = Pool(processes=2)
    start = time.time()
    r1 = pool.apply_async(countdown, [COUNT//2])
    r2 = pool.apply_async(countdown, [COUNT//2])
    pool.close()
    pool.join()
    end = time.time()
    print('Time taken in seconds - ', end - start)
```
<img src="https://user-images.githubusercontent.com/49056225/120758860-51c5c000-c54d-11eb-9733-4fc86ffe2f8f.png" height="600" height="300"><br>

multi-threaded로 구현했을 때보다 실행 속도가 줄어들었지만, 프로세스 관리 자체에 오버헤드가 발생하기 때문에 실행 속도가 절반으로 줄지 않았다. 
또한 multi-processing은 multi-threaed보다 무겁기 때문에 scaling bottleneck이 발생할 수 있다.

### Alternative Python interpreter
파이썬은 CPython, Jython, IronPython, PyPy 등 여러 개의 interpreter를 가지고 있다. 
GIL은 CPython에만 존재하기 때문에 가능하다면 다른 interpreter를 사용하여 GIL의 영향을 받지 않을 수 있다.

## 마치며
GIL의 존재를 처음 알았을때는 GIL이 무엇이고, 왜 파이썬의 악명 높은 단점인지 이해가 잘 되지 않았다. 
최근에 운영체제를 공부하고 나서 다시 GIL을 공부하니 문장 하나하나가 더 깊이 이해되는 기분이다.

## reference
- <https://realpython.com/python-gil/>
- <https://www.slideshare.net/kthcorp/h32011c6pythonandcloud-111205023210phpapp02?from_m_app=ios>




