---
layout: post
title:  "Promise 패턴3 - 응용편"
subtitle: "Promise 패턴 고급 응용"
date:   2018-05-06 00:00:00 -0400
background: '/img/posts/callback_hell.jpeg'
---

이전 포스팅에서는 `promise`에 대한 기본 개념과 사용방법에 대해서만 서술하였다. 하지만 실제로 실무에서 사용해보니, 기본 사용방법만 가지고는 터무니없이 부족하다는 느낌이 들어 몇가지 없지만 보충해본다.

## 1. 반복적인 비동기 작업에 대한 처리

반복적으로 발생하는 비동기 작업에 대한 처리를 하기위해서 `promise`를 어떻게 활용해야 할까? 단순하게 생각하여 각 작업에 대해서 `then` 메서드를 통해 연결해주면 되겠지만, 이 반복적인 횟수가 정해지지 않았다면? 아래의 예제를 살펴보자.

```js

    // 반복되는 작업의 배열
    var tasks = [1,2,3,4,5]; 

    var allTasks = tasks.reduce((p, task) => {
        
        return p.then(() => {
            
            console.log(`task ${task} is completed!`);
            
        });
        
    }, Promise.resolve());

    allTasks.then(() => {
    
        console.log('all task is completed!');
    
    }).catch(err => {
        
        console.log(new Error(err));

    });

```

이 예제를 이해하기 위해서는 먼저 배열 API인 `reduce`를 알아야한다. `reduce`는 두가지의 인자를 받는데, 첫번째는 각 배역 요소에 대한 콜백함수이고, 두번째 인자는 콜백함수의 첫번째 인수로 사용하는 값이다. 더 자세한 내용은 [이곳](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)을 참고하자.  
<br/>
이 예제에서 `tasks` 배열은 반복적인 작업에 대한 배열이 된다. 코드상에서는 1~5로 설정하였지만, 이 배열은 얼마든지 동적으로 활용하면 된다.
다음으로, `reduce`의 두번째 인자를 잘 살펴보면, `Promise.resolve()`를 확인할 수 있다. 이 코드는 아래와도 같이 작성될 수 있다.

```js

    new Promise((resolve, reject) => resolve());

```
즉, 상태가 `이행된` `promise` 객체를 반환한다. 이 `promise` 객체를 가지고 `then` 메서드 내부에서 하고자 하는 작업을 실행하면된다. 최종적으로 이 `reduce`는 모든 작업에 대한 처리 결과를 반환하며, 이 작업 역시 `then`이나 `catch`를 통해 적절하게 처리하면된다.   
<br/>
그런데 여기서 헷갈리는 것이 하나 있다. 저번 포스트에서 링크로만 대체 했지만, `Promise`에는 `all`이라는 메서드가 있다. 이것으로는 처리할 수 없을까?

## 2. Promise.all

결론부터 말하자면, 위과 같이 반복적인 비동기 작업을 순차 처리 해야 하는 상황(1번 task 종료 후 2번 task가 실행되어야 하는 상황)이라면 `Promise.all`을 사용하면 안된다. `Promise.all`은 각 작업에 대한 순서를 보장해 주지 않는다. 아래의 예제를 살펴보자.

```js

    var task1 = new Promise((resolve, reject) => {
         setTimeout(()=>{
            console.log('task1');
            resolve('task1');
         }, 3000);
    })
    var task2 = new Promise((resolve, reject) => {
        setTimeout(()=>{
            console.log('task2');
            resolve('task2');
        }, 2000);
    })
    var task3 = new Promise((resolve, reject) => {
        setTimeout(()=>{
            console.log('task3');
            resolve('task3');
        }, 1000);
    })
    
    Promise.all([p1, p2, p3]).then(values => console.log(values));
    
    // console 결과
    // task3
    // task2
    // task1
    // ['task1', 'task2', 'task3']

```

이해를 돕기 위해 각 task에 대한 시간을 1초 간격으로 조절하였다. 배열 순서대로 작업이 순차 실행될 것 같지만, 각 작업은 비동기 처리되고 이 작업들에 대한 처리 결과만 종합한 `Promise`객체를 반환한다. 
따라서 1번 예제에서와 같이 각 작업이 순차처리되어야 하는 상황에서는 사용할 수 없고 여러 비동기 작업들에 대한 완료 시점과 결과를 알고 싶을 때 `Promise.all` 을 사용해야 한다.  
<br/>
`Promise.all`은 작업중 하나라도 거부(reject)되는 경우 이후 작업 처리 결과와 상관없이 즉시 거부(reject) 상태로 반환한다.

```js

    var task1 = new Promise((resolve, reject) => {
         setTimeout(()=>{
            console.log('task1');
            resolve('task1');
         }, 3000);
    })
    var task2 = new Promise((resolve, reject) => {
        reject('reject');
    })
    var task3 = new Promise((resolve, reject) => {
        setTimeout(()=>{
            console.log('task3');
            resolve('task3');
        }, 1000);
    })
    
    
    Promise.all([p1, p2, p3]).then(values => {
        console.log(values);
    }).catch(err => {
        console.log(err);
    });
    
    // console 결과
    // reject;


```

위와 같이, task2에서 거부되는 경우, task1과, task2는 이행되었음에도 최종적으로 거부되어 `catch`에서 처리되는 것을 볼 수 있다. 즉, 모든 결과를 보지 않고 빠르게 거부 처리한다. 

<h1 style="text-align: center;">. . .</h1>

쓰다보니 두가지 밖에 안나왔지만 실제로 햇갈렸던 부분이라 따로 정리해보았다. 좀더 자세한 내용은 아래 링크를 통해 참고하길 바란다.
                                        
- [MDN - Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [eBook - You Don't Know JS 비동기와 성능](http://www.hanbit.co.kr/store/books/look.php?p_code=E7131315570)