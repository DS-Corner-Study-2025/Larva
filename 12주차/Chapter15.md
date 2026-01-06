# 15.1 서비스 운영을 위한 패키지

- 서비스를 출시한 이후에 서버에 문제가 생기면 서비스 자체에 심각한 타격
- 서비스의 취약점을 노린 공격이 들어올 수도 있음
- 이러한 문제를 막기 위한 최소한의 조치 필요

## 15.1.1 morgan과 express-session

- 현재 익스프레스 미들웨어 중 일부가 개발용으로 설정되어 있음
- 미들웨어들을 배포용으로 설정

```jsx
...
sequelize.sync({ force: false })
  .then(() => {
    console.log('데이터베이스 연결 성공');
  })
  .catch((err) => {
    console.error(err);
  });

if (process.env.NODE_ENV === 'production') { // 배포 환경인지 아닌지 판단하는 환경 변수
// NODE_ENV는 .env에 넣을 수 없음 : .env는 정적 파일이기 때문
  app.use(morgan('combined'));
} else {
  app.use(morgan('dev'));
}
app.use(express.static(path.join(__dirname, 'public')));
...
```

- express-session을 배포용으로 설정
- 단, express-session은 사용자의 환경에 따라 설정이 달라짐

```jsx
...
app.use(cookieParser(process.env.COOKIE_SECRET));
const sessionOption = {
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true, // 배포 환경일 때는 true로 변경
    secure: false,
  },
  store: new RedisStore({ client: redisClient }),
};
if (process.env.NODE_ENV === 'production') {
  sessionOption.proxy = true;
  // 배포 환경일 때는 true로 변경(https 적용을 위해 노드 서버 앞에 다른 서버를 뒀을 때)
  // sessionOption.cookie.secure = true;
}
app.use(session(sessionOption));
app.use(passport.initialize());
...
```

## 15.1.2 시퀄라이즈

- 데이터베이스도 배포 환경으로 설정
- 시퀄라이즈(sequelize)의 경우 수정이 필요
- 시퀄라이즈에서 가장 큰 문제
    - 비밀번호가 하드 코딩되어 있음
    - JSON 파일이므로 변수를 사용할 수 없음
- 시퀄라이즈는 JSON 대신 JS 파일을 설정 파일로 쓸 수 있게 지원

- config 폴더에서 config.json을 지우고 config.js를 생성

```jsx
require('dotenv').config(); // JS 파일이므로 dotenv 모듈을 사용할 수 있음

module.exports = {
  development: {
    username: 'root',
    password: process.env.SEQUELIZE_PASSWORD,
    database: 'nodebird',
    host: '127.0.0.1',
    dialect: 'mysql',
  },
  test: {
    username: "root",
    password: process.env.SEQUELIZE_PASSWORD,
    database: "nodebird_test",
    host: "127.0.0.1",
    dialect: "mysql"
  },
  production: {
    username: 'root',
    password: process.env.SEQUELIZE_PASSWORD,
    database: 'nodebird',
    host: '127.0.0.1',
    dialect: 'mysql',
    logging: false,
    // 현재 쿼리를 수행할 때마다 콘솔에 SQL문이 노출됨
    // 배포 환경에서는 어떤 쿼리가 수행되는지 숨기는 것이 좋음
    // production일 경우에는 logging에 false를 줘서 쿼리 명령어를 숨김
  },
};

// 보안 규칙에 따라 password 외에 나머지 정보도 process.env로 바꿀 수 있음
// username 속성이나 host 속성은 각각 아이디와 DB 서버 주소 역할을 하므로 숨기는 것이 좋음
// process.env가 development일 때는 development 속성의 설정 내용이 적용
// process.env가 production일 떄는 production 속성의 설정 내용이 적용
```

- 데이터베이스 비밀번호는 .env 파일에 입력

```tsx
COOKIE_SECRET=nodebirdsecret
KAKAO_ID=5d4daf57becfd72fd9c919882552c4a6
SEQUELIZE_PASSWORD=nodejsbook
```

## 15.1.3 cross-env

- **cross-env 패키지**를 사용하면 동적으로 process.env(환경 변수)를 변경할 수 있음
- 모든 운영체제에서 동일한 방법으로 환경 변수를 변경하는 것도 가능

- 기존 package.json을 수정

