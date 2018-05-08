---
layout: post
title:  "Promise 패턴2 - Promise란 무엇인가"
subtitle: "콜백지옥에서 벗어나기"
date:   2018-05-03 00:00:00 -0400
background: '/img/posts/callback_hell.jpeg'
---

이전 포스트, 자바스크립트에서 비동기 사례와 콜백함수의 처리 방법, 단점에 대해서 설명하였다. 그렇다면 이 `콜백지옥` 에서 어떻게 벗어날 수 있을까? 다시 현기증나는 코드를 들여다보도록하자.

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


이 코드에서 단 하나의 장점이 있다. 바로 가장 먼저 호출된 콜백 함수로부터 지역 변수를 공유할 수 있다는 것이다. 하지만 앞서 설명하였듯 가독성이 매우 떨어지며, 변수명 관리도 쉽지않다는 치명적인 단점이 있다. 여기서 `Promise`의 필요성이 대두된다.

## Promise

말 그대로 `약속`이다. 비동기 작업에 대한 처리 결과 즉 `약속값`을 반환한다. 이 `약속값`을 통해 개발자는 작업에 대한 종료 시점을 알 수 있고, 그다음 작업을 결정할 수 있다. `Promise`는 다음중 하나의 상태를 가진다.

- 대기중(pending): 초기 상태, 아직 작업이 완료되지 않은 상태이다.
- 이행됨(fulfilled): 작업이 성공적으로 완료된 상태이다.
- 거부됨(rejected): 에러등과 같은 이유로 작업이 실패한 상태이다.

그럼 실제 코드로 살펴보자.

### Promise 생성하기 (대기)
```js

   var promise = new Promise(function(resolve, reject){
        
        // do async action
        
   });
   
```

`Promise`는 `Promise` 생성자를 통해 생성한다. 이때 `Promise`는 비동기 작업을 처리하기 전까지 `대기중` 상태값을 갖는다. 비동기 작업이 완료된 이후에야 비로소 `이행됨` 또는 `거부됨` 상태로 변환하여, 개발자는 이 상태값을 가지고 이후 작업을 결정할 수 있다.

### 작업 이행 처리
```js

   var promise = new Promise(function(resolve, reject){
        setTimeout(function(){
            resolve('success');
        }, 5000);
   });
   
```
`resolve()`를 눈여겨보자. 비동기 작업이 완료 된 이후, `resolve()`를 실행함으로써 이 작업이 성공적으로 완료되었음을 알려 줄 수 있다. 이 함수의 인자값으로 처리 걸과를 넘길 수 있다.

### 작업 거부 처리
```js

   var promise = new Promise(function(resolve, reject){
        setTimeout(function(){
            reject('fail');
        }, 5000);
   });
   
```
`reject()`는 이 작업이 실패로 끝났음을 알려준다. 마찬가지로, 이 함수의 인자로써 처리 결과를 넘길 수 있다.


### 예제

위에서 설명한 상태들을 가지고 간단한 예제를 작성하였다.
```js

    function httpRequest(){
       return promise = new Promise(function(resolve, reject){
          
          $.get('/test1', function(response){
                
              if (response.status !=== 200){
                    resolve('success');
              }
              else {
                    reject(new Error('http request err'));
              }
          
          })
          
       });
    }
    
    httpRequest().then(function(response){
        console.log(response);
    }).catch(function (err){
        console.log(err);
    });
  
   
```
http 상태 코드가 200이면, 작업을 성공 처리하여 서버에서 받아온 응답 값을 콘솔에 출력하고,200이외의 코드이면 작업을 실패 처리하여 에러를 생성하여 콘솔에 출력한다. 하단부의 코드를 보면, `httpRequest()`함수를 통해 `promise`를 생성하고, `then`과 `catch` 메서드를 체이닝 한 것을 확인할 수 있다. 이행 상태의 결과는 `then`으로, 거부 상태의 결과는 `catch`에서 처리된다. 이것으로 더 많은 비동기 작업을 아름답게 처리할 수 있다. 
```js

    function httpRequest(url){
       return promise = new Promise(function(resolve, reject){
          
          $.get(url, function(response){
                
              if (response.status !=== 200){
                    resolve('success');
              }
              else {
                    reject(new Error('http request err'));
              }
          
          })
          
       });
    }
    
    
    httpRequest('/request1').then(function(response){
        return httpRequest('/request2'); 
    }).then(function(response){
        return httpRequest('/request3');  
    }).catch(function (err){
        console.log(err);
    });
  
   
```

