---
title: 'nestjs/passport 에 대해서'
date: '2023-05-15'
tags: ['passport', 'nestjs/passport', 'nestjs']
draft: false
summary: 'nestjs/passport 에 대해서'
---

# nestjs/passport

`passport` 는 `Node` 진형에서 인증 및 인가를 `OAuth` 를 통해 쉽게 구현할 수 있도록 제공하는 `library` 이다.
기본적은 `express` 를 사용하여 처리한다면, `passport` 의 `Docs` 의 내용대로 구현하면 되겠지만, `Nestjs` 를 사용하여 구현한다면,
변형한 `nestjs/passport` 를 사용해야만 한다.
`passport` 를 한번더 `wrapping` 한 느낌이라 `nestjs/passport` 어떻게 구현되고, 사용되는지 굉장히 추상적으로 느껴진다.
그래서 `nestjs/passport` 의 `github` 를 보고 어떠한 방식으로 구현되어 있는지 연구해보고자 한다.

일단 `passport` 부터 어떠한 방식으로 구현되는지 살펴본다.

## passport 

`passport` 는 인증 및 인가를 쉽게 만들어주는 `nodejs` 를 위한 `middleware` 이다.
`middleware` 로 이루어져 있어, 여러 `web frameworks` 에 쉽게 사용할 수 있도록 만들어준다.
`express` 를 사용해보았다면, 이러한 `middleware` 에 대한 개념을 쉽게 이해할 수 있다

기본적으로 `passport` 는 총 3가지의 컨셉을 가진다

1. Middleware
2. Strategies
3. Sessions

### passport middleware 

```ts
app.post('/login/password',
  passport.authenticate('local', { failureRedirect: '/login', failureMessage: true }),
  function(req, res) {
    res.redirect('/~' + req.user.username);
  });
```

이 코드를 보면 `/login/password` 라우터를 `post` 로 받으면, 
`passort authenticate` 를 `middleware` 로 실행되고, 그다음 `post` 에서 `req, res` 를 받아서 응답을 해준다.

이렇게, `passport authenticate` 는 `middleware` 로써, 인증된 `user` 인지 아닌지를 평가한후, 인증된 `user` 라면 `default` 로 `req` 에 `user property` 를 생성한다. 
여기서 `user property` 는 인증된 `user` 의 정보를 담고있다.

`passport authenticate` 의 첫번째 인자는 사용한 `Strategies` 이름이고, 2번재 인자는 설정할 설정값이다.
여기서는 인증이 실패했을 경우, 실패 메시지를 같이 포함하여 보내고, `/login` 으로 `redirect` 하는 설정을 넣었다.

이 요청인증을 위한 `mechanism` 은 `Strategy` 에 의해 구현된다.
각 `Strategy` 에 따라 인증 처리 옵션이 다를 수 있다. 

일단, 위의 `local Strategy` 는 검증하는데 `username` 과 `password` 를 사용한다. 
(`OpenId Connect` 를 통한 인증은 전혀 다른 방식으로 처리될 수 있다.)

### Strategies

`Strategy` 는 인증요청에 대한 책임을 가진다.
이 `Strategy` 에서 사용되는 인증 매커니즘은 `identity provider(idP)` 를 통해 해당 `id` 가 존재하는지, `password` 가 맞는지 확인한다.
`JWT` 같은경우는 `JWT encoding` 을 통해 `idP` 를 확인하고, `id` 를 조회한후, `password` 를 확인한다음 자격증명을 확인한다.

이렇게 자격증명이 성공적으로 검증된다면, 요청은 인증된것이다.
이러한 `Strategy` 는 종류에 따로 구분되므로, 해당 `Strategy` 를 사용하기 위해서는 그에 따른 `Strategy` 의 `package` 를 받아야한다. 

이렇게 처리한다면, 불필요한 종속성없이 필요한 기능만 `package` 단위로 받아서 사용하므로, 더 효율적이다.

```sh

$ npm install passport-local

```

이렇게 받은 `passport-local` 를 사용하여 설정은 다음처럼 작업한다.

```ts
var LocalStrategy = require('passport-local');

var strategy = new LocalStrategy(function verify(username, password, cb) {
  db.get('SELECT * FROM users WHERE username = ?', [ username ], function(err, user) {
    if (err) { return cb(err); }
    if (!user) { return cb(null, false, { message: 'Incorrect username or password.' }); }

    crypto.pbkdf2(password, user.salt, 310000, 32, 'sha256', function(err, hashedPassword) {
      if (err) { return cb(err); }
      if (!crypto.timingSafeEqual(user.hashed_password, hashedPassword)) {
        return cb(null, false, { message: 'Incorrect username or password.' });
      }
      return cb(null, user);
    });
  });
});
```

여기서는 `LocalStrategy` 로 `passort-local` 을 가져온후 할당한다.
그리고 `LocalStrategy` 를 `new` 연산자를 통해 `strategy` 객체를 만드는것을 볼수 있다.

기본적으로 `LocalStrategy Constructor` 는 함수 매개변수를 갖는다. 
이 함수 매개변수가 `verify` 함수로써 지정될 매개변수이다.