```json
{
  "name": "nodebird",
  "version": "0.0.1",
  "description": "익스프레스로 만드는 SNS 서비스",
  "main": "server.js",
  "scripts": { // 서버 실행을 위한 npm 스크립트를 두 개로 나눔
    "start": "NODE_ENV=production PORT=80 pm2 node server",
    // npm start : 배포 환경에서 사용하는 스크립트
    // NODE_ENV=production PORT=80 : 스크립트를 실행할 때 process.env를 동적으로 설정
    // process.env.NODE_ENV가 production이 되고, process.env.PORT가 80이 됨
    "dev": "nodemon server", // npm run dev : 개발 환경에서 사용하는 스크립트
    "test": "jest"
  },
 ...
```

- 하지만 이 방식은 리눅스나 맥에서는 되지만, 윈도에서는 process.env를 이렇게 설정할 수 없음
- 이럴 때 cross-env가 사용됨
- npm을 통해 설치 : `$ npm i cross-env`

- package-json을 다음과 같이 수정

```json
{
	...
  "scripts": {
    "start": "cross-env NODE_ENV=production PORT=80 pm2 node server",
    // 앞에 cross-env를 붙임으로서 윈도에서도 실행됨
    "dev": "nodemon server",
    "test": "jest"
  },
 ...
```

## 15.1.4 sanitize-html, csurf

- **sanitize-html**과 **csurf** 패키지 : 각각 XSS(Cross Site Scripting), CSRF(Cross Site Request Forgery) 공격을 막기 위한 패키지
    - XSS : 악의적인 사용자가 사이트에 스크립트를 삽입하는 공격
        - 악성 사용자가 게시글이나 댓글 등을 업로드할 때 자바스크립트가 포함된 태그를 올림
        - 나중에 다른 사용자가 그 게시글이나 댓글을 볼 때 해당 스크립트가 실행되어 예기치 못한 동작을 하게 됨
        - 서버에서는 사용자가 게시글을 업로드할 때 스크립트가 포함되어 있는지 검사해서, 존재한다면 제거해야 함
        - 공격성 스크립트의 유형이 많으므로 라이브러리의 도움을 받는 것이 좋음
            
            ```jsx
            const sanitizeHtml = require('sanitize-html');
            
            const html = "<script>location.href = ' https://gilbut.co.kr'</script> ";
            console.log(sanitizeHtml(html)); // ''
            // 사용자가 업로드한 HTML을 sanitize-html 함수로 감싸면 하용하지 않는 태그나 스크립트는 제거됨
            ```
            
    - CSRF : 사용자가 의도치 않게 공격자가 의도한 행동을 하게 만드는 공격
        - 특정 페이지에 방문할 때 저절로 로그아웃되거나 게시글이 써는 현상을 유도할 수 있음
        - 은행과 같은 사이트에서는 다른 사람에게 송금하는 행동을 넣는 등 상황에 따라 크게 악용될 수 있는 공격
        - 공격을 막으려면 내가 한 행동이 내가 한 것이 맞다는 사실을 인증해야 함
        - 이때 CSRF 토큰이 사용되고, csurf 패키지는 이 토큰을 쉽게 발급하거나 검증할 수 있도록 도움
            
            ```jsx
            const csrf = require('csurf');
            const csrfProtection = csrf({ cookie: true });
            
            app.get('/form', csrfProtection, (req, res) => {
            	res.render('csrf', { csrfToken: req. csrfToken() });
            }); // form을 렌더링하는 라우터
            
            app.post('/form', csrfProtection, (req, res) => {
            	res.send('ok');
            }); // form에서 보낸 데이터를 처리하는 라우터
            // 익스프레스의 미들웨어 형식으로 동작
            // form같은 것을 렌더링할 때 CSRF 토큰을 같이 제공
            // 현재 cookie를 사용하는 것으로 옵션을 설정했으므로 cookie-parser 패키지도 연결되어 있어야 함
            // 토큰은 req.csrfToken()으로 가져올 수 있음
            // 프런트엔드에 렌더링된 CSRF 토큰을 나중에 form을 제출할 때 데이터와 함께 제출
            ```
            

## 15.1.5 pm2

