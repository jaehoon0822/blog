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
| secretOrKeyProvider | 이는 `secretOrKeyProvider(request, rawJwtToken, done)` 으로 된 `callback` 함수이다.<br/>이 함수는 주어진 키와 요청 조합에 대한 `secret` 혹은 `PEM-encoded public key` 로 `done` 을 호출해야 한다.<br/>`done` 은 `done(err, secret)` 의 포멧을 갖는다. <br/>추가적으로 `rowJwtToken` 을 `decoding` 하는것은 구현자에게 달려있다.<br/>`secretOrKey` 가 없다면 필수적으로 제공되어야 한다.
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

## 드디어, nestjs/passport 에 대해서 살펴보자!!

지금까지 `nestjs/passport` 를 살펴보기전 개념적 부분에 대해서 대략적으로 알아보았다

이제, `nextjs/passport` 를 살펴보도록 하자.
`nestjs` 의 `docs` 에서는 `passport` 사용시 `nestjs/passport` 를 사용하라고 한다.
이는 `passport` 를 `nestjs` 에서 사용가능하도록 만든 `library` 로써 사용된다.

처음으로 `passport` 를 사용하기 위한 `dynamicModule` 을 보도록 한다

### PassportModule

일단 Module 의 내부를 살펴보기 이전에 `nestjs/common` 에서 가져오는 `Type` 에 대해서 살펴보도록 한다.  

> `@nestjs/common` 의 `Type`
```ts
// nestjs/common 의 Type 
export interface Type<T = any> extends Function {
  new (...args: any[]): T;
}
```

위의 `Type` 인터페이스는 `new (...args: any[]): T` 를 가진구조를 가지고 있다.
이때 `T` 는 `Generic` 을 통해 받은 `T` 를 사용하여, 반환값을 설정해주는 구조로,  
이루어져 있다.

`new (...args: any[]): T` 부분은 `Constructor Signature` 로, 생성자를 가진 함수를 뜻한다.

> `Javascript` 는 `Prototype based Langauge` 라서, 함수내부에 `Constructor Function` 이 존재한다.  
> `new` 연산자와 이 `Constructor` 를 통해, 객체를 생성한다.
> 그래서 `new (...args: any[]): T` 는 `Constructor` 를 나타내는 `Signature` 를 나타낸다

즉, `T` 값을 반환하는 `Constructor` 임을 나타내는 함수(`Class 라고 봐도 된다.`) 를 나타낸다
이는 다음처럼 사용가능하다.

```ts
class MyClass {
  constructor(public name: string) {}
}

// Type<T>를 사용하여 MyClass의 타입을 지정
const myClassType: Type<MyClass> = MyClass;

// Type<T>로 생성된 타입을 사용하여 객체 인스턴스를 생성
const myInstance: MyClass = new myClassType('example');

// 객체 인스턴스의 프로퍼티에 접근
console.log(myInstance.name); // 출력: "example"
```

이렇게, 클래스로 부터 생성된 객체 인스턴스를 다룰수 있도록 만들어준다.
 `PassprotModule` 부분은 이러한 `Type` 을 사용하는 몇개의 코드가 보이므로 참고하도록 한다. 

또한 `ModuleMetadata` 를 가져와 사용하기도 한다.
이 부분은 `Module` 을 구성하는 `Interface` 로써, 사용되므로 코드자체는 그렇게 어렵지는 않을 것이다.

> `@nestjs/common` 의 `ModuleMetadata`
```ts
import { Abstract } from '../abstract.interface';
import { Type } from '../type.interface';
import { DynamicModule } from './dynamic-module.interface';
import { ForwardReference } from './forward-reference.interface';
import { Provider } from './provider.interface';

/**
 * Interface defining the property object that describes the module.
 *
 * @see [Modules](https://docs.nestjs.com/modules)
 *
 * @publicApi
 */
export interface ModuleMetadata {
  /**
   * Optional list of imported modules that export the providers which are
   * required in this module.
   */
  imports?: Array<
    Type<any> | DynamicModule | Promise<DynamicModule> | ForwardReference
  >;
  /**
   * Optional list of controllers defined in this module which have to be
   * instantiated.
   */
  controllers?: Type<any>[];
  /**
   * Optional list of providers that will be instantiated by the Nest injector
   * and that may be shared at least across this module.
   */
  providers?: Provider[];
  /**
   * Optional list of the subset of providers that are provided by this module
   * and should be available in other modules which import this module.
   */
  exports?: Array<
    | DynamicModule
    | Promise<DynamicModule>
    | string
    | symbol
    | Provider
    | ForwardReference
    | Abstract<any>
    | Function
  >;
}
```