> `Docs` 에서는 많은 `Strategies` 들이 이러한 패턴으로 이루어져 있음을 말하고 있다.

인증 요청을 받을때, 이 `Strategy` 는 `request` 에 포함된 자격증명을 `parse` 한다.
그런다음에, 자격증명에 맞는지 확인한다.

위의 로직을 보면 `username` 과 `passowrd` 를 받아 처리하는 로직을 볼 수 있다.

> 
```ts

  db.get('SELECT * FROM users WHERE username = ?', [ username ], function(err, user) { ... }
```

위 로직을 보면 `SQL` 을 통해, `username` 이 있는지 확인하는 로직이다.
이 역시 `callback` 함수를 통해 비동기로 이루어지는것을 볼 수 있다.

이 `callback` 함수는 `err` 와 `user` 인자를 받는데, `username` 이 존재한다면, `user` 에 값이 있을것이고 없다면 `err` 에 오류 관련 `message` 를 가질 것이다.

> `callback` 함수 안에서 사용되는 검증로직

```ts
    if (err) { return cb(err); }
    if (!user) { return cb(null, false, { message: 'Incorrect username or password.' }); }

    crypto.pbkdf2(password, user.salt, 310000, 32, 'sha256', function(err, hashedPassword) {
      if (err) { return cb(err); }
      if (!crypto.timingSafeEqual(user.hashed_password, hashedPassword)) {
        return cb(null, false, { message: 'Incorrect username or password.' });
      }
      return cb(null, user);
    });
```

위의 로직을 보면 `cb` 에 값을 전달하는 형태로 이루어진것을 볼 수 있다.
이 `cb` 는 `verify` 에서 사용되는 매개변수중 3번째 인자인 `callback` 함수이다.

`cb` 의 첫번째 값은 `error` 처리하는 인자이며, 없다면 `null` 값을 넣어준다.

두번째 값은 유효하지 않은 사용자인 경우 `false` 를 넣으면 인증 실패를 알려주며, 유효한 사용자가 있는 경우 `user` 값을 넘기면, 이후 `user` 에 대한 값이 `req.user` 에 담기게 된다.

세번째 값은 두번째 값이 `false` 일 경우 인증실패를 알리며, 실패 `message` 를 같이 전달하는 역할을 한다.

이때, `cb` 의 `error` 를 사용한 것과 두번째 인자에 `false` 를 전달한것의 차이점이 뭔가 이해가 안갈수 있는데, 첫번째 인자에 `error` 를 전달하면, 서버가 비정상적으로 작동하는 경우의 내부 오류를 나타낼 수 있으며, 2번째 인자에 `false` 전달은 인증 실패 및 유효하지 않은 자격증명을 전달하는것이다. 이경우 서버는 정상적으로 작동한다.

#### Register 

이렇게 만든 `Strategy` 를 `passport` 에서 사용할 수 있도록 등록해주는 과정이 있어야 한다.

```ts

const passport = require('passport')
passport.use(strategy)

```
등록된 `Strategy` 를 사용하기 위해, `authenticate` 에서 불러오는데 다음의 `convention` 을 따른다

모든 `Strategies` 의 이름은 `package-{name}` 패턴에 따라 대응된다.
그러므로 `LocalStrategy` 는 `passort-local` 이므로, `local` 로 사용할 수 있다.

```ts
app.post('/login/password',
  passport.authenticate('local', { failureRedirect: '/login', failureMessage: true }),
  function(req, res) {
    res.redirect('/~' + req.user.username);
  });
```

위에서 `authenticate` 로 `local strategy` 를 사용하기 위해, `strategy` 이름을 `local` 로 지정한것을 볼 수 있다.

이렇게 하면 이제 `authenticate` 에서 `LocalStrategy` 를 사용할수 있도록 `passport` 에 등록되었고, `authenticate` 에서 사용한 것이다.

만약, `strategy` 이름이 충돌되어, 변경해야 한다면 다음처럼 설정도 가능하다고 `Docs` 에서 설명한다.

```ts

var passport = require('passport');

passport.use('password', strategy);

```

이제 해당 `strategy` 의 이름은 `password` 에 `password` 로 등록 된다.

```ts
app.post('/login/password',
  passport.authenticate('password', { failureRedirect: '/login', failureMessage: true }),
  function(req, res) {
    res.redirect('/~' + req.user.username);
  });
```

이러한 방식을 사용하여, `strategy` 의 변경된 이름을 `authenticate` 에서 사용하는것을 볼 수 있다.

### Sessions

`page` 에서 `page` 로 탐색할때, 사용자 식별을 위한 능력이 필요하다.
각각의 동일한 사용자와 관련된 일련의 요청 및 응답을 세션이라고 부른다.

`HTTP` 는 `Stateless protocol` 이라고 부른다.  
이말은 상태가 없다는것을 의미하며, 각 요청마다 전혀 다른 `context` 를 형성하므로, 매번 요청이 다른것으로 이해된다는 것이다.

그러므로, `user` 의 정보를 받기 위해서는 매번 `idP`를 확인하고, 검증한다음 `user` 정보를 받아서 처리해야 함을 뜻한다.

