---
title: nodejs blah blah
thumbnail: https://cdn-images-1.medium.com/max/2000/1*q9ww_u32hhpMaA-Q_s1ujw.png
categories:
- Web
- Nodejs
tags:
- nodejs
- web
---

### Filestream Module

* **File I/O**
  * **readFile**
    ``` javascript
    var fs = require('fs');
    fs.readFile('sample.txt', 'utf8', function(err, data){
      // data 이용한 코드
    });
    ```
    파일명은 경로명을 포함하여 적고, 해당 파일로부터 utf-8 형식으로 data를 읽어온다.

  * **writeFile**
    ``` javascript
    fs.writeFile(`data/${title}`, data, 'utf8', function(err){
      // blah blah
    });
    ```
    다음과 같이 data를 저장하는 파일을 생성할 수 있다.


* **Readdir**
  ``` javascript
  var testFolder = '../data';
  var fs = require('fs');

  fs.readdir(testFolder, function(err, filelist){
    console.log(filelist);
  });
  ```
  해당 directory에 있는 file들의 리스트를 찾아준다. `filelist`를 글 목록 등으로 사용 가능하다.

### Expression Language
``blah blah ${표현식} blah blah...`` 형태로 사용된다.
표현식으로는 **변수**, **객체의 속성**, **배열의 원소** 등이 사용된다.

### Url
`url.parse(_url, true)` 에는 URL 상의 정보가 다음과 같이 저장되어 있다.
```
 {
  protocol: null,
  slashes: null,
  auth: null,
  host: null,
  port: null,
  hostname: null,
  hash: null,
  search: '?id=introduce',
  query: { id: 'introduce' },
  pathname: '/',
  path: '/?id=introduce',
  href: '/?id=introduce'
}

```

다음과 같이 이 객체의 각 속성에 접근하여 유용하게 사용 가능하다.
``` javascript
var app = http.createServer(function(request,response){
var _url = request.url;
var queryData = url.parse(_url, true).query;
var pathname = url.parse(_url, true).pathname;
```



### Callback function
``` javascript
fs.readFile(`data/${queryData.id}`, 'utf8', function(err, description){
  // here, code for callback function
});
```
예시를 통해 **callback 함수** 를 쉽게 이해할 수 있다.
위 예시의 경우 해당파일을 읽은 후, `readFile`의 3번째 인자로 전달된 **callback 함수** 가 실행된다.


### pm2
**npm** 을 이용하여 **pm2** 를 설치한다.
`pm2 start main.js --watch` : `main.js` 파일의 수정 사항을 자동으로 반영하며 실행
`pm2 list` : 현재 `pm2`로 실행되고 있는 file list
`pm2 monit`
`pm2 log` : 현재 `pm2`로 실행되고 있는 file들의 상태; error 체크도 가능

### Synchronize & Asynchronize
'sample.txt'에는 'B'라는 내용이 저장되어있다.
* **Synchronized code**
  ``` javascript
  console.log('A');//1
  var result = fs.readFileSync('./sample.txt', 'utf8');//2
  console.log(result);
  console.log('C');//3
  ```
  위 코드를 실행시키면 A B C 순으로 출력된다. `result`에 파일을 읽어온 후에 이후의 코드가 실행되기 때문이다.
  하지만, 파일을 읽어오는 것은 상대적으로 긴 실행시간을 가지므로 효율성을 위해 **비동기적** 으로 실행시켜야한다

* **Asychronized code**
  ``` javascript
  console.log('A');//1
  fs.readFile('./sample.txt', 'utf8', function(err, result){
    console.log(result);//3
  });
  console.log('C');//2
  ```
  위 코드를 실행시키면 A C B 순으로 출력된다. 파일을 읽어오는동안 C를 출력하고, 파일을 다 읽어온 후에 실행되는 callback 함수에서 B가 출력된다.


### Create
**Form tag** 를 이용하여 게시글을 작성한다.