`passportModule` 에서 사용하는 `options` 인터페이스를 살펴보도록 한다.

```ts

// auth-module.options.ts 
import { ModuleMetadata, Type } from '@nestjs/common';

export interface IAuthModuleOptions<T = any> {
  defaultStrategy?: string | string[]; // 사용될 기본 Strategy 
  session?: boolean; // session 을 사용할지 안할지 boolean 값
  property?: string; // request 객체에 저장될 property name 지정
  [key: string]: any; // 그 외...
}

export interface AuthOptionsFactory {
  createAuthOptions(): Promise<IAuthModuleOptions> | IAuthModuleOptions;
} // createAuthOptions 함수를 가진 인터페이스로 반환값은 IAuthModuleOptions 이다.

export interface AuthModuleAsyncOptions
  extends Pick<ModuleMetadata, 'imports'> {
    // 다른 모듈을 확장할 수 있도록
    // ModuleMetadata 중 imports 를 가져와 확장시킨다.
    // 밑은 나머지 추가적인 함수들이다.
  
  useExisting?: Type<AuthOptionsFactory>;
  // 기존에 존재하는 `AuthOptionsFactory` 클래스 타입을 사용하여 옵션을 생성 
  useClass?: Type<AuthOptionsFactory>;
  // 새로운 `AuthOptionsFactory` 클래스 타입을 생성하여 옵션을 생성
  useFactory?: (
    ...args: any[]
  ) => Promise<IAuthModuleOptions> | IAuthModuleOptions;
  // factory 함수를 사용하여 옵션을 생성
  // factory 함수는 paramter 를 받아 `IAuthModuleOptions` 을 생성한다.
  inject?: any[];
  // 의존성 주입을 위한 token 배열
  // 이 주입된 token 을 사용하여 위의 클래스 생성자에서 받아 처리 가능하다
}

export class AuthModuleOptions implements IAuthModuleOptions {
  defaultStrategy?: string | string[];
  session?: boolean;
  property?: string;
} // IAuthModuleOptions 을 구현한 클래스

```

위는 `dynamic Module` 의 `options` 들을, 동적으로 생성해주는 코드라고 이해하면 될듯하다.
이때, 위의 `options` 를 준수하여 `module` 을 생성하는것은 매우 중요하다.
실제 위 `options` 들을 `passport module` 에서 가져와서 사용하는것을 볼 수 있다.

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

`docs` 에서는 `AuthModule` 을 사용하여 `Passport` 기능을 설정할 필요가 있어서 반드시 정의되어야 한다 말한다.

여기서, 단순히 `PassportModule` 로 `import` 를 했는데, 내가 이해한것이 맞다면, `register` 를 호출하지 않으면, 자동적으로 `register` 를 호출한다.

이후, `Strategy` 들을 설정하는 로직이 실행된다.

`nestjs/passport` 의 내부 로직중, 따로 `options` 설정이 되어 있지 않다면, 
`default options` 이 지정되어 있는데 다음과 같다.

```ts
// passport/lib/options.ts

export const defaultOptions = {
  session: false,
  property: 'user'
};

```

보면 알겠지만, `session` 사용은 `default` 로 `false` 처리 되어 있으며,
`property` 값은 기본적으로 `user` 로 되어 있으므로, `request` 객체를 통해  
`user` 정보를 가져올때, `request.user` 로 접근해야함을 말한다.

자 그럼, 이렇게 `DynamicModule` 을 통해, `PassportModule` 을 만들어내는것을 알게 되었다.
그렇다면, `Passport Strategy` 는 어떻게 사용할지 살펴본다. 

### nestjs/Passport Strategy 

`Docs` 에서는 다음처럼 `Strategy` 를 등록하라고 명시되어 있다.

```ts
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';
import { jwtConstants } from './constants';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: jwtConstants.secret,
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

사실 이부분만 보더라도, `Logic` 자체가 쉽게 이해가는 로직은 아니다.
단순히 추상적으로 생각하자면 `PassportStrategy` 를 `@nestjs/passport` 에서 가져온후,  
`class` 로 확장하면 알아서 처리해주는구나.

라고 생각하고 사용하면 편하지만, 이해없이 사용하면 이 자체를 그냥 외워서 사용하는것 밖에 되지 않는다.  

지금 당장은 모면할 수 있더라도 장기적으로 보면 좋은 방식은 아니다.
일단 `PassportStrategy` 가 어떻게 이루어져 있는지 먼저 살펴보도록 한다.

```ts
// passport/lib/passport/passport.strategy.ts
import * as passport from 'passport';
import { Type } from '../interfaces';