이는 매우 비효율적인 방식이다.
`session` 은 `client` 와 `server` 간의 상태를 유지하기 위해 만들어졌다.

`HTTP` 는 상태를 가질 수 없으므로, 이를 대체하기 위해 `HTTP-cookie` 를 제공한다.
만약, `server` 를 통해 로그인요청이 이루어진다고 가정하자. 

`HTTP cookie` 는 `server` 에서 설정하여 `client` 로 보내면 이후, `client` 의 매 요청마다 `cookie` 값을 같이 포함하여 요청하게 된다.

이런경우, `server` 에서 `user` 의 정보를 `client` 로 `cookie` 로 보내면, 쉽게 인증된 유저인지 확인이 가능할 것이다.

하지만 `user` 의 정보를 `cookie` 를 통해 직접적으로 보여주는것은 중간에 정보가 탈취당할 우려가 있으며, 이를 통해 엄청난 보안 사고가 발생할 수 있다.

이러한 부분들을 전부 해결해주는 방식이 바로 `Session` 이다.
`session` 은 `cookie` 에 직접적인 `user` 정보를 보내는것이 아닌, `session ID` 를 전달해 준다.

> `session` 객체 생성 방식을 간단하게 설명하면 아래와 같다.
>
> 첫 로그인이후에, 검증된 `user` 의 정보를 `session` 객체에 값으로써 넣고, 임의의 `id` 값을  만들어 `session` 객체의 `key` 값으로 갖는다. 이 `key` 값을 `session id` 라고 부른다.

그리고 `server` 는 `client` 에 `cookie` 를 보낸다.
이후에 `client` 는 `server` 에 매 요청마다 `cookie` 를 같이 보내게 된다.

이러한 `cookie` 에 담긴값을 `session ID` 로써 제공하고, `session ID` 를 `session` 의 `key` 값으로 제공하여, `session` 에 담긴값을 참조해서 가져온다.

이렇게 하면, `user` 의 정보를 `cache` 로 제공하며,
`cookie` 값을 `sessionID` 로 제공하므로, `user` 정보를 직접 전달하는 방식보다 훨씬 안전한 방식이다. 

> 현재로써는 이러한 `cookie` 방식도 안전하지 않다고 판단되어, `cookie` 를 통한 인증과정을 없애는 추세라고는 한다.
>
> 이 부분에 대해서는 더 많이 공부가 필요할듯 보인다.

이로써 상태를 유지하면서, `HTTP` 응답 및 요청을 보낼 수 있도록 만들어 준다. 

하지만, `Application` 에서 `session` 을 사용할때, 인증과 관련없는 처리가 이루어질 수도 있다고 한다.

이러한 처리를 더 명확하게 구분하기 위해 `Passport` 에서는 인증 상태를 격리하여 처리하도록 설계되어 있다.

일단 `express` 에서 `session` 을 사용할 수 있도록 `express-session` 을 사용하여 미들웨어로 등록한다.

```ts

var session = require('express-session');

app.use(session({
  secret: 'keyboard cat',
  resave: false,
  saveUninitialized: false,
  cookie: { secure: true }
}));
app.use(passport.session());

```

이렇게 `session` 을 사용할 수 있도록 만든이후, `login` 세션을 유지하기 위해, `Passport` 에서 제공해주는 `serializeUser` 와 `deserializeUser` 를 사용하여 처리가 이루어져야 한다. 

여기서 `serialize(직렬화)` 및 `deserialize(역직렬화)` 라고 하는 용어를 사용하는데,  
`serialize` 는 사용자 객체를 세션에 저장 가능한 형식으로 변환하는 과정을 말한다.
`deserialize` 는 사용자의 정보를 읽어와서(`session` 객체에서 `sessionID` 를 통해서..) 원래의 사용자 객체로 변환하는 과정이라고 생각하면 된다.

이러한 방식을 다음의 `code` 를 통해 살펴본다.

```ts
passport.serializeUser(function(user, cb) { // <-- 여기의 user 는
                                            //     strategy 에서 인증된 user 이다.
                                            //     즉, strategy 성공시 호출된다.
                                            //     serializeUser 는 처음 로그인시에만 호출된다.
  process.nextTick(function() {
    return cb(null, {
      id: user.id,
      username: user.username,
      picture: user.picture
    }); // <-- `session` 에 고유식별자를 등록한다.
  });
});

passport.deserializeUser(function(user, cb) { // <-- 여기의 user 는 
                                              //     session 객체에서 가져온 user 값이다.
                                              //     deserializeUser 는 페이지 방문시 매번 호출된다.
  process.nextTick(function() {
    return cb(null, user);
  });
});
```

이때, 위의 예는 `session` 자체에 값을 `user` 객체로써 등록했는데, 이는 `tradeoff` 가 있다고 한다.
객체의 값을 저장하여 사용하므로, `req.user` 에 바로 값을 전달할 수 있는 장점이 있는 반면에, `session` 에 저장될 수 잇는 최대 데이터 양이 최과될 위험이 있다.  

반면에, `session` 값을 `id` 로 저장하고, `deserialize` 할때, 매번 `database` 에서 값을 가져와서 `req.user` 로 설정하는 로직이 있다고 해보자.

