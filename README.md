# Lidemy 學生專用 API Server

這個專案由 [JSON server](https://github.com/typicode/json-server) 改造而成，新增了一些登入相關功能。

除了自訂的一些路由之外，其他的資料都採用 json server 預設的路徑，詳情可以參考 json server 的文件。

此 API server 僅供測試使用，資料隨時都有可能清掉。此 API 一共有三個不同的資料：

1. comments
2. posts
3. users

底下會分開來講解。

Base URL: https://student-json-api.lidemy.me

## Comments

這個 API 底下的資料不需要登入即可進行 CRUD，拿來做簡易留言板用的。

資料結構：

``` json
{
  "id": 1,
  "nickname": "Huli",
  "body": "Welcome to the message board!",
  "createdAt": 1604125999135
}
```

id 跟 createdAt 都會在 server 自動加上，你只要記得傳入 nickname 跟 body 即可。

範例：

``` js
fetch('https://student-json-api.lidemy.me/comments', {
  method: 'POST',
  headers: {
    'content-type': 'application/json'
  },
  body: JSON.stringify({
    nickname: 'hello',
    body: 'comment content'
  })
})
.then(res => res.json())
.then(data => console.log(data))
```

更改資料的時候可以用 PUT 或是 PATCH，前者會把資料整個蓋過去，後者只會更改你有傳的資料。

拿資料的時候可以搭配不同的 query string，例如說按照時間倒序排列：https://student-json-api.lidemy.me/comments?_sort=createdAt&_order=desc

相關參數請參考開頭提供的 json server 文件。

## Users

這邊自己新增了三個 API endpoint，以下會一一說明。

### 註冊

URL: `/register`

用來註冊使用者的，需要傳入 username, password 以及 nickname，即可在系統內註冊一個使用者。

註冊之後會拿到一個 token，是驗證身份用的，這個之後會再提到。

另外，因為這個系統的密碼是用明文儲存，所以會統一在後端把密碼改成 `Lidemy`，因此每個 user 的密碼都會一樣。

範例：

``` js
fetch('https://student-json-api.lidemy.me/register', {
  method: 'POST',
  headers: {
    'content-type': 'application/json'
  },
  body: JSON.stringify({
    nickname: 'hello',
    username: 'hey',
    password: '1234'
  })
})
.then(res => res.json())
.then(data => console.log(data))
```

response:

``` js
{
  ok: 1,
  token: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImhleSIsInVzZXJJZCI6IjAwNmIwNjkwYTgyY2YiLCJpYXQiOjE2MDQxMzI4MTZ9.dfJ4z8DIASsPEytsHE3zA1i2MgNCb2zMLogfqq5ugWU"
}
```

### 登入

URL: `/login`

傳入 username 以及 password 即可登入，登入成功以後一樣會拿到上面註冊成功的 response。

### 身份驗證

URL: `/me`

這個 server 會利用 HTTP Request header 中的 authorization 欄位進行驗證，假設你拿到的 token 是 `1234`，那 authorization 就必須帶：`Bearer 1234`

Server 在需要驗證身份的時候會去檢查這個 header，並把 JWT token 拿出來做比對，如果你有成功攜帶正確的 header，打這一個 API 的時候會回覆正確的使用者資料

範例：

``` js
fetch('https://student-json-api.lidemy.me/me', {
  headers: {
    'authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImhleSIsInVzZXJJZCI6IjAwNmIwNjkwYTgyY2YiLCJpYXQiOjE2MDQxMzI4MTZ9.dfJ4z8DIASsPEytsHE3zA1i2MgNCb2zMLogfqq5ugWU'
  }
})
.then(res => res.json())
.then(data => console.log(data))
```

response:

``` js
  "ok": 1,
  "data": {
    "id": "006b0690a82cf",
    "username": "hey",
    "nickname": "hello",
    "password": "Lidemy"
  }
}
```

## Posts

URL: `/posts`

這是文章的功能，需要帶 title 跟 body 兩個參數，除此之外需要登入才能新增文章，因為文章會跟 user 關聯。所以新增以及編輯文章時請傳入正確的 authorization header，才能正確新增。

在拿文章的時候也請利用 json server 提供的功能，例如說我只想抓取某個使用者的文章，我可以這樣：

https://student-json-api.lidemy.me/posts?userId=1

還可以利用 `_expand` 一併把每篇文章的 user 抓出來：https://student-json-api.lidemy.me/posts?userId=1&_expand=user

相同的，也可以反過來做，抓取某個 user，然後把他底下的文章一起抓出來：https://student-json-api.lidemy.me/users/1?_embed=posts

相關用法請參考 json server 官方文件