export function PassportStrategy<T extends Type<any> = any>(
  Strategy: T,
  name?: string | undefined
): {
  new (...args): InstanceType<T>;
} {
  abstract class MixinStrategy extends Strategy {
    abstract validate(...args: any[]): any;

    constructor(...args: any[]) {
      const callback = async (...params: any[]) => {
        const done = params[params.length - 1];
        try {
          const validateResult = await this.validate(...params);
          if (Array.isArray(validateResult)) {
            done(null, ...validateResult);
          } else {
            done(null, validateResult);
          }
        } catch (err) {
          done(err, null);
        }
      };
      /**
       * Commented out due to the regression it introduced
       * Read more here: https://github.com/nestjs/passport/issues/446
        const validate = new.target?.prototype?.validate;
        if (validate) {
          Object.defineProperty(callback, 'length', {
            value: validate.length + 1
          });
        }
      */
      super(...args, callback);

      const passportInstance = this.getPassportInstance();
      if (name) {
        passportInstance.use(name, this as any);
      } else {
        passportInstance.use(this as any);
      }
    }

    getPassportInstance() {
      return passport;
    }
  }
  return MixinStrategy;
}
```

`PassportStrategy` 는 위와 같은 형태를 띄고 있다.
여기서 눈여겨 볼것은, 해당 함수의 `Strategy: T,  name?: string | undefined`   
부분과 `{ new (...args): InstanceType<T>; }` 부분이다.

`Staregy` 를 받고, `name` 값도 지정가능하지만 선택적 인자이다.

```ts
export function PassportStrategy<T extends Type<any> = any>(
  Strategy: T, // <-- 이부분
  name?: string | undefined
): {
  new (...args): InstanceType<T>;
} { ... }
```

이 `Strategy class` 를 반환하는 생성자를 가진 `Class` 를 반환하는 로직이다.
```ts
export function PassportStrategy<T extends Type<any> = any>(
  Strategy: T, 
  name?: string | undefined
): {
  new (...args): InstanceType<T>; // <-- 이부분
} { ... }
```

이는 `Strategy` 를 인자로 받는 다음의 코드에서 볼수 있다.

```ts

import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';
import { jwtConstants } from './constants';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {...}
```

앞전의 `JwtStrategy` 전략을 만들때의 코드이다.
여기서 `Strategy` 를 인자값으로 넘겨서 처리하는것을 볼 수 있다. 

이때 넘겨진 `Strategy` 는 `Passport-jwt` 에서 넘겨주는 `jwt` 를 기반으로하는 `Strategy` 이다. 
이러한 타입만을 살펴보면 내용은 간단하다.

> 반환타입은 해당 `Strategy` 의 `class` 의 인스턴스 타입을 반환하는 생성자를 뜻한다.

그래서 이러한 `class` 인스턴스 타입을 반환하는 `abstract class` 를 구현한다.

```ts

import * as passport from 'passport';
import { Type } from '../interfaces';

export function PassportStrategy<T extends Type<any> = any>(
  Strategy: T,
  name?: string | undefined
): {
  new (...args): InstanceType<T>;
} {
  abstract class MixinStrategy extends Strategy {
    ...
  }
  return MixinStrategy;
}
```

이렇게 `MixinStrategy` 를 `Strategy`로 확장한 `추상 Class` 를 만들고, 내용을 구현한다.
첫번째로 `getPassportInstance` 메서드를 만들어서, `import` 한 `passport` 를 반환하는 메서드먼저 살펴본다.

```ts
import * as passport from 'passport';
import { Type } from '../interfaces';