```ts

passport.serializeUser(function(user, cb) {
  process.nextTick(function() {
    return cb(null, user.id); // session 객체에 user.id 를 저장
  });
});

passport.deserializeUser(function(id, cb) { // session 에서 id 값을 가져온다
  db.get('SELECT * FROM users WHERE id = ?', [ id ], function(err, user) {
    if (err) { return cb(err); }
    return cb(null, user); // user 값을 db 에서 조회하여 req.user 에 할당한다.
  });
});

```

이러한 경우, 매번 `query` 가 필요하게 된다.
대신에 `session data` 를 최소화한다.

이러한 `balance` 를 맞추기 위해서, `Application` 에서 사용되는 모든 요청에 필요한 사용자 정보를 `session` 에 저장하고, 특정 경로에서 필요한 `data` 를 `query` 하는 방식을 사용하면 더 효과적으로 `query` 가 가능하다.

위의 방법을 사용하여 `session` 을 사용할 수 있다.
`session` 기반의 `stateful` 서버는 `client` 요청마다 클라이언트 상태를 유지하지만, `token` 기반을 사용하면, `stateless` 하며 상태정보를 유지없이 사용가능하다.

위 방식처럼 `session` 을 통해 `server` 에서 상태 정보를 저장하여 이전 요청에 대해 기억하는 것을 `stateful` 하다고 말할 수 있다

여기서 보통 `session` 으로 처리하는 방식의 단점은 말 그대로 상태정보를 저장하여 처리해야 하므로, `server` 의 `memory` 를 차치할수 밖에 없다는 것이다. 

또한, 만약 `server` 가 한개가 아닌 여러개의 `server` 로 관리되는 상황에서, `session` 의 상태를 관리하는 `A server` 에서 `B server` 로 통신이 변경된 상황을 생각해보자.

`B server` 는 이전의 상태를 알지 못하므로, 로그인된 `User` 로 인지하지 못한다.
이러한 상황에서는, `session` 을 `DB` 에서 처리하여 공통적으로 관리할 수 있는 환경이 갖추어져야 함을 뜻한다.

이러한 단점을 극복하고자, `Stateless` 상태에서도 `Login User` 인지 알 수 있는 방식을 제공하는데, `JWT(Json Web Token)`을 그것이 사용하면 가능하다.

### JWT

위의 설명처럼, `server` 에서 `client` 의 상태를 보존하지 않는다면 `stateless` 하다고 말한다.

`JWT(Json Web Token)` 는 `stateless` 한 상태에서 `client` 가 로그인했는지 안했는지 확인할 수 있는 방법은 어떠한 방식으로 구현되는지 아는것이 중요하다.

먼저, `JWT` 는 `RFC 7519` 표준에 의해 지정되었으며, `JSON` 객체로 정보를 안전하게 전송하기 위한 작고 독립적인 방식이라고 말한다

`JWT` 의 구조는 `.` 구분자에 의해 3개의 구조로 이루어져 있다.

1. Header
2. Payload
3. Signature

예를 들면 다음과 같다.

```JWT
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### Header

여기서 첫번째 구두점 구분자인 `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9` 는 `Header` 이다.

```json
{
  "alg": "HS256", // <-- 서명된 알고리즘 ex) HMAC, SHA256, RSA
  "typ": "JWT" // <-- Type
}

```

`Header` 는 일반적으로 두부분으로 나누어지는데, 서명된 알고리즘과 `JWT type` 인지 알려준다. 
이러한 `Header` 를 `Base64URL` 로 부호화 시킨것이 `JWT` 의 첫번째 부분이다.

#### Payload

`JWT` 의 두번째 부분은 `Token` 의 `Payload` 값이다.
이는 다음의 코드인 `eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ` 이다.

`Payload` 는 `claims` 를 가지고 있는데, `claims` 는 추가 `data` 및 `entity` 에 대한 구문이다.
> `claims` 라는 용어를 사용하는데, 이는 `JWT` 에 포함되는 정보를 나타낸다.  
이는 마치 객체의 프로퍼티처럼 이름-값 쌍으로 구성된다.

이 `claims` 는 3개의 타입이 존재하는데 다음과 같다

---
1. Registred claims  
미리 정의된 `claims` 의 묶음이며, 이는 필수는 아니지만 상호 운용 가능하기에 권장된다고 한다.  
이는 `RFC 7519` 에 의해 다음의 종류로 정의되어 있으며 전부 `Optional` 하므로 반드시 권장되지는 않지만 추천하는 부분이므로, 사용용도를 알고 준수하면서 작성하는것이 좋을 듯 싶다.  

- "lss"(Issuer) claim  
`JWT` 를 발행한 보안주체를 정의한다  

- "sub"(subject) claim  
`JWT` 의 제목에 대해 식별하는 주체이다.  
`sub` 정의시 반드시 `issuer context` 내 에서 지역적으로 `unique(고유)` 하거나, 전역적으로 `unique` 해야 한다.

- "aud"(Audience) claim  
`aud claim` 은 `JWT` 가 대상으로하는 수신자를 식별하기 위해 사용된다.  

- "exp"(Expiration Time) claim  
토큰의 만료 시간을 나타낸다

- "nbf"(Not Before) claim
토큰의 사용 가능한 시작 시간을 나타낸다

- "iat"(Issued At) claim
토큰의 발급 시간을 나타낸다

- "jti"(JWT ID) claim
`JTW` 의 고유 식별자를 타나낸다

---
2. Public Claims
`JWT` 를 사용하는 개발자들 사이에서 협의된 `claim` 이라고 한다.  
즉 마음대로 정의도 가능하지만, `collistion(충돌)` 를 방지하기위해 협의된 [`IANA JSON Web Token Registry`](https://www.iana.org/assignments/jwt/jwt.xhtml) 혹은 네임스페이스를 포함하는 `URL` 형태로 정의한다고 한다. 

```json