- **pm2** : 원활한 서버 운영을 위한 패키지
- 가장 큰 기능은 서버가 에러로 인해 꺼졌을 때 서버를 다시 켜주는 것
- 또 다른 중요한 기능은 멀티 프로세싱
    - 멀티 스레딩은 아니지만 멀티 프로세싱을 지원해 노드 프로세스 개수를 한 개 이상으로 늘릴 수 있음
    - 기본적으로는 CPU 코어를 하나만 사용하는데, pm2를 사용해서 프로세스를 여러 개 만들면 다른 코어들까지 사용할 수 있음
    - 클라이언트로부터 요청이 올 때 알아서 요청을 여러 노드 프로세스에 고르게 분배 → 하나의 프로세스가 받는 부하가 적어지므로 서비스를 더 원활하게 운영할 수 있음
    - 멀티 스레딩이 아니므로 서버의 메모리 같은 자원을 공유하지는 못함
        - 메모리를 공유하지 못해서 프로세스 간에 세션이 공유되지 않게 됨
        - 로그인 후 새로 고침을 반복할 때 세션 메모리가 있는 프로세스로 요청이 가면 로그인된 상태가 되고, 세션 메모리가 없는 프로세스로 요청이 가면 로그인되지 않은 상태가 되는 것
        - 문제를 극복하려면 세션을 공유할 수 있게 해주는 무언가가 필요 : 주로 **멤캐시드(Mencached)**나 **레디스(Redis)** 같은 서비스를 사용

- pm2 설치 : `$ npm i pm2`
- pm2는 nodemon처럼 콘솔에 입력하는 명령어
- package.json을 수정해서 nodemon 대신 pm2를 쓰도록 npm start 스크립트 수정

```jsx
{
  "name": "nodebird",
  "version": "0.0.1",
  "description": "익스프레스로 만드는 SNS 서비스",
  "main": "server.js",
  "scripts": {
    "start": "cross-env NODE_ENV=production PORT=80 pm2 start server.js",
    // start 스크립트에 node server 대신 pm2 start server.js를 입력
    "dev": "nodemon server",
    "test": "jest"
  },
 ...
```

- pm2를 실행했더니 node나 nodemon 명령어와는 다르게 노드 프로세스가 실행된 후 콘솔에 다른 명령어를 입력할 수 있음(pm2가 노드 프로세스를 백그라운드로 돌리므로 가능한 것)
- 백그라운드에서 돌고 있는 노드 프로세스를 확인하려면 `npx pm2 list` 명령어를 사용
    - npm start를 실행했을 때처럼 현재 프로세스 정보가 표시됨
        - 프로세스 아이디(pid), CPU와 메모리 사용량(mem) 등이 보여 편리
        - uptime과 status 사이에 재시작된 횟수가 나오는데, 0이 아니라면 서버가 재부팅된 적이 있다는 것을 의미 → 왜 재시작되었는지 확인 필요
        - npx pm2 logs로 로그를 확인할 수 있음
        - 에러 로그만 보고 싶다면 뒤에 `--err`을 붙이면 됨
        - 출력 수룰 바꾸고 싶다면 `--lines 숫자` 옵션 사용
    - pm2 프로세스를 종료하고 싶다면 콘솔에 `npx pm2 kill`을 입력
    - 서버를 재시작하고 싶다면 `npx pm2 reload all` 입력(다운타임이 거의 없이 서버가 재시작되어 좋음)

- 노드의 cluster 모듈처럼 클러스터링을 가능하게 하는 pm2의 클러스터링 모드를 사용

```jsx
{
  "name": "nodebird",
  "version": "0.0.1",
  "description": "익스프레스로 만드는 SNS 서비스",
  "main": "server.js",
  "scripts": {
    "start": "cross-env NODE_ENV=production PORT=80 pm2 start server.js -i 0",
    // i 뒤에 생성하길 원하는 프로세스 개수를 기입
    // 0 : 현재 CPU 코어 개수만큼 프로세스를 생성한다는 뜻
    // -1 : 프로세스를 CPU 코어 개수보다 한 개 덜 생성하겠다는 뜻
    "dev": "nodemon server",
    "test": "jest"
  },
 ...
```

- 현재 프로세스를 모니터링 : `$ npx pm2 monit`

## 15.1.6 winston

- 실제 서버를 운영할 때 console.log와 console.error를 대체하기 위한 모듈
- console.log와 console.error를 사용하면 개발 중에는 편리하게 서버의 상황을 파악할 수 있지만, 실제 배포 시에는 사용하기 어려움(console 객체의 메서드들이 언제 호출되었는지 파악하기 힘들 뿐만 아니라 서버가 종료되는 순간 로그들도 사라져 버리기 때문)
- 로그를 파일이나 다른 데이터베이스에 저장해야 함 → winston을 사용
- winston 설치 : `$ npm i winston`