export function PassportStrategy<T extends Type<any> = any>(
  Strategy: T,
  name?: string | undefined
): {
  new (...args): InstanceType<T>;
} {
  abstract class MixinStrategy extends Strategy {
    ...

    getPassportInstance() {
      return passport
    }
  }
  return MixinStrategy;
}
```

이렇게 구성되어 있는데, 아직 `OOP` 에 대한 지식이 많이 부족한듯 보인다.
굳이, 이렇게 따로 메서드를 만들면서 처리하는 이유가 존재할것인데 쉽게 와 닿지는 않는다.

이런방식으로 구현하는 부분은 외부에서 주입받지 않고 내부에서 사용하도록 처리하기 위함인듯 싶다.
이로인해 모듈 내부에 대한 결합도를 낮추어서 사용하는건가 싶기도 하다.
아니면 다른 이유가 있는것인가?

아직은 이부분에 대해 명확히 이해가 가지는 않는 상황이다.

그리고 이 `getPassportInstance` 를 사용하여 `passport` 를 사용하는 로직은 `constructor` 에서 사용한다.

```ts
mport * as passport from 'passport';
import { Type } from '../interfaces';

export function PassportStrategy<T extends Type<any> = any>(
  Strategy: T,
  name?: string | undefined
): {
  new (...args): InstanceType<T>;
} {
  abstract class MixinStrategy extends Strategy {
    abstract validate(...args: any[]): any;

    constructor(...args: any[]) {
      const callback = async (...params: any[]) => {
        const done = params[params.length - 1];
        try {
          const validateResult = await this.validate(...params);
          if (Array.isArray(validateResult)) {
            done(null, ...validateResult);
          } else {
            done(null, validateResult);
          }
        } catch (err) {
          done(err, null);
        }
      };
      /**
       * Commented out due to the regression it introduced
       * Read more here: https://github.com/nestjs/passport/issues/446
        const validate = new.target?.prototype?.validate;
        if (validate) {
          Object.defineProperty(callback, 'length', {
            value: validate.length + 1
          });
        }
      */
      super(...args, callback);

      const passportInstance = this.getPassportInstance();
      if (name) {
        passportInstance.use(name, this as any);
      } else {
        passportInstance.use(this as any);
      }
    }

    getPassportInstance() {
      return passport;
    }
  }
  return MixinStrategy;
}
```

이렇게 보면 복잡하니, `constructor` 부분만 따로 빼서 살펴본다.

```ts

    constructor(...args: any[]) {
      const callback = async (...params: any[]) => {
        const done = params[params.length - 1];
        try {
          const validateResult = await this.validate(...params);
          if (Array.isArray(validateResult)) {
            done(null, ...validateResult);
          } else {
            done(null, validateResult);
          }
        } catch (err) {
          done(err, null);
        }
      };
      super(...args, callback);

      const passportInstance = this.getPassportInstance();
      if (name) {
        passportInstance.use(name, this as any);
      } else {
        passportInstance.use(this as any);
      }
    }
```

로직 자체만 보면 간단하게 구성되어 있다.
`passport` 의 `Stragy` 에 사용될 `callback` 은 가장 마지막의 인자로 `done` 함수를 받는다.
이러한 `done` 함수는 `verify` 함수에서 전달한 값을, `request.user` 에 담아주는 역할을 한다. 
이부분은 앞전의 `Passport` 관련 내용을 참고하도록 한다.

그러므로, 이러한 `done` 을 유동적으로 사용하기위해, 변수로 선언한후, `parameter` 의 가장 마지막 값을 할당한다.  

이후, `this.validate(...params)` 를 실행하는데,  

```ts
mport * as passport from 'passport';
import { Type } from '../interfaces';

export function PassportStrategy<T extends Type<any> = any>(
  Strategy: T,
  name?: string | undefined
): {
  new (...args): InstanceType<T>;
} {
  abstract class MixinStrategy extends Strategy {
    abstract validate(...args: any[]): any; // <-- 여기

    constructor(...args: any[]) {
      ...
    }

    getPassportInstance() {
      return passport;
    }
  }
  return MixinStrategy;
}
```

`this.validate` 는 `abstract method` 로 만들어, 실제 `Class` 구현시 해당 `Strategy` 에 맞도록 구현하도록 한다.

이렇게 잘 실행된 `this.validate` 함수는 그 결과값을 내보내는데, 그 결과를 `validateResult` 변수에 담고, `done` 을통해 처리하도록 한다.

이러한 처리가 이루어지는 `callback` 함수를 만들고, `super` 를 통해, 부모 `Class` 로 보내어 해당 `Strategy` 를 생성할 수 있도록 만든다.

```ts

    constructor(...args: any[]) {
      // callback 생성
      const callback = async (...params: any[]) => {
        const done = params[params.length - 1];
        try {
          const validateResult = await this.validate(...params);
          if (Array.isArray(validateResult)) {
            done(null, ...validateResult);
          } else {
            done(null, validateResult);
          }
        } catch (err) {
          done(err, null);
        }
      };
      // 부모 Strategy Class 에 인자와, callback 전달
      super(...args, callback);

      // Strategy 등록
      const passportInstance = this.getPassportInstance();
      if (name) {
        passportInstance.use(name, this as any);
      } else {
        passportInstance.use(this as any);
      }
    }