{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true,
  "https://www.MyNamespaceUrl.com/logindId": "idid" // <-- NameSpace URL 형태로 정의
}

```
---

3. Private Claims  
비공개 `claim` 이라고 부르며, `server` 와 `client` 간의 협의된 `claim` 이다.  
이 `claim` 은 수신자와 발급자간의 양쪽이 알고 있고 있고 사용된다.
이는 `Public claim` 과 충돌없이 사용하도록 한다.

```json
{
  "test_user_id": "ad11dacff12", // <-- 협의된 custom claim
  "test_user_name": "Jhon Doe" // <-- 협의된 custom claim
}
```
---

이 `Payload` 는 `Base64URL` 로 부호화되서 `JWT` 의 두번째 부분에 삽입된다.

#### Signature

`JWT` 는 암호화 알고리즘을 사용하여, `Base64URL` 을 통해 부호화된, `Header`, `Payload`, `secret` 을 이용해 암호화된다.  

이러한 암호화를 위해서는 지정된 알고리즘을 통해 `Signature(서명)` 작업이 이루어져야 한다.
`Docs` 에서는 예시로 `HMAC SHA256` 알고리즘을 사용하여 서명하는 것을 보여주고 있다

```ts
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  your-256-bit-secret
)
```
이 `Signature` 는 도중에 `message` 가 변경되지 않도록 검증하는데 사용되며, 이 경우에 `token` 은 `private key` 와 함께 `Sign` 된다.

만약 `message` 상에 변경이 있다면, 이렇게 만들어진 `Signature` 역시 달라지므로, 검증이 가능한 것이다.

또한, 이것으로 `JWT` 발신자가 누구인지 확인할 수도 있다.
이를 총 합친 `Encoded` 된 형태는 다음과 같다.

```JWT
// JWT Encoded

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

// JWT Decoded
---
Header
{
  "alg": "HS256",
  "typ": "JWT"
}
---
Payload
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
---
Signature
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  your-256-bit-secret
)
---
```

`JWT` 는 보안 문제를 방지하기 위해 세심한 주의가 필요하다.
위에도 설명했지만, `Header` 와 `Payload` 를 `Base64URL` 로 `Encoded` 되었기에,  
쉽게 `Base64URL` 로 `Decoded` 가 가능하다.

그러므로, 적재되어 있는 `Payload` 의 값에 보안상 민감한 정보를 포함했을때, 큰 위험이 발생할 수 있으므로, 토큰에 내용 적재시 필요이상으로 토큰을 보관해서는 안된다.

또한 일반적으로 `Bearer Schema` 를 사용하여, `Authorization` 헤더에 `JWT` 를 보내야한다.
해당 내용은 다음과 같다.

```HTTP
Authorization: Bearer <token>
```

이렇게 하면 `server` 의 보호된 경로는 `Authorization header` 안의 `JWT` 검증을 확인할 수 있고, 만약 존재한다면, 사용자는 보호된 `resource` 정보 에 접근 가능할 수 있다.

이렇게 접근 가능한 정보가 포함되어 있다면, `query` 를 통해 `DB` 상에서 내용을 가져오지 않더라도, 해당 정보를 가져와 처리가 가능하다.

> 물론, 추가적인 정보가 필요하다면 `DB` 를 통한 `Query` 는 이루어져야 한다.

`HTTP` 헤더를 통해 `JWT` 토큰을 보내는 경우에는 토큰이 사이즈가 커지지 않도록 주의해야 한다.  
일부 서버에서는 `8KB` 이상의 헤더를 허용하지 않는다고도 한다.

모든 사용자 권한 같은 너무 많은 정보를 포함하려는 경우에 [`like Auth0 Fine-Grained Authorization`](https://auth0.com/developers/lab/fine-grained-authorization) 같은 새로운 대채 수단이 필요할 수 있다고 경고하고 있다.

바로 이러한 방식으로, `stateless` 한 방식으로 정보를 주고 받을 수 있도록 만들어 준다.
이러한 `JWT` 에 대한 개념을 알았으니, 이번에는 `passport-jwt`  에 대해서 개념적 이해가 필요하다.

### Passport-jwt

이전에 앞에서 `passportjs` 는 `Strategies` 를 제공한다고 하였다.
앞전 설명은 `passport-local` 을 통해 `passport` 의 작동원리에 대해서 알아보았으며, 이번에는 `passport` 를 통해 `JWT` 를 다룰수 있는 `passport-jwt Strategies` 를 사용법에 대해서 살펴본다.  

`Docs` 에서는 이 `strategy` 를 다음처럼 설명한다.

> `JSON Web Token` 과 함께 인증을 위한 `Passport strategy`이다.
>
> 이 모듈은 `JSON Web Token` 을 사용하여 `endpoint` 에서 인증할 수 있다.
> 이것은 `session` 없이 `RESTful endpoint`를 안전하게 사용되도록 의도되었다.

이말은 간단하게 말하면, `session` 사용 안하므로, 안전하고 `stateless` 하게 `JWT` 를 사용하도록 `passport` 방식으로 만든것이라고 생각하면 된다.

> 앞에서 이미 언급했지만, `session` 은 `stateful` 하다. 반면 `JWT` 는 `server` 에서 상태관리를 하지 않으므로, `stateless` 하다고 한것이다.

사용하기 위해 해당 전략을 설치하도록 한다.

```sh
$ npm i passport-jwt
```

`JWT` 인증 전략을 위한 생성자는 다음과 같다

```ts