``` html
// file: form.html
<form action="http://localhost:3000/process_create" method="post">
  <p><input type="text" name="title"></p>
  <p>
    <textarea name="description"></textarea>
  </p>
  <p>
    <input type="submit">
  </p>
</form>
```
**form** 안의 각 content에 사용자가 넣은 정보를 submit 눌렀을때, **Query String** 의 형태로 **action** 에서 가리키는 서버애 데이터를 전송하는 html 의 기능이다.
`method` 를 통해 전송방식을 설정할 수 있다. Default로는 `method="get"`가 설정되고, 이 떄는 url을 통해 전달되는 데이터가 모두 표시된다.
따라서, 단순한 읽기가 아닌 생성이나 수정가능한 경우에는 눈에 보이지 않는 방법으로 보내야한다. 이때는 `method="post"`를 사용하면 되고, 긴 데이터를 전송할 때도 사용된다.


### post 방식으로 전송된 데이터
눈에 보이지 않는 **post 방식** 으로 보내진 데이터는 어떻게 가져올 수 있을까.
``` javascript
var qs = require('querystring');
var app = http.createServer(function(request,response){
  // request, response는 createServer의 callback 함수의 인자
  var body = '';
  request.on('data', function(data){
    body = body+data;
  }); // data를 수신할 때마다 해당 callback 함수가 실행된다.
  request.on('end', function(){
    var post = qs.parse(body);
    var title = post.title;
    var description = post.description;
    fs.writeFile(`data/${title}`, description, 'utf8', function(err){
    response.writeHead(302, {Location: `/?id=${title}`}); // redirection
    response.end();
  });// 파일 생성에 성공하면 해당 callback함수가 실행된다.
  }); // 정보수신이 끝났음을 의미
});
```

`qs.parse(body)`를 이용하여 post 방식으로 보내진 데이터를 다음고 같이 가져올 수 있다.
`0|main     | { title: 'nodejs ', description: 'nj is ...' }`
이 객체의 각 속성에 접근하여 데이터를 사용하면 된다.

여기서 **form** 에 작성한 데이터는 게시글이다. `fs.writeFile`을 통해 입력된 data를 저장하는 파일을 생성한다.

`fs.writeFile`을 통해 성공적으로 파일이 성공적으로 생성되면, 해당 callback 함수에서 **리다이랙션** 이 실행된다. 해당 `Location`으로 이동되는 것이다.


### Update  

작성된 게시글을 수정한다.
`<a href="/update?id=${title}">update</a>`
**update 링크** 를 클릭하면 해당 링크로 이동한다.

이 링크를 클릭하면 글수정이 실행된다.
``` javascript
else if(pathname==='/update'){
  fs.readdir('./data', function(error, filelist){
    fs.readFile(`data/${queryData.id}`, 'utf8', function(err, description){
      var title = queryData.id;
      var list = templateList(filelist);
      var template = templateHTML(title, list, `
        <form action="/update_process" method="post">
          <input type="hidden" name="id" value="${title}">
          <p><input type="text" name="title" placeholder="title" value="${title}"></p>
          <p>
            <textarea name="description" placeholder="description">${description}</textarea>
          </p>
          <p>
            <input type="submit">
          </p>
        </form>`, `<a href="/create">create</a> <a href="/update?id=${title}">update</a>`);
      response.writeHead(200);
      response.end(template);
    });
  });
}
```
각 칸에 기존 글의 제목과 내용이 들어가 있도록 value를 설정한다.
이 때 제목도 수정됐을 때에도 어느파일을 수정할 지 알 수 있도록 하기위해, **원제목** 을 **id** 라는 hidden 칸에 저장하여 **post** 방식으로 /update_process로 전달한다.

``` javascript
else if(pathname === '/update_process'){
  var body = '';
  request.on('data', function(data){
    body = body+data;
  }); // 수신할 때 마다 해당 콜백함수를 호출하게 되어있다.
  request.on('end', function(){
    var post = qs.parse(body);
    var id = post.id;
    var title = post.title;
    var description = post.description;
    fs.rename(`data/${id}`, `data/${title}`, function(error){
      fs.writeFile(`data/${title}`, description, 'utf8', function(err){
        response.writeHead(302, {Location: `/?id=${title}`});
        response.end();
      });// 파일 생성 성공시
    })// 파일명 수정
  }); // 정보수신이 끝났음을 의미
}
```
원제목을 통해 수정할 파일을 찾아서 사용자가 입력한대로 수정한다.

### Delete

작성된 게시글을 삭제한다. 데이터의 생성, 수정 등 변형을 실행하는 버튼은 링크로 생성하지 않도록 주의한다.
``` javascript
<form action="delete_process" method="post">
 <input type="hidden" name="id" value="${title}">
 <input type="submit" value="delete">
</form>
```
해당 form 을 홈화면에 추가한다.