```

이후에는, 만든 `Strategy` 를 등록하는 과정이 필요하다.  
이를 위해 아까 만든 `getPassportInstance` 를 불러온후,해당 `passportInstance` 의 `use` 를 사용하여  
등록하도록 한다.

이때, `MixinStrategy` 의 인자로 `name` 값이 있다면, 해당 전략의 이름을 인자로 받은 `name` 값으로 설정하고, 그렇지 않다면, `Strategy` 의 이름으로한 전략을 등록한다.

다시, `Strategy` 를 구현하는 로직으로 되돌아가보자.

```ts
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';
import { jwtConstants } from './constants';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: jwtConstants.secret,
    });
  }

  async validate(payload: any) {
    return { userId: payload.sub, username: payload.username };
  }
}
```

이제 이 로직이 이해가 가기 시작한다.
위 로직에서 `PassportStrategy` 를 불러와 사용할 `Strategy` 를 인자값으로 넘겨주면,  
추상클래스인 `MixinStrategy` 를 상속받아서 `JwtStrategy` 를 구현하게 된다.

이때, 추상메서드인 `validate` 함수를 직접 구현했으며, 이는 추후, 사용되는 전략에  
넘길 `callback` 의 `done` 에 넘기는 값을 리턴해주는것임을 알수 있다.

`constructor` 를 보면 `super` 를 통해 옵션값을 넘기는 것을 볼 수 있는데,  
이를 통해 `MixinStrategy` 의 `...args` 로 해당 옵션값이 넘어가고, `MixinStrategy` 가 확장한  
`Strategy` 에 `...args` 와 `callback` 을 `super` 를통해 전달되는것으로 이해할 수 있다.

이러한 모든 동작을 `class` 로 추상화하여, 간단한 로직만을 보여주어 처리할 수 있도록 만든것이  
객체지향의 장점인가 다시한번 느끼게된다.

이렇게 `nestjs/passport` 의 `Strategy` 가 어떠한 동작으로 작동하는지 살펴보았다.
그렇다면, `passport` 는 해당 전략을 사용하기 위해, `passport.authenticate` 를 사용하여 등록된 전략을 호출해야 하는데, 이 로직은 어디에 있을까?

이를 사용하기 위해 `nestjs/passort` 의 `guard` 를 사용하여 구현해야만 한다.

### nestjs/passport Guard

현재까지 블로그 작성하면서 가드 관련된 부분에 대해서 정리해놓은 글이 없다.
`nestjs` 의 `Guard` 는 `권한`, `역할`, `ACL` 등에 따라 요청이 라우트 핸들러에 의해 처리될지 여부를 결정하는데 사용된다.

사용법은 기존의 `Provider` 와 비슷한데, `@Injectable()` 데커레이터와 같이 사용한다.

추가적인것이 있다면, `canActive` 라는 인터페이스를 불러와서 구현해야 한다.

`canActive` 는 현재 요청이 혀용되는지 여부를 `boolean` 값으로 반환하는 함수이다.

```ts
@Injectable()
export class AuthAGuard implements CanActive {
  constructor(private readonly authSrevice: AuthService) {}

  canActive(context: ExecutionContext) {
    const user = this.authService.getCurrentUser();
    if (!user) {
      return false
    }
    return true;
  }
}
```

여기서 궁금한 부분은 `ExecutionContext` 와 `CanActive` 는 의 구현사항이다.

> `nestjs` 에 있는 `CanActive` 인터페이스
```ts
// nest/packages/commons/interface/features/can-active.interface.ts

import { Observable } from 'rxjs';
import { ExecutionContext } from './execution-context.interface';

/**
 * Interface defining the `canActivate()` function that must be implemented
 * by a guard.  Return value indicates whether or not the current request is
 * allowed to proceed.  Return can be either synchronous (`boolean`)
 * or asynchronous (`Promise` or `Observable`).
 *
 * @see [Guards](https://docs.nestjs.com/guards)
 *
 * @publicApi
 */