new JwtStrategy(options, verify)

```

`verify` 는 `LocalStrategy` 와 같은 검증 `callback` 함수이며, 여기서는 `options` 가 중요하다.

> options 목록

| name | description |
| :--- | :--- |
| secretOrKey | 이는 `token signature` 를 검증하기 위한 `string` 혹은 `buffer` 를 포함한 <br/>`secret`(대칭되는) 혹은 `PEM-encoded public key` 이다.<br/>`secertOrKeyProvider` 가 제공되지 않는한 필수값이다.  
| secretOrKeyProvider | 이는 `secretOrKeyProvider(request, rawJwtToken, done)` 으로 된 `callback` 함수이다.</br>이 함수는 주어진 키와 요청 조합에 대한 `secret` 혹은 `PEM-encoded public key` 로 `done` 을 호출해야 한다.</br>`done` 은 `done(err, secret)` 의 포멧을 갖는다. <br/>추가적으로 `rowJwtToken` 을 `decoding` 하는것은 구현자에게 달려있다.<br/>`secretOrKey` 가 없다면 필수적으로 제공되어야 한다.
| jwtFromRequest | 이 옵션은 필수값이다. 이 함수는 `request` 를 인수로써 받아들이고,<br/> 인코딩된 `JWT` 문자열 혹은 `null` 을 반환한다. <br/> 이부분은 추후 설명할 `Extracting the JWT from the request` 에서 확인해보도록 한다.
| issuer | 만약 정의된다면 `iss(claim)` 에 이 값이 정의된다 |
| audience | 만약 정의된다면, `aud(claim)` 에 이 값이 정의된다 |
| algorithmes | 허용된 `algorithmes` 의 이름이 포함된 문자열 목록이다.<br/>예를들면, ["HS256","HS384"] 같은.. |
| ignoreExpiration | 만약 `true` 이면 토큰의 유효기간 확인을 하지 않는다. |
| passReqToCallback | 만약 `true` 이면 `verify callback` 에 `request` 를 전달한다.<br/>예를들면, `verify(request, jwt_payload, done_callback)` 형식이다. |
| jsonWebTokenOptions | `passport-jwt` 는 `jsonwebtoken` 을 이용한 토큰을 확인한다.<br/>`jsonwebtoken` 검증자를 전달할 수 있는 다른 옵션에 대한 옵션 `object` 를 여기에 전달한다.

여기까지가 `JwtStrategy` 에 전달하는 옵션이다.  
이제 `verify` 를 통해 어떻게 구현되는지 살펴보도록 한다.

`verify` 함수는 `jwt_payload` 와 `done` 두개의 매개변수를 받는다. 

- `jwt_payload` 는 `decoded JWT payload`를 가진 `object literal` 이다.  
- `done` 은 첫번째 인자로 `error` 를 가지는 `callback` 함수이다.  
이는 다음의 인자를 받는다. `done(error, user, info)`  
생긴형태가 `LocalStrategy` 의 `verify 함수의 done` 과 같아보인다.

다음은 `Docs` 에 `bearer schema` 와 함께 `HTTP Authorization header` 로 부터 `JWT` 를 읽은 로직이다.

```ts
var JwtStrategy = require('passport-jwt').Strategy, 
    // <-- Stragy constructor 를 가져온다
    ExtractJwt = require('passport-jwt').ExtractJwt;
    // <-- header 에서 JWT 를 추출하기 위한 함수를 제공하는 ExtractJwt 를 가져온다
var opts = {}
opts.jwtFromRequest = ExtractJwt.fromAuthHeaderAsBearerToken();
    // <-- ExtractJwt.fromAuthHeaderAsBearerToken 를 통해 
    //     request 객체를 인수로 받아 실행하는 함수를 가진다.