- logger.js 작성

```jsx
const { createLogger, format, transports } = require('winston');

const logger = createLogger({ // createLogger 메서드로 logger 생성
// 인수로 logger에 대한 설정을 넣어줄 수 있음
  level: 'info', // 로그의 심각도(error가 가장 심각)
  // info를 고른 경우, info보다 심각한 단계의 로그(error, warn)도 함께 기록
  format: format.json(), // 로그의 형식
  transports: [ // 로그 저장 방식
    new transports.File({ filename: 'combined.log' }), // 파일로 저장
    new transports.File({ filename: 'error.log', level: 'error' }), // 파일 이름도 설정 가능
  ],
});

if (process.env.NODE_ENV !== 'production') {
  logger.add(new transports.Console({ format: format.simple() }));
}

module.exports = logger;
```

- logger 객체를 만들어 다른 파일에서 사용
- info, warn, error 등의 메서드를 사용하면 해당 심각도가 적용된 로그가 기록됨

```jsx
...
dotenv.config();
const pageRouter = require('./routes/page');
const authRouter = require('./routes/auth');
const postRouter = require('./routes/post');
const userRouter = require('./routes/user');
const { sequelize } = require('./models');
const passportConfig = require('./passport');
const logger = require('./logger');

const app = express();
...
app.use('/', pageRouter);
app.use('/auth', authRouter);
app.use('/post', postRouter);
app.use('/user', userRouter);

app.use((req, res, next) => {
  const error =  new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
  error.status = 404;
  logger.info('hello');
  logger.error(error.message);
  next(error);
});
```

- `npm run dev` 명령어로 개발용 서버를 실행한 후 http://localhost:8001/abcd에 접속
    - 없는 주소이므로 404 Not Found 에러 발생

## 15.1.7 helmet, hpp

- 서버의 각종 취약점들을 보완해주는 패키지들
- 익스프레스 미들웨어로서 사용 가능
- 이 패키지들을 사용한다고 해서 모든 취약점을 방어해주는 것은 아니므로 서버를 운영할 때는 주기적으로 취약점을 점검해야 함
- 개발 환경에서는 사용할 필요가 없으므로 배포 환경일 때만 적용

```jsx
...
const passport = require('passport');
const helmet = require('helmet');
const hpp = require('hpp');
...
if (process.env.NODE_ENV === 'production') {
  app.enable('trust proxy');
  app.use(morgan('combined'));
  app.use(
    helmet({
      contentSecurityPolicy: false,
      crossOriginEmbedderPolicy: false,
      crossOriginResourcePolicy: false,
    }),
  );
  app.use(hpp());
} else {
  app.use(morgan('dev'));
}
...
```

- 기본적으로 배포 전에 이 두 패키지를 넣어주는 것이 좋음
- 다만, 때로는 보안 규칙을 너무 엄격하게 적용해서 서비스가 제대로 돌아가지 않는 경우도 있으니, 공식 문서를 보면서 필요 없는 옵션은 해제해야 함

## 15.1.8 connect-redis

- 멀티 프로세스 간 세션 공유를 위해 레디스와 익스프레스를 연결해주는 패키지
- 기존에는 서버가 종료되어 메모리가 날아가면 접속자들의 로그인이 모두 풀려버림
- 이를 해결하기 위해 세션 아이디와 실제 사용자 정보를 데이터베이스에 저장하는데, 이때 사용하는 데이터베이스가레디스
- 다른 데이터베이스를 사용해도 되지만 주로 레디스를 많이 사용 → 메모리 기반의 데이터베이스라서 성능이 우수하기 때문

- connect-redis 패키지 설치 : `$ npm i redis connect-redis`
- 레디스를 사용하려면 connect-redis 패키지뿐만 아니라 레디스 데이터베이스를 설치해야 함
- 서버에 직접 설치할 수도 있지만, 레디스를 호스팅해주는 서비스를 쓰는 것이 편리 → redislabs

- 정보 화면에서 Public endpoint와 password를 확인해 .env에 붙여 넣음
    - Public endpoint에서 Host와 Port를 분리