export interface CanActivate {
  /**
   * @param context Current execution context. Provides access to details about
   * the current request pipeline.
   *
   * @returns Value indicating whether or not the current request is allowed to
   * proceed.
   */
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean>;
}
```

여기에 `CanActive interface` 코드가 있다.
구현사항으로는, `canActive` 함수를 구현하도록 되어있을뿐, 다른것은 없다.

그리고, 반환값역시 예상대로 `boolean` 이며, `Sync` 인지 `Async` 인지에 따라 다를뿐 동일하다.

이제 `canActivate` 의 인자인 `context` 의 타입 `ExecutionContext` 에 대해서 살펴보자.

```ts
import { Type } from '../index';
import { ArgumentsHost } from './arguments-host.interface';

/**
 * Interface describing details about the current request pipeline.
 *
 * @see [Execution Context](https://docs.nestjs.com/guards#execution-context)
 *
 * @publicApi
 */
export interface ExecutionContext extends ArgumentsHost {
  /**
   * Returns the *type* of the controller class which the current handler belongs to.
   */
  getClass<T = any>(): Type<T>;
  /**
   * Returns a reference to the handler (method) that will be invoked next in the
   * request pipeline.
   */
  getHandler(): Function;
}
```

`ExecutionContext` 는 `AgrumentsHost` 를 상속받아 구현되어 있으며, 더불어 `getClass` 와 `getHandler` 를 가진 인터페이스이다.

일단, `ExecutionContext` 에서 구현하는메서드를 살펴보도록한다.

- getClass  
이 메서드는 현재 실행중인 `handler` 가 속한 `controller class` 를 리턴한다.

- getHandler
이 메서드는 `request pipeline` 에서 다음에 호출될 핸들러에 대한 참조를 리턴한다

그럼 `request pipeline` 이란 무엇일까?

> `NestJs` 어플리케이션에서 요청이 처리되는 방법을 말한다.
> 보통 `request` 를 처리할때, `pipeline` 을 갖는데, 그 순서는 다음과 같다. 
>
> Middleware -> gaurd -> pipe -> controlelr -> servide 
>
> 이러한 시퀀스로 구성되어, 처리가 이루어지는데 이러한 순서를
> `requst pipeline` 이라고 부른다.

그렇다면, 꼬리에 꼬리를물어 `ArgumentsHost` 에 대해서 보도록 하자.

```ts

export type ContextType = 'http' | 'ws' | 'rpc';

/**
 * Methods to obtain request and response objects.
 *
 * @publicApi
 */
export interface HttpArgumentsHost {
  /**
   * Returns the in-flight `request` object.
   */
  getRequest<T = any>(): T;
  /**
   * Returns the in-flight `response` object.
   */
  getResponse<T = any>(): T;
  getNext<T = any>(): T;
}

/**
 * Methods to obtain WebSocket data and client objects.
 *
 * @publicApi
 */
export interface WsArgumentsHost {
  /**
   * Returns the data object.
   */
  getData<T = any>(): T;
  /**
   * Returns the client object.
   */
  getClient<T = any>(): T;
}

/**
 * Methods to obtain RPC data object.
 *
 * @publicApi
 */
export interface RpcArgumentsHost {
  /**
   * Returns the data object.
   */
  getData<T = any>(): T;

  /**
   * Returns the context object.
   */
  getContext<T = any>(): T;
}

/**
 * Provides methods for retrieving the arguments being passed to a handler.
 * Allows choosing the appropriate execution context (e.g., Http, RPC, or
 * WebSockets) to retrieve the arguments from.
 *
 * @publicApi
 */
export interface ArgumentsHost {
  /**
   * Returns the array of arguments being passed to the handler.
   */
  getArgs<T extends Array<any> = any[]>(): T;
  /**
   * Returns a particular argument by index.
   * @param index index of argument to retrieve
   */
  getArgByIndex<T = any>(index: number): T;
  /**
   * Switch context to RPC.
   * @returns interface with methods to retrieve RPC arguments
   */
  switchToRpc(): RpcArgumentsHost;
  /**
   * Switch context to HTTP.
   * @returns interface with methods to retrieve HTTP arguments
   */
  switchToHttp(): HttpArgumentsHost;
  /**
   * Switch context to WebSockets.
   * @returns interface with methods to retrieve WebSockets arguments
   */
  switchToWs(): WsArgumentsHost;
  /**
   * Returns the current execution context type (string)
   */
  getType<TContext extends string = ContextType>(): TContext;
}