opts.secretOrKey = 'secret';
opts.issuer = 'accounts.examplesoft.com';
opts.audience = 'yoursite.net';
passport.use(new JwtStrategy(opts, function(jwt_payload, done) {
  // JwtStrategy 를 생성하며, 인자로 첫번째로 opts 를, 두번째로 `verify callback` 함수를 전달
  // verify callback 함수에서 `jwt_payload` 는 `decoded JWT Payload` 가 존재한다.
    User.findOne({id: jwt_payload.sub}, function(err, user) {
      // `decoded JWT Payload` 의 `sub` 를 가져오고, `DB` 에서 조회한다.
      //  여기서 `sub` 는 `ID` 값이다.
        if (err) { // error 가 있다면
            return done(err, false); // DB 상 `error` 발생시, `Error` 를 전달
        }
        if (user) { // user 가 있다면
            return done(null, user); // done 값에 user 전달
        } else {
            return done(null, false); // 없다면, false 전달
                                      // 혹은 새로운 User 생성 로직을 써도 괜찮다
        }
    });
}));
```

#### Extracting the JWT from the request

`JWT` 가 `request` 에 포함되는 방법은 여러방식이 있다.  
가능한 유연하게 유지하기 위해 `JWT` 는 `jwtFromRequest` 매개변수로 전달된 사용자 제공 `callback` 에 의한 요청에서 `parsed` 한다

이 `callback` 은 `extractor` 라고 언급되는 것으로 부터, `request` 객체를 인수로 받고, `encoded JWT string` 혹은 `null` 을 반환한다.

`extractor` 는 `passport-jwt.ExtractJwt` 에 여러 `extractor factory function` 이 제공된다.
이 `factory function` 들은 지정된 매개변수로 구성된 새로운 `extractor` 를 반환한다.

- `fromHeader(header_name)`  
이 `extractor` 는 주어진 `http header` 에서 `JWT` 를 찾는 새로운 `extractor` 를 생성한다

- `fromBodyField(field_name)`
이 `extractor` 는 주어진 `body filed` 에서 `JWT` 를 찾는 새로운 `extractor` 를 생성한다  
이 `method` 를 사용하기 위해서는 반드시 `body parser` 가 설정되어야 한다.

- `fromUrlQueryParamter(param_name)`  
이 `extractor` 는 주어진 `URL query parameter` 에서 `JWT` 를 찾는 새로운 `extractor` 를 생성한다.

- `fromAuthHeaderWithSchema(auth_schema)`  
이 `extractor` 는 `authrization header` 에서 `JWT` 를 찾는 새로운 `extractor` 를 생성한다.  

- `fromAuthHeaderAsBearerToken()`  
이 `extractor` 는 `bearer schema`와 함께 `authorization header` 에서 `JWT` 를 찾는 새로운 `extractor` 를 생성한다.

- `fromExtractor([array of extractor functions])`  
이 `extractor` 는 제공된 `extractor` 들의 배열을 사용하는 새로운 `extractor` 를 생성한다.<br/>각 `extractor` 는 `token` 반환할 때까지 순서대로 시도된다.

이렇게 `Extractor` 를 통해 새로운 원하는 방식에 맞는 `Extractor` 를 반환하는 형태를 가진다.  
여기서 착각할 수 있는것이, 마치 위의 `ExtractJwt.Extractor` 자체가 `JWT` 를 반환하도록 알아서 처리되는것 처럼 보일수 있다는 것이다.

실제 동작은 그렇게 이루어지지 않고, 말그대로 새로운 `extractor` 를 반환한다는 것이다.
이는 다음의 로직처럼 구성되어 있다.

```ts
function fromAuthHeaderAsBearerToken() {
  return function (req) {
    let token = null;
    if (req.headers && req.headers.authorization) {
      const parts = req.headers.authorization.split(' ');
      if (parts.length === 2 && parts[0] === 'Bearer') {
        token = parts[1];
      }
    }
    return token;
  };
}
```

위는 `request` 객체를 인자로 받는 함수를 반환하며, `request` 객체를 통해 `JWT` 를 가져오는 가상의 로직이다.

`jwtFromRequest` 는 이렇게 `Extractor` 에 의해 생성된 함수를 참조하여 처리한다.

### Authentication requests

이제 `jwt` 전략을 사용하기 위해, `passport.authenticate` 에 지정한다

```ts
app.post('/profile', passport.authenticate('jwt', { session: false }), 
// <-- stateless 인 jwt 는 session 을 사용하지 않으므로, 옵션으로 session 값을 false 로 준다.
    function(req, res) {
        res.send(req.user.profile);
    }
);
```

이렇게, `req` 객체의 `user` 프로퍼티에서 `jwt` 토큰을 `deconding` 한 `payload` 참조가 가능하게 된다.
이렇게, `passport-jwt` 가 어떻게 작동하는지를 살펴보았다.

이제, `nextjs/passport` 를 살펴보도록 하자.
`nestjs` 의 `docs` 에서는 `passport` 사용시 `nestjs/passport` 를 사용하라고 한다.
이는 `passport` 를 `nestjs` 에서 사용가능하도록 만든 `library` 로써 사용된다.

내부적으로 `module` 은 다음처럼 구성되어 있다.

```ts

// auth-module.options.ts 
import { ModuleMetadata, Type } from '@nestjs/common';

export interface IAuthModuleOptions<T = any> {
  defaultStrategy?: string | string[];
  session?: boolean;
  property?: string;
  [key: string]: any;
}

