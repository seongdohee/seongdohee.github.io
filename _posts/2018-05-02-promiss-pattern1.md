---
layout: post
title:  "Promise 패턴1 - 자바스크립트와 비동기"
subtitle: "자바스크립트 비동기 처리와 콜수"
date:   2018-05-02 00:00:00 -0400
background: '/img/posts/callback_hell.jpeg'
---

`Promise`에 대해 설명하기 전에 먼저 자바스크립트의 비동기 처리에 대해 이해해야 이것을 왜 사용하는지, 어떻게 사용해야 하는지 감이 잡힐것이다.  
<br/>
자바스크립트는 싱글쓰레드 기반으로, 한번에 한가지 일만 처리할 수 있다. 즉, 모든 프로세스가 동기 처리된다. 하지만 자바스크립트가 사용된 어플리케이션들을 살펴보면, 비동기로 처리되는 일들을 종종 볼 수 있다. 대표적으로 `Ajax`를 들 수 있다. 만약 `Ajax`가 동기 처리되었다면, 네트워크나 서버환경에 의해 응답이 지연되었을 때 프로그램은 아무것도 할 수 없는 블로킹 상태가 될 것이다. 이러한 단점들로 인해 자바스크립트가 실행되는 브라우저나 Node.js 같은 환경에서는 이벤트루프를 통해 동시성을 지원한다. *(이와 같은 자바스크립트 동작 원리에 대해서는 따로 포스팅할 예정)*

자바스크립트에서 비동기 처리되는 경우를 먼저 살펴보자.


### setTimeout
지정 시간 후 어떠한 일을 진행시키고자 할때 `setTimeout`을 사용한다. 

```js

    function task1(){
        setTimeout(function(){
            console.log('task1');
        }, 0);
    }
    
    function task2(){
        console.log('task2');
    }
    
    function task3(){
        console.log('task3');
    }
    
    task1();
    task2();
    task3();

```
위의 코드를 살펴보면, 지연 시간이 0이기 때문에 콘솔에 task1 -> task2 -> task3 순서로 찍힐 것이라고 생각할 수 있지만, 예상대로 동작하지 않는다.


### Ajax

서버와 통신하여 필요한 데이터를 가져올 때, 일반적으로 `Ajax`를 사용한다. 서론에서 잠깐 언급하였듯이 비동기로 동작한다.
```js

    var data;

    $.get('/test', function (response) {
            data = response;
    });
    
    console.log(data); // undefined

```

자바스크립트가 `Ajax` 통신부를 만나면 서버와 통신을 시작하고, 그외에 코드를 실행한다. `console.log(data);` 코드를 해석할 때에는 아직 통신이 완료되지 않아 data에 아무런 값도 할당되지 않았기 때문에 `undefined`를 반환한다.
<br/>

<h1 style="text-align: center;">. . .</h1>

<br/>
  위 두가지 사례와 같이, 비동기로 동작하는 일들을 고려하지 않고 개발한다면 심각한 문제가 발생할 수 있다. 또한 프로젝트를 진행하다보면, 여러 비동기 프로세스를 순차적으로 처리해야하는 요구사항을 빈번하게 마주하게 된다. 이럴때 어떻게 해결해야 할까?

### 콜백 함수

가장 단순하고 쉽게 콜백 함수를 이용하여 사용할 수 있다. 이름 그대로 콜백 함수는 해당 프로세스가 실행 완료 된 시점에 호출된다. 예를 들어, 아래와 같은 요구사항이 있다.
> 어느 프로그램에서, 고객의 이름을 성과 이름을 각각의 데이터로 따로 관리한다고 가정하자.
1. "/getFirstName" url을 통해 통신하여 성(first name)을 얻어올 수 있다.
2. "/getLastName" url을 통해 통신하여 이름(last name)을 얻어올 수 있다.
3. 성과 이름을 합하여 fullname을 콘솔에 반환한다.

```js

    $.get('/getFirstName', function callback1(firstName) {
        $.get('/getLastName', function callback2(lastName) {
            console.log(firstName + lastName);
        });
    });

```

위와 같이 각각의 콜백 함수에서 응답 값을 받아, 고객의 이름을 콘솔에 표현하였다. `/getFirstName` 통신이 완료 된 후 `callback1` 콜백 함수가 실행되므로, first name 값을 확실히 얻어온 후 이후 `/getLastName` 통신이 실행되어 `callback2` 콜백함수에서 last name 값을 얻어 비로소 원하는 값을 도출할 수 있다. 

하지만 이 코드엔 문제점이 존재한다. 여기서는 두가지의 프로세스만 동기 처리하였다. 이렇게 단순한 요구사항만 있으면 만사 좋겠지만.. 대부분의 경우 더 많고 복잡한 프로세스를 요구한다. 10개의 프로세스를 동기 처리한다고 생각해보자.

```js

    $.get('/task1', function (value1) {
        $.get('/task2', function (value2) {
            $.get('/task3', function (value3) {
                $.get('/task4', function (value4) {
                    $.get('/task5', function (value5) {
                        
                        // ..... 지쳐서 더 못쓰겠음.. 중간 생략
                        $.get('/task10', function (value10) {
                            
                            // do anything
                        
                        });
                        
                    });
                });
            });
        });
    });

```

<div style="text-align: center;">
<img src="http://post.phinf.naver.net/MjAxNzA2MjRfMTM4/MDAxNDk4Mjk3NDY3OTcw.wZPwpV9Ua3LzZXH38Mhbx_WxIr0157zj-5YuHfBqInQg.he_kWQez42qgMo8Meqi9Cfzdd7_hMwEOWLUS_aADRqsg.JPEG/IvLTZ9S6kQPUfFwmTXejiLRmReeI.jpg">
</div>

썸네일처럼 실로 아도겐스러운 상황이 연출된다. 가독성은 물론이거니와 변수명 관리도 어려워진다. (비슷한 속성의 응답값일 경우 위와 같이 숫자를 붙이는 방법밖에는...) 이런 상황을 흔히 "콜백 지옥에 빠졌다"고 표현한다. 다음 포스팅에서는 실제로 실무에서 겪은 콜백 지옥과 Promise 패턴을 통해 콜백 지옥에서 탈출하는 방법에 대해서 서술하겠다.