```

`ArgumentsHost Interface` 는 핸들러에 전달되는 인수를 검색하는 메서드를 제공한다.

이때, 각 메서드는 종류에 따라 다른 인터페이스를 갖는데, 각 사용하는 인터페이스는 다음의 의미를 가지고 만들어졌다. 

- `HttpArgumentHost`  
`HTTP` 요청과 응답을 검색하는 메서드를 제공

- `WsArgumentsHost`  
`WebSocket` 데이터와 클라이언트를 검색하는 메서드를 제공

- `RpcArgumentsHost`  
`Rpc`데이터와 컨텍스트를 검색하는 메서드를 제공

이제 이 인터페이스를 반환하는 `ArgumentsHost` 의 각 메서드를 살펴본다.

- `getArgs()`  
핸들러에 전달되는 인수의 배열을 반환

- `getArgsByIndex()`  
특정 인수를 인텍스별로 반환

- `switchToRpc()`  
`RPC` 컨텍스트로 전환

- `switchToHttp()`  
`HTTP` 컨텍스트로 전환

- `switchToWs()`  
`WS` 컨텍스트로 전환

- `getType()`  
현재 실행 컨텍스트의 유형을 반환

이와 같은 방식의 `ArgumentsHost` 를 확장한 `ExecutionContext` 를 사용하면, `request pipeline` 상 `request` 가 `service` 로직으로 가기전에 `HTTP` 의 `request` 객체를 확인하고 객체를 넘길지 안넘길지 결정이 가능하다.

즉, 말그대로 중간에서 `Guard` 할수 있는것이다.
이제 `Guard` 에 대해서 알게 되었으니, 다시 `nestjs/passport` 의 `Guard` 에 대해서 살펴본다.

```ts
import {
  CanActivate,
  ExecutionContext,
  Inject,
  Logger,
  mixin,
  Optional,
  UnauthorizedException
} from '@nestjs/common';
import * as passport from 'passport';
import { Type } from './interfaces';
import {
  AuthModuleOptions,
  IAuthModuleOptions
} from './interfaces/auth-module.options';
import { defaultOptions } from './options';
import { memoize } from './utils/memoize.util';

export type IAuthGuard = CanActivate & {
  logIn<TRequest extends { logIn: Function } = any>(
    request: TRequest
  ): Promise<void>;
  handleRequest<TUser = any>(
    err,
    user,
    info,
    context: ExecutionContext,
    status?
  ): TUser;
  getAuthenticateOptions(
    context: ExecutionContext
  ): IAuthModuleOptions | undefined;
};
export const AuthGuard: (type?: string | string[]) => Type<IAuthGuard> =
  memoize(createAuthGuard);

const NO_STRATEGY_ERROR = `In order to use "defaultStrategy", please, ensure to import PassportModule in each place where AuthGuard() is being used. Otherwise, passport won't work correctly.`;
const authLogger = new Logger('AuthGuard');

function createAuthGuard(type?: string | string[]): Type<CanActivate> {
  class MixinAuthGuard<TUser = any> implements CanActivate {
    @Optional()
    @Inject(AuthModuleOptions)
    protected options: AuthModuleOptions = {};

    constructor(@Optional() options?: AuthModuleOptions) {
      this.options = options ?? this.options;
      if (!type && !this.options.defaultStrategy) {
        authLogger.error(NO_STRATEGY_ERROR);
      }
    }

    async canActivate(context: ExecutionContext): Promise<boolean> {
      const options = {
        ...defaultOptions,
        ...this.options,
        ...(await this.getAuthenticateOptions(context))
      };
      const [request, response] = [
        this.getRequest(context),
        this.getResponse(context)
      ];
      const passportFn = createPassportContext(request, response);
      const user = await passportFn(
        type || this.options.defaultStrategy,
        options,
        (err, user, info, status) =>
          this.handleRequest(err, user, info, context, status)
      );
      request[options.property || defaultOptions.property] = user;
      return true;
    }

    getRequest<T = any>(context: ExecutionContext): T {
      return context.switchToHttp().getRequest();
    }

    getResponse<T = any>(context: ExecutionContext): T {
      return context.switchToHttp().getResponse();
    }

    async logIn<TRequest extends { logIn: Function } = any>(
      request: TRequest
    ): Promise<void> {
      const user = request[this.options.property || defaultOptions.property];
      await new Promise<void>((resolve, reject) =>
        request.logIn(user, (err) => (err ? reject(err) : resolve()))
      );
    }

    handleRequest(err, user, info, context, status): TUser {
      if (err || !user) {
        throw err || new UnauthorizedException();
      }
      return user;
    }

    getAuthenticateOptions(
      context: ExecutionContext
    ): Promise<IAuthModuleOptions> | IAuthModuleOptions | undefined {
      return undefined;
    }
  }
  const guard = mixin(MixinAuthGuard);
  return guard;
}