export interface AuthOptionsFactory {
  createAuthOptions(): Promise<IAuthModuleOptions> | IAuthModuleOptions;
}

export interface AuthModuleAsyncOptions
  extends Pick<ModuleMetadata, 'imports'> {
  useExisting?: Type<AuthOptionsFactory>;
  useClass?: Type<AuthOptionsFactory>;
  useFactory?: (
    ...args: any[]
  ) => Promise<IAuthModuleOptions> | IAuthModuleOptions;
  inject?: any[];
}

export class AuthModuleOptions implements IAuthModuleOptions {
  defaultStrategy?: string | string[];
  session?: boolean;
  property?: string;
}

```

```ts
// passport.module.ts
import { DynamicModule, Module, Provider } from '@nestjs/common';
import {
  AuthModuleAsyncOptions,
  AuthModuleOptions,
  AuthOptionsFactory,
  IAuthModuleOptions
} from './interfaces/auth-module.options';

@Module({})
export class PassportModule {
  static register(options: IAuthModuleOptions): DynamicModule { 
    //<-- options 를 받아 provider 에 제공되도록 하는 `static register` 
    //    함수를 제공한다.
    //    이 함수는 동기적으로 작동한다.
    return {
      module: PassportModule,
      providers: [{ provide: AuthModuleOptions, useValue: options }],
      exports: [AuthModuleOptions]
    };
    // <-- return 값으로 module 을 반환하며, `options` 를 값으로하는 `provider` 를
    //     주입하며, 필요시 해당 `Provider` 를 `exports` 해서 사용할수 있도록 만든다 
  }

  static registerAsync(options: AuthModuleAsyncOptions): DynamicModule {
    //<-- options 를 받아 provider 에 제공되도록 하는 `static register` 
    //    함수를 제공한다.
    //    이 함수는 비동기적으로 작동한다.
    return {
      module: PassportModule,
      imports: options.imports || [],
      providers: this.createAsyncProviders(options),
      exports: [AuthModuleOptions]
    };
  }

// 밑은 provider 를 만들어주는 함수같다
  private static createAsyncProviders(
    options: AuthModuleAsyncOptions
    /*
    export interface AuthModuleAsyncOptions
      extends Pick<ModuleMetadata, 'imports'> {
      useExisting?: Type<AuthOptionsFactory>;
      useClass?: Type<AuthOptionsFactory>;
      useFactory?: (
        ...args: any[]
      ) => Promise<IAuthModuleOptions> | IAuthModuleOptions;
      inject?: any[];
    }
    이 AuthModuleAsyncOPtions 는 module 의 custom provider 모음을 
    제공하는 함수인듯 하다.
    
    아직, custom provider 에 익숙치 않은 듯하다.
    많이 사용하면서 익숙해지고 다시 살펴보아야 겠다.
   */
  ): Provider[] {
    if (options.useExisting || options.useFactory) {
      return [this.createAsyncOptionsProvider(options)];
    }
    return [
      this.createAsyncOptionsProvider(options),
      {
        provide: options.useClass,
        useClass: options.useClass
      }
    ];
  }

// 실제 options 를 받아서 provider 를 직접 생성하는 함수이다.
  private static createAsyncOptionsProvider(
    options: AuthModuleAsyncOptions
  ): Provider {
    if (options.useFactory) {
      return {
        provide: AuthModuleOptions,
        useFactory: options.useFactory,
        inject: options.inject || []
      };
    }
    return {
      provide: AuthModuleOptions,
      useFactory: async (optionsFactory: AuthOptionsFactory) =>
        await optionsFactory.createAuthOptions(),
      inject: [options.useExisting || options.useClass]
    };
  }
}

```

`Docs` 에서 사용된 `module` 을 살펴본다.

```ts
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { LocalStrategy } from './local.strategy';
import { JwtStrategy } from './jwt.strategy';
import { UsersModule } from '../users/users.module';
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
import { jwtConstants } from './constants';

@Module({
  imports: [
    UsersModule,
    PassportModule,
    JwtModule.register({
      secret: jwtConstants.secret,
      signOptions: { expiresIn: '60s' },
    }),
  ],
  providers: [AuthService, LocalStrategy, JwtStrategy],
  exports: [AuthService],
})
export class AuthModule {}
```

모듈을 가져오는 것을 볼 수 있다.
이때, 사실 모듈의 `register` 를 사용하지 않았으므로, 빈 객체를 반환하지 않을까 싶다.
굳이 `module` 을 가져와서 사용하는 의미가 있나 싶을 정도이다.

하지만, 일단 `docs` 에서는 `AuthModule` 을 사용하여 `Passport` 기능을 설정할 필요가 있어서 반드시 정의되어야 한다 말한다.

여기서, 단순히 `PassportModule` 로 `import` 를 했는데, 내가 이해한것이 맞다면, `register` 를 호출하지 않으면, 자동적으로 `register` 를 호출하는것과 같다고 이야기한다.

이렇게 `PassportModule` 을 입력해야만 `passport` 초기화 및 `middleware` 등록이 가능하도록 처리가 이루어진다.