```tsx
COOKIE_SECRET=nodebirdsecret
KAKAO_ID=5d4daf57becfd72fd9c919882552c4a6
SEQUELIZE_PASSWORD=nodejsbook
REDIS_HOST=redis-18954.c92.us-east-1-3.ec2.cloud.redislabs.com
REDIS_PORT=18954
REDIS_PASSWORD=JwTwGgKM4P0OFGStgQDgy2AcXvZjX4dc
```

```jsx
...
const hpp = require('hpp');
const redis = require('redis');
const RedisStore = require('connect-redis').default;

dotenv.config();
// dotenv.config()보다 코드가 아래에 있어야 한다는 점에 주의
// .env 파일에 적힌 process.env 객체의 값들은 dotenv.config() 이후에 생성
const redisClient = redis.createClient({ // redisClient 객체 생성
  url: `redis://${process.env.REDIS_HOST}:${process.env.REDIS_PORT}`,
  password: process.env.REDIS_PASSWORD, // 접속 정보 입력
});
redisClient.connect().catch(console.error);
const pageRouter = require('./routes/page');
...
const sessionOption = {
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
  },
  store: new RedisStore({ client: redisClient }),
};
...
// 세션 정보가 메모리 대신 레디스에 저장
// 로그인 후 서버를 껐다 켜도 로그인이 유지됨
// 실제 서비스에서 서버 업데이트 시 로그인이 풀리는 현상을 막을 수 있음
```

## 15.1.9 nvm, n

- 노드 버전을 업데이트하기 위한 패키지
- 윈도에서는 nvm-installer을 사용
- 리눅스나 맥에서는 n 패키지를 사용하면 편리

### 15.1.9.1 윈도

- 윈도에서는 [https://github.com/coreybutler/nvm-windows/releases 에](https://github.com/coreybutler/nvm-windows/releases에) 접속해 nvm-setup.zip을 내려받음
- 설치된 노드 버전을 확인하는 명령어 : $ nvm list
- 새로운 버전을 설치하고 싶다면 `nvm install [버전]` 입력
    - `nvm install 18.7.0`처럼 특정 버전을 입력하거나 `nvm install latest`처럼 최신 버전을 설치하게 하면 됨
- 설치된 버전을 사용하려면 `nvm use [버전명]` 입력

### 15.1.9.2 맥, 리눅스

- 맥과 리눅스에서는 n 패키지를 사용하면 편리 : $ sudo npm i -g n
- 콘솔에 n을 입력하면 현재 설치된 노드 버전을 볼 수 있음(나중에 설치된 버전을 선택할 때도 이 명령어 사용)
- 새로운 버전을 설치하고 싶다면 `n` 버전 입력
    - `n 18.7.0`처럼 특정 버전을 입력하거나 `n latest`처럼 최신 버전을 설치하게 하면 됨

### 15.1.9.3 업그레이드 후 npm 충돌 시

- 노드 버전을 업그레이드한 후 기존 npm 패키지들이 동작하지 않는 경우 `npm rebuild` 명령어로 해결 가능
- 만약 이 명령어로 해결되지 않으면 node_modules 폴더를 제거한 후 `npm i` 명령어로 패키지들을 다시 설치

# 15.2 깃과 깃허브 사용하기

- 소스 코드를 AWS와 GCP에 업로드하려고 할 때, 각 클라우드의 서버로 직접 파일과 폴더를 업로드할 수도 있지만, 실무에서는 대부분 그렇게 하지 않음
- 소스 코드가 수정될 때마다 직접 업로드하는 게 귀찮거니와 협업할 때도 서로 코드가 달라서 충돌이 발생하는 등의 문제가 많기 때문
- 그래서 깃(Git)이란느 분산형 버전 관리 시스템을 많이 사용
- 깃허브(Github) : 깃으로부터 업로드한 소스 코드를 서버에 저장할 수 있는 원격 저장소
    - 깃은 하나의 컴퓨터에서 코드를 관리하는 데 사용하지만, 깃허브에 소스 코드를 업로드하면 여러 사람이 코드를 공동으로 관리할 수 있음
    - 다른 컴퓨터를 사용하게 되었더라도 깃허브에서 코드만 내려받으면 됨
    - 깃허브의 경쟁 업체로는 깃랩(Gitlab)이나 빗버킷(BitBucket) 등이 있음

## 15.2.1 깃 설치하기

- 깃 설치 후, 제대로 설치되었는지 확인 : `$ git --version`
- 깃의 버전을 확인하는 명령어로, 버전이 제대로 표시된다면 설치에 성공한 것

- node_moduls, uploads 디렉터리는 자동으로 생성되므로 추가할 필요가 없음
- winston 로그도 굳이 깃을 통해 관리할 필요가 없음
- 따라서 이를 추가하지 않겠다고 깃에 알려야 하는데, 이때 .gitignore 파일이 사용됨
- NodeBird 프로젝트 폴더에 .gitignore 파일을 생성

```tsx
node_modules
uploads
*.log
coverage
```

- 깃에 추가하지 않을 폴더 또는 파일을 한 줄씩 적음

## 15.2.2 깃허브 사용하기

- 깃허브는 기업의 서비스이므로 사용하려면 회원 가입이 필요
- 회원 가입 후 리포지터리 생성
- 콘솔에서 NodeBird 프로젝트로 이동한 뒤 `git init` 명령어 입력(현재 디렉터리를 깃 관리 대상으로 지정하는 명령어)
- `git add .` : 모든 파일을 추가하겠다는 의미
    - 명령어를 실행하면 warning이 뜰 수도 있는데, 무시해도 됨
- `git config -m` : 누가 확정했는지 기록하기 위한 명령어
    - 사용자의 이메일 주소와 이름을 등록
    - -m 뒤의 문자열은 확정에 관한 설명 메시지
- `git remote add [별명] [주소]` : 깃에 깃허브 주소 등록
- `git push [별명] [브랜치]` : 깃허브에 업로드

# 15.3 AWS 시작하기

- 클라우드 서비스 중 가장 유명한 AWS에 배포
- AWS 웹 사이트에 접속해 회원 가입 절차 진행

# 15.4 AWS에 배포하기

1. 인스턴스 화면에서 Connect using SSH(SSH를 사용하여 연결) 버튼을 누름
2. SSH는 Lightsail 인스턴스와 연결되어 있음
3. 노드는 이미 설치되어 있으므로 MySQL을 설치하면 됨
4. mysql-server를 설치하는 중에 비밀번호 설정 화면이 나타나며, 여기서 비밀번호를 입력
    1. NodeBird 실습 시 사용한 MySQL 비밀번호를 입력하는 것이 좋음
5. Use Legacy Authentication Method 선택
6. MySQL에 접속해 정상적으로 설치되었는지 확인
7. git clone 명령어를 이용해 깃허브에 올렸던 소스 코드 내려받기 → node-deploy 폴더 생성
8. 서버를 실행하기 전에 Lightsail에서는 기본적으로 비트나미아파치(Bitnami apache) 서버가 켜져 있기 때문에 아파치 서버를 종료하는 명령어를 입력
9. 다시 node-deploy 폴더로 이동해 npm 패키지를 설치하고 서버를 실행

- 서버가 실행되지 않는다면 `sudo pm2 logs --err` 명령어를 입력해 어떤 에러가 발생했는지 확인 가능
- 에러 해결 후 `sudo pm2 reload all`로 서버를 재시작
- 퍼블릭 IP를 확인한 뒤, http://퍼블릭IP 로 접속
    - https로 접속하기 위해서는 도메인 구입과 인증서 발급 같은 별도 설정이 필요

# 15.5 GCP 시작하기

- 구글 클라우드 플랫폼은 구글 계정이 있어야 사용 가능
    - 기존 계정이 있다면 그대로 사용
    - 기존 계정이 없다면 구글 회원 가입 필요

# 15.6 GCP에 배포하기

1. SSH가 브라우저 새 창에서 실행되면 SSH에 명령어를 입력
2. 우분투에 노드와 MySQL 설치
3. 깃허브에 올렸던 소스 코드를 내려받음 → `git clone` 명령어 사용
4. node-deploy 폴더로 이동해 npm 패키지들을 설치하고 서버 실행(시퀄라이즈로 MySQL 데이터베이스도 생성)
5. 외부 IP를 확인한 뒤, http://외부IP로 접속
    1. https로 접속하기 위해서는 도메인 구입과 인증서 발급 같은 별도 설정 필요

- 서버가 실행되지 않는다면 `sudo npx pm2 logs --err` 명령어를 입력해 어떤 에러가 발생했는지 확인 가능
- 에러를 해결한 후 `sudo npx pm2 reload all`로 서버를 재시작