위와 같이 체이닝읕 통해 여러 비동기 작업을 순차 처리하도록 하였다. 지금까지 본 `promise`의 흐름은 아래의 도표로 표현될 수 있다.

<div style="text-align: center;">
    <img src="https://mdn.mozillademos.org/files/8633/promises.png">
</div>

### 좋은 예와 나쁜 예

`promise`라는 것을 처음 알게 된 계기는, 실무에서 콜백 지옥을 경험하고 나서부터였다. 요구사항은 다음과 같았다.
> 1. 지정된 디렉토리의 모든 파일을 삭제한다.
2. 정적 리소스(이미지, 소스파일 등)가 압축된 zip 파일을 다운받는다.
3. 다운받은 zip 파일의 압축을 해제한다.
4. zip 파일은 삭제한다.  
(사실 이것보다 많지만 생략..)

Node.js 환경이였기 때문에 각 작업은 비동기로 동작하지만, 순차처리가 되어야 했다. 코드는 이러하였다. (최대한 간소화하여 작성한다.)
```js

    new Promise(function(resolve, reject){
        
         // 각각의 함수는 `Promise` 객체를 리턴한다. 
         removeAllFile().then(function(){
                
            downloadZip().then(function(){
                    
                unzipFile().then(function(){
                            
                      return removeZipFile().then(function(){
                           resolve();
                      }).catch(function(err){
                           reject(err);
                      });  
                    
                }).catch(function(err){
                        reject(err);
                });     
                    
            }).catch(function(err){
                reject(err);
            });         
            
         }).catch(function(err){
            console.log(err);
         });
        
    });
  
   
```
<div style="text-align: center;">
    <img src="data:image/jpeg;base64,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">
</div>

(마크다운으로 작성하다보니 검사기가 없어서 괄호 열고닫는데 헷갈려 죽을뻔했다...) `Promise`를 사용하였지만 여전히 콜백 지옥에서 벗어날수 없었다. 만약에 이 요구사항이 변동되거나 추가된다면?? 그야말로 헬이 아닐 수 없다. (실제로 요구사항이 변동되어 수정하려고 하자 저 코드를 만났다.. 살려조 ㅠㅠ) 그러면 정말 `Promise` 답게 코드를 수정해보자.

```js

    removeAllFile().then(function(){
        
        return downloadZip();
        
    }).then(function(zipPath){
        
        return unzipFile(zipPath);
        
    }).then(function(zipPath){
     
        return removeZipFile(zipPath);
    
    }).catch(function(err){
    
        throw new Error(err);
        
    });
 
   
```
훨씬 깔끔해졌으며 가독성도 높아졌다. 각 작업의 성공 결과 처리값은 리턴값으로 보낼 수 있으며, `then`에서 인자로 받아 처리할 수 있다. 거부된 상태는 `catch`에서 한번에 관리한다. 

<h1 style="text-align: center;">. . .</h1>

이렇게 2차례의 포스팅으로 `Promise`에 대해 알아보았다. 작성한 포스팅은 단편적인 면만 설명하고 있으므로 좀더 자세한 내용은 아래 링크를 통해 참고하길 바란다.

- [MDN - Promise](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [eBook - You Don't Know JS 비동기와 성능](http://www.hanbit.co.kr/store/books/look.php?p_의code=E7131315570)