const createPassportContext =
  (request, response) => (type, options, callback: Function) =>
    new Promise<void>((resolve, reject) =>
      passport.authenticate(type, options, (err, user, info, status) => {
        try {
          request.authInfo = info;
          return resolve(callback(err, user, info, status));
        } catch (err) {
          reject(err);
        }
      })(request, response, (err) => (err ? reject(err) : resolve()))
    );
```

여기서 중요한 부분은 `canActive` 부분과 `createPassportContext` 로 생각이든다.

`createPassportContext` 는 `passport` 의 `authenticate` 를 실행시켜, 해당 전략이 실행될수 있도록 만드는 함수이다.

`MixinAuthGuard` 안의 `canActivate` 는 실제 `Guard` 의 `canActive` 를 실행하고, 아까 언급한 `createPassportContext` 함수를 호출해서 처리할 수 있도록 해주는 함수이다.

만약, `callback` 으로 `handleRequest` 를 사용하여, `user` 가 없거나, `err` 가 있으면, `UnauthorizedException` 이 발생하며, 그렇지 않을경우 `user` 값을 내보낸다.

그러한 `user` 값을 `options.property` 값을 `key` 로 하는 `request` 객체에 할당하고, `true` 를 반환하는 로직이다.

이로인해, `Passport` 에서 제공하는 `authenticate` 관련 로직도 처리되며, 중간에 `user` 가 있는지 없는지에 따라 `boolean` 값 및 `exception` 을 실행시키는 모든 조건을 충족한 `Guard` 가 만들어졌다.

이제 `Docs` 에서 `Guard` 를 어떻게 사용하고 있는지 살펴보도록 한다.

```ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}

```

위는 `jwt` 를 넘겨서 `passport.authenticate` 를 실행하는 로직이라는것을 볼수 있다.

`passport` 는 `passport.authenticate` 를 실행할때, 해당 전략을 실행한후, 그 전략의 반환된 값을 받아 `requst` 객체의 `user` (`default property` 가 `user` 라면...) 에 값을 넣어준다.

`Guard` 는 이러한 방식을 구현해줌과 동시에, 중간에서 `canActivate` 함수를 실행시켜, `Strategy` 에서 반환된 값이 없거나 `error` 가 발생하면, 더이상 진행되지 않도록 막아주는 효과도 같이 병행해준다.

그리고 그 `Gaurd` 를 사용하는 로직이다.

```ts
import { Controller, Get, Request, Post, UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from './auth/jwt-auth.guard';
import { LocalAuthGuard } from './auth/local-auth.guard';
import { AuthService } from './auth/auth.service';

@Controller()
export class AppController {
  constructor(private authService: AuthService) {}

  @UseGuards(JwtAuthGuard)
  @Get('profile')
  getProfile(@Request() req) {
    return req.user;
  }
}
```

`/profile` 경로에서, `@UseGuards` 데커레이터를 사용하고, `JwtAuthGuard` 를 사용하도록 한다.

이로인해, `user` 를 반환하지 못한다면, 통과되지 않고 `UnauthorizedException` 을 발생시키고, 그에 맞는 `Error` 값을 보내게 될것이다.

만약, `user` 가 존재한다면, `request.user` 객체에 해당 `user data` 를 넣어주고, `getProfile` 을 실행시킨다.

내부 로직을 모를때는 어떠한 연관관계에서 처리 되었는지 몰랐지만, 이제는 그 흐름이 이해가 간다.

`code` 와 함께 그 흐름을 분석하면서 알아가다 보니, 모든 코드를 완벽하게 이해했다고 말하기는 어렵지만, `passport` 를 어떻게 `wrapping` 하고, `nestjs` 에 맞도록 만들었는지 알게된 귀중한 시간이었던것 같다.

> 물론, 이렇게 코드분석하고 찾아보고 하나하나 알아보기까지 시간이 조금 걸려서 진도가 늦어진부분이 없지 않아 있다...ㅠㅠ
>

일허게 `passport` 동작원리를 이해하기 위해 앞으로 더 알아야 할 것들을 알게 된 기분이다.

추후 시간이 된다면 `OAuth` 관련 부분과 함께, `HTTP`, `Network` 지식을 쌓아가도록 노력해야 겠다.