``` javascript
else if(pathname === '/delete_process'){
  var body = '';
  request.on('data', function(data){
    body = body+data;
  }); // 수신할 때 마다 해당 콜백함수를 호출하게 되어있다.
  request.on('end', function(){
    var post = qs.parse(body);
    var id = post.id;
    fs.unlink(`data/${id}`, function(error){
      response.writeHead(302, {Location: `/`});
      response.end();
    })
  }); // 정보수신이 끝났음을 의미
}
```
id라는 제목의 게시글을 위 코드와 같이 삭제한다.

### 객체

객체와 배열은 다음과 같이 다르게 정의한다.
``` javascript
// array
var members = ['egoing', 'k3', 'sungju'];

// object
var roles = {
  'programmer' : 'egoing',
  'designer' : 'k3',
  'manager' : 'sungju'
  f1 : function(){
    console.log(this.designer);
  }
}
roles.f1(); // 객체 안의 함수 호출
```
객체는 각 성분에 대한 key값과 이에 상응하는 value를 가진다. `object.key`나 `object[key]`로 value를 사용할 수 있다.
객체는 데이터와 처리방법을 담는 그릇이다. 객체의 value로 단순한 data 뿐 아니라 함수도 성분으로 가질 수 있다. 함수는 다른 statement와 달리 값을 가져서 변수에 함수를 저장할 수 있다. `var a = function(){...}` * `this` : 객체 안에서 자신의 객체를 참조하는 키워드

``` javascript
console.log(roles.designer);

for(var name in object){
  console.log('key: ',name, ' value: ', object[name]);
}
```
다음과 같은 반복문에서 `name`은 **key** 값에 해당하고, **value** 에는 `object[name]`과 같이 접근할 수 있다.

* **refactoring**
같은 동작을 하는 코드를 함수나 객체화, 배열화 하는 과정을 통해 유지보수가 쉬운 코드로 개선하는 것

### Module
위에서 살펴본 **refactoring** 방법으로 **모듈화** 가 있다.
``` javascript
var M = {
  v: 'v',
  f: function(){
    console.log(this.v);
  }
}
module.exports = M;
```
다음과 같이 M이 가진 기능을 module로 내보내면 다음 코드와 같이 해당 파일 밖에서도 사용가능하다.
`var part = require('./mpart.js');`

`template`처럼 복잡하고 긴 코드를 `lib`과 같은 폴더에서 모듈화시키면 훨씬 좋은 코드를 작성할 수 있다.

### 보안
보안이 필요한 경우를 크게 두 가지로 살펴본다.
1. 오염된 정보의 입력
  간단한 예시로 사용자가 `../` 등의 형태의 입력을 통해 상위폴더를 탐색할 수 있을 수도 있다. 따라서 입력된 경로정보를 안전하게 세탁하여 사용하는 것이 좋다.
  ``` javascript
  var path = require('path');
  ...
  var filteredId = path.parse(queryData.id).base;
  fs.readFile(`data/${filteredId}`, 'utf8', function(err, description){
    ...
  }
  ```
2. 오염된 정보의 출력
  간단한 예시로 html 용어가 그대로 출력되지 않고, html 언어에 따라 실행되는 것을 방지한다.

  먼저 다음과 같이 `sanitize-html` npm을 설치한다.
  ``` javascript
  npm init
  npm install -S sanitize-html
  ```
  그 후 필요한 입력 부분을 **sanitize** 시켜서 사용하면 된다.
  ``` javascript
  var sanitizeHtml = require('sanitize-html');
  ...
  var sanitizedTitle = sanitizeHtml(title);
  var html = template.html(title, list,
    `<h2>${sanitizedTitle}</h2>${sanitizedDescription}`,
    ...
  ```

### More to Learn
* **Webbrowser**
* **Database**
  * mongodb
  * mysql
* **Framework** : 공통적인 실행을 하는 부분을 미리 구현해 놓은 것으로  사용을 위해 많은 공부가 필요하지만 아주 편리하다.
* **Module/ Api**
 * Awesome 북마크 : node.js awesome 같은 검색어로 좋은 모듈을 찾을 수 있다.
