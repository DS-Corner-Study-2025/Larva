# 6장 : 익스프레스 웹 서버 만들기

- npm에는 서버를 제작하는 과정에서 겪게 되는 불편을 해소하고 편의 기능을 추가한 웹 서버 프레임워크가 존재
- 대표적인 것이 **익스프레스**
    - http 모듈의 요청과 응답 객체에 추가 기능들을 부여
    - 편리한 메서드들을 추가해 기능을 보완
    - 코드를 분리하기 쉽게 만들어 관리하기 용이
    - if문으로 요청 메서드와 주소를 구별하지 않아도 됨

# 6.1 익스프레스 프로젝트 시작하기

- 항상 package.json을 제일 먼저 생성
- `npm init` 명령어를 콘솔에서 호출해 단계적으로 내용물을 입력해도 되고, `npm init -y`를 입력해 파일을 만든 뒤 내용을 수정
- scripts 부분에 start 속성은 잊지 말고 넣어줘야 함
    - “start” : “nodemon app” → app.js를 nodemon으로 실행한다는 뜻(서버 코드에 수정 사항이 생길 때마다 매번 서버를 재시작하기 귀찮기 때문에 서버를 자동으로 재시작)
    - nodemon은 개발용으로만 사용할 것을 권장(배포 후에는 서버 코드가 빈번하게 변경될 일이 없으므로 nodemon을 사용하지 않아도 됨)

- 서버의 역할을 할 app.js 작성

```jsx
const express = require('express');

const app = express();
app.set('port', process.env.PORT || 3000);

app.get('/', (req, res) => {
	res.send('Hello, Express');
});
app.listen(app.get('port'), () => {
	console.log(app.get('port'), '번 포트에서 대기 중');
});
```

- Express 모듈을 실행해 app 변수에 할당, 익스프레스 내부에 http 모듈이 내장되어 있으므로 서버의 역할을 할 수 있음
- app.set(’port’, 포트)로 서버가 실행될 포트 설정
    - process.env 객체에 PORT 속성이 있다면 그 값을 사용, 없다면 기본값으로 3000번 포트를 이용하도록 설정
- app.get(주소, 라우터) : 주소에 대한 GET 요청이 올 때 어떤 동작을 할지 적는 부분
    - 매개변수 req : 요청에 관한 정보가 들어 있는 객체
    - 매개변수 res : 응답에 관한 정보가 들어 있는 객체
    - 익스프레스에서는 res.write나 res.end 대신 red.send를 사용
- GET 요청 외에도 POST, PUT, PATCH, DELETE, OPTIONS에 대한 라우터를 위한 메서드가 존재
- listen을 하는 부분은 http 웹 서버와 동일
- 단순한 문자열 대신 HTML로 응답하고 싶다면 res.sendFile 메서드 사용(단, 파일의 경로를 path 모듈을 사용해서 지정해야 함)

# 6.2 자주 사용하는 미들웨어

- 미들웨어 : 익스프레스의 핵심, 요청과 응답의 중간(middle)에 위치하기 때문에 미들웨어라고 부름
- 라우터와 에러 핸들러 또한 미들웨어의 일종
- 미들웨어는 요청과 응답을 조작해 기능을 추가하기도 하고, 나쁜 요청을 걸러내기도 함
- 미들웨어는 app.use와 함께 사용됨 → app.use(미들웨어)

```jsx
...
app.set('port', process.env.PORT || 3000);

app.use((req,res, next) => {
	console.log('모든 요청에 다 실행됩니다.');
	next();
});
app.get('/', (req, res, next) => {
	console.log('GET / 요청에서만 실행됩니다.');
	next();
}, (req, res) => {
	throw new Error('에러는 에러 처리 미들웨어로 갑니다');
});

app.use((err, req, res, next) => {
	console.error(err);
	res.status(500).send(err.message);
});

app.listen(app.get('port'), () => {
...
```

- app.use에 매개변수가 req, res, next인 함수를 넣음
- 미들웨어는 위에서부터 아래로 순서대로 실행되면서 요청과 응답 사이에 특별한 기능을 추가할 수 있음
- 매개변수 next : 다음 미들웨어로 넘어가는 함수(next를 실행하지 않으면 다음 미들웨어가 실행되지 않음)
- 주소를 첫 번째 인수로 넣어주지 않으면 미들웨어는 모든 요청에서 실행되고, 주소를 넣으면 해당하는 요청에서만 실행

| `app.use(미들웨어)` | 모든 요청에서 미들웨어 실행 |
| --- | --- |
| `app.use('/abc', 미들웨어)` | abc로 시작하는 요청에서 미들웨어 실행 |
| `app.post('/abc', 미들웨어)` | abc로 시작하는 POST 요청에서 미들웨어 실행 |
- app.use나 app.get 같은 라우터에 미들웨어를 여러 개 장착할 수 있음
    - 위의 코드에서는 app.get 미들웨어가 두 개 연결되어 있음
    - 이때도 next를 호출해야 다음 미들웨어로 넘어갈 수 있음
- 에러 처리 미들웨어는 모두 사용하지 않더라도 매개변수가 반드시 4개여야 함
    - 위의 코드 첫 번째 매개변수 err에는 에러가 관한 정보가 담겨 있음
    - res.status 메서드로 HTTP 상태 코드를 지정할 수 있음(기본값은 200, 즉 성공)
    - 에러 처리 미들웨어를 직접 연결하지 않아도 기본적으로 익스프레스가 에러를 처리하긴 하지만 실무에서는 직접 연결해주는 것이 좋음
    - 에러 처리 미들웨어는 특별한 경우가 아니면 가장 아래에 위치하도록 함

## 6.2.1 morgan

- 요청과 응답에 대한 정보를 콘솔에 기록
- `app.use(morgan('dev'));`와 같이 사용
    - 인수로 dev 외에 combined, common, short, tiny 등을 넣을 수 있음
    - dev 모드 기준으로 콘솔에 출력되는 GET / 500 7.409 ms - 50은 각각 [HTTP 메서드] [주소] [HTTP 상태 코드] [응답 속도] - [응답 바이트]를 의미

## 6.2.2 static

- 정적인 파일들을 제공하는 라우터 역할
- 기본적으로 제공되기에 따로 설치할 필요 없이 express 객체 안에서 꺼내 장착하면 됨
- `app.use('요청 경로', express.static('실제 경로'));` 와 같이 사용
    - 함수의 인수로 정적 파일들이 담겨 있는 폴더 지정
        - ex. public/stylesheets/style.css는 http://localjost:3000/stylesheets/style.css로 접근할 수 있음
        - 실제 서버의 폴더 경로에는 public이 들어 있지만 요청 주소에는 public이 들어 있지 않음 → 서버의 폴더 경로와 요청 경로가 다르므로 외부인이 서버의 구조를 쉽게 파악할 수 없어 **보안에 큰 도움**
- 정적 파일들을 알아서 제공해주므로 fs.readFile로 파일을 직접 읽어서 전송할 필요 없음
- 만약 요청 경로에 해당하는 파일이 없으면 알아서 내부적으로 next를 호출

## 6.2.3 body-parser

- 요청의 본문에 있는 데이터를 해석해서 req.body 객체로 만들어주는 미들웨어
- 보통 폼 데이터나 AJAX 요청의 데이터를 처리
- 멀티파트(이미지, 동영상, 파일) 데이터는 처리하지 못함 → multer 모듈 사용
- `app.use(express.join());` 또는 `app.use(express.urlencoded({extended: false}));`과 같이 사용
- 익스프레스 4.17.0 버전부터 body-parser 미들웨어의 기능이 익스프레스에 내장되었으므로 따로 설치할 필요 없음
- 익스프레스는 JSON과 URL-encoded 형식의 데이터 외에도 Raw, Text 형식의 데이터를 추가로 해석할 수 있음
    - Raw : 요청의 본문이 버퍼 데이터일 때 해석하는 미들웨어
    - Text : 요청의 본문이 텍스트 데이터일 때 해석하는 미들웨어
- 요청 데이터 종류
    - JSON : JSON 형식의 데이터 전달 방식
    - URL-encoded : 주소 형식으로 데이터를 보내는 방식(폼 전송할 때 주로 사용), urlencoded의 옵션이 false이면 노드의 querystring 모듈을 사용해 쿼리스트링을 해석하고 true이면 qs 모듈을 사용해 쿼리스트링을 해석(qs 모듈은 내장 모듈이 아니라 npm 패키지이며, querystring 모듈의 기능을 좀 더 확장한 모듈)
- POST와 PUT 요청의 본문을 전달받고자 할 때 패키지가 내부적으로 스트림을 처리해 req.body에 추가

## 6.2.4 cookie-parser

- 요청에 동봉된 쿠키를 해석해 req.cookies 객체로 생성
- `app.use(cookieParser(비밀 키));` 와 같이 사용
    - 첫 번째 인수로 비밀 키를 넣어줄 수 있음
    - 서명된 쿠키가 있는 경우, 제공한 비밀 키를 통해 해당 쿠키가 내 서버가 만든 쿠키임을 검증
    - 쿠키는 클라이언트에서 위조하기 쉬우므로 비밀 키를 통해 만들어낸 서명을 쿠키 값 뒤에 붙임
    - 서명이 붙으면 쿠키가 name=serocho.sjgn과 같은 모양이 되고, 서명된 쿠키는 req.signedCookies 객체에 들어 있음
- 해석된 쿠키들은 req.cookies 객체에 들어감
- 유효 기간이 지난 쿠키는 알아서 걸러냄
- 쿠키를 생성/제거하려면 `res.cookie`, `res.clearCookie` 메서드를 사용해야 함
- 쿠키를 지우려면 키와 값 외에 옵션도 정확히 일치해야 쿠키가 지워짐(expries, maxAge 옵션은 제외)
- signed 옵션을 true로 설정하면 쿠키 뒤에 서명이 붙음(내 서버가 쿠키를 만들었다는 것을 검증할 수 있으므로 대부분의 경우 서명 옵션을 켜두는 것이 좋음)
- 서명을 위한 비밀 키는 cookieParser 미들웨어에 인수로 넣은 process.env.COOKIE_SECRET이 됨

## 6.2.5 express-session

- 세션 관리용 미들웨어
- 로그인 등의 이유로 세션을 구현하거나 특정 사용자를 위한 데이터를 임시적으로 저장해둘 때 매우 유용
- 세션은 사용자별로 req.session 객체 안에 유지됨
- express-session은 인수로 세션에 대한 설정을 받음
    - `resave` : 요청이 올 때 세션에 수정 사항이 생기지 않더라도 세션을 다시 저장할지 설정
    - `saveUninitialized` : 세션에 저장할 내역이 없더라도 처음부터 세션을 생성할지 설정
- express-session은 세션 관리 시 클라이언트에 쿠키를 보냄
    - 쿠키를 안전하게 전송하려면 쿠키에 서명을 추가해야 하고, 쿠키를 서명하는 데 secret의 값이 필요
    - 세션 쿠키의 이름은 name 옵션으로 설정
    - 기본 이름은 connect.sid
- `cookie` 옵션은 세션 쿠키에 대한 설정
    - maxAge, domain, path, expires, sameSite, httpOnly, secure 등 일반적인 쿠키 옵션이 모두 제공됨
- 세션을 한 번에 삭제하려면 `req.session.destroy()` 메서드 호출

## 6.2.6 미들웨어의 특성 활용하기

## 6.2.7 multer

- 이미지, 동영상 등을 비롯한 여러 가지 파일을 멀티파트 형식으로 업로드할 때 사용하는 미들웨어
    - **멀티파트 방식** : enctype이 multipart/form-data인 폼을 통해 업로드하는 데이터의 형식
- 다음과 같은 multipart.html이 있다면 멀티파트 형식으로 데이터를 업로드할 수 있음

```html
<form action="/upload" method="post" enctype="multipart/form-data">
	<input type="file" name="image" />
	<input type="text" name="title" />
	<button type="submit">업로드</button>
</form>
```

- 폼을 통해 업로드하는 파일은 body-parser로는 처리할 수 없고 직접 파싱(해석)하기도 어려우므로 multer라는 미들웨어를 따로 사용하면 편리

```jsx
const multer = require('multer');

const upload = multer({
	storage: multer.disStorage({
		destination(req, file, done) {
			done(null, 'uploads/');
		},
		filename(req, file, done) {
			const ext = path.extname(file.originalname);
			done(null, path.basename(file.originalname, ext) + Date.now() + ext);
		},
	}),
	limits: {fileSize: 5 * 1024 * 1024},
});
```

- multer 함수의 인수로 설정을 넣음
- storage 속성에는 어디에(destination) 어떤 이름으로(filename) 저장할지 넣음
- file 객체에는 업로드한 파일에 대한 정보
- done 매개변수는 함수
    - 첫 번째 인수에는 에러가 있다면 에러를 넣음
    - 두 번째 인수에는 실제 경로나 파일 이름을 넣어줌
    - req나 file의 데이터를 가공해서 done으로 넘기는 형식
- limits 속성에는 업로드에 대한 제한 사항을 설정할 수 있음
- 위 설정을 실제로 활용하려면 서버에 uploads 폴더가 꼭 존재해야 함
    - 없다면 직접 만들어주거나 fs 모듈을 사용해서 서버를 시작할 때 생성

- 파일을 하나만 업로드하는 경우 single 미들웨어를 사용
    - single 미들웨어를 라우터 미들웨어 앞에 넣어두면 multer 설정에 따라 파일 업로드 후 req.file 객체가 생성됨
    - 인수는 input 태그의 name이나 폼 데이터의 키와 일치하게 넣음
    - 업로드 성공 시 결과는 req.file 객체 안에 들어있으며 req.body에는 파일이 아닌 데이터인 title이 들어 있음
- 여러 파일을 업로드하는 경우 HTML의 input 태그에는 multiple을 사용
    - 미들웨어는 single 대신 array로 교체
    - 업로드 결과도 req.file 대신 req.files 배열에 들어 있음
- 파일을 여러 개 업로드하지만 input 태그나 폼 데이터의 키가 다른 경우 fields 미들웨어 사용
- 파일을 업로드하지 않고도 멀티파트 형식으로 업로드하는 경우 none 미들웨어를 사용
    - 파일을 업로드하지 않았으므로 req.body만 존재

# 6.3 Router 객체로 라우팅 분리하기

- app.js에서 app.get 같은 메서드가 라우터 부분
- 라우터를 많이 연결하면 app.js 코드가 매우 길어지므로 익스프레스에서는 라우터를 분리할 수 있는 방법을 제공
- routes 폴더를 만들고 그 안에 .js를 작성

```jsx
const express = require('express');

const router = express.Router();

// GET / 라우터
router.get('/', (req, res) => {
	res.send('Hello, Express');
});

module.exports = router;
```

```jsx
const express = require('express');

const router = express.Router();

// GET /user 라우터
router.get('/', (req, res) => {
	res.send('Hello, User');
});

module.exports = router;
```

- 만들어진 .js들을 app.use를 통해 app.js에 연결 + 에러 처리 미들웨어 위에 404 상태 코드를 응답하는 미들웨어 하나 추가

```jsx
...
const path = require('path');

dotenv.config();
const indexRouter = require('./routes');
const userRouter = require('./routes/user');
...
	name: 'session-cookie',
})));

app.use('/', indexRouter);
app.use('/user', userRouter);

app.use((req, res, next) => {
	res.status(404).send('Not Found');
});

app.use((err, req, res, next) => {
...
```

- index. js와 user.js는 모양이 거의 비슷하지만 다른 주소의 라우터 역할을 하고 있음 → app.use로 연결할 때의 차이 때문
    - indexRouter는 app.use(’/’)에 연결, userRouter는 app.use(’/User’)에 연결
    - indexRouter는 use의 ‘/’와 get의 ‘/’가 합쳐져 GET / 라우터가, userRouter는 use의 ‘/user’와 get의 ‘/’가 합쳐져 GET /user 라우터가 됨
- next 함수에 다음 라우터로 넘어가는 기능 존재 → next(’route’)
    - 라우터에 연결된 나머지 미들웨어들을 건너뛰고 싶을 때 사용
- 라우트 매개변수

```jsx
router.get('/user/:id', (req, res) => { // :id에는 다른 값을 넣을 수 있음(/users/1 등)
	console.log(req.params, req.query);
});

// 라우트 매개변수 사용 시 주의점 : 일반 라우터보다 뒤에 위치해야 함
// 다양한 라우터를 아우르는 역할을 하므로 뒤에 위치해야 다른 라우터를 방해하지 않음
```

- 주소에 쿼리스트링을 쓸 때도 있음
    - 쿼리스트링의 키-값 정보는 req.query 객체 안에 존재
    - /users/123?limit=5&skip=10이라는 주소의 요청이 들어왔을 때 req.params와 req.query 객체는 각각 `{id: '123'}` `{limit: ‘5’, skip: '10'}`
- 웬만하면 404 응답 미들웨어와 에러 처리 미들웨어를 연결해주는 것이 좋음

# 6.4 req, res 객체 살펴보기

- 익스프레스의 req, res 객체는 http 모듈의 req, res 객체를 확장한 것
- 기존 http 모듈의 메서드도 사용할 수 있고, 익스프레스가 추가한 메서드나 속성을 사용할 수 있음
    - ex. res.writeHead, res.write, res.end 메서드를 그대로 사용할 수 있으면서 res.send나 res.sendFile 같은 메서드도 쓸 수 있음
    - 익스프레스의 메서드가 워낙 편리하므로 기존 http 모듈의 메서드는 잘 쓰이지 않음
- 자주 쓰이는 속성과 메서드(req 객체)
    - [req.app](http://req.app) : req 객체를 통해 app 객체에 접근 가능
    - req.body : body-parser 미들웨어가 만드는 요청의 본문을 해석한 객체
    - req.cookies : cookie-parser 미들웨어가 만드는 요청의 쿠키를 해석한 객체
    - req.ip : 요청의 ip 주소
    - req.params : 라우트 매개변수에 대한 정보가 담긴 객체
    - req.query : 쿼리스트링에 대한 정보가 담긴 객체
    - req.signedCookies : 서명된 쿠키들이 담겨 있음
    - req.get(헤더 이름) : 헤더의 값을 가져오고 싶을 때 사용하는 메서드
- 자주 쓰이는 속성과 메서드(res 객체)
    - [res.app](http://res.app) : req.app처럼 res 객체를 통해 app 객체에 접근할 수 있음
    - res.cookie(키, 값, 옵션) : 쿠키를 설정하는 메서드
    - res.clearCookie(키, 값, 옵션) : 쿠키를 제거하는 메서드
    - res.end() : 데이터 없이 응답을 보냄
    - res.json(JSON) : JSON 형식의 응답을 보냄
    - res.locals : 하나의 요청 안에서 미들웨어 간에 데이터를 전달하고 싶을 때 사용하는 객체
    - res.redirect(주소) : 리다이렉트할 주소와 함께 응답을 보냄
    - res.render(뷰, 데이터) : 템플릿 엔진을 렌더링해서 응답할 때 사용하는 메서드
    - res.send(데이터) : 데이터와 함께 응답을 보냄
    - res.sendFile(경로) : 경로에 위치한 파일을 응답
    - res.set(헤더, 값) : 응답의 헤더를 설정
    - res.status(코드) : 응답 시의 HTTP 상태 코드를 지정
- 각 메서드는 메서드 체이닝(method chaining)을 지원하는 경우가 많음 → 메서드 체이닝을 활용하면 코드 양을 줄일 수 있음

# 6.5 템플릿 엔진 사용하기

- 템플릿 엔진은 자바스크립트를 사용해서 HTML을 렌더링할 수 있게 함 → 기존 HTML과는 문법이 살짝 다를 수도 있고, 자바스크립트 문법이 들어 있기도 함

## 6.5.1 퍼그(제이드)

- 퍼그는 문법이 간단해서 코드양이 줄어들기 때문에 꾸준한 인기를 얻고 있음
- 퍼그 설치 : npm i pug
- 익스프레스와 연결하려면 app.js에 다음 부분이 들어있어야 함

```jsx
...
app.set('port', process.env.PORT || 3000);
app.set('views', path.join(__dirname, 'views'));
// views : 템플릿 파일들이 위치한 폴더들을 지정, res.render 메서드가 이 폴더 기준으로 템플릿 엔진을 찾아서 렌더링
app.set('view engine', 'pug');
// view engine : 어떠한 종류의 템플릿 엔진을 사용할지를 나타냄

app.use(morgan('dev'));
...
```

### 6.5.1.1 HTML 표현

- 기존 HTML과 다르게 화살괄호(<>)와 닫는 태그가 없음
- 탭 또는 스페이스로만 태그의 부모 자식 관계를 규명
    - 자식 태그는 부모 태그보다 들여쓰기되어 있어야 함
    - 들여쓰기에 오류가 있으면 제대로 렌더링되지 않음
- 태그의 속성을 표현할 때 태그명 뒤에 소괄호로 묶어 적음
- 속성 중 아이디와 클래스가 있는 경우에는 각각 `#`, `.`을 앞에 붙여서 표현
- HTML 텍스트는 속성 뒤에 한 칸을 띄고 입력
- 텍스트를 여러 줄 입력하고 싶은 경우 파이프(`|`) 삽입
- style이나 script 태그로 CSS 또는 자바스크립트 코드를 작성할 경우 태그 뒤에 `.`을 붙임

### 6.5.1.2 변수

- HTML과 다르게 자바스크립트 변수를 템플릿에 렌더링할 수 있음
- res.render을 호출할 때 보내는 변수를 퍼그가 처리
- res.render 메서드에 두 번재 인수로 변수 객체를 넣는 대신, res.locals 객체를 사용해서 변수를 넣을 수도 있음
    - 템플릿 엔진이 res.locals 객체를 읽어서 변수를 집어넣음
    - 현재 라우터뿐만 아니라 다른 미들웨어에서도 res.locals 객체에 접근할 수 있다는 장점
- 서버로부터 받은 변수는 다양한 방식으로 퍼그에서 사용 가능
    - 변수를 텍스트로 사용하고 싶은 경우 태그 뒤에 `=`을 붙인 후 변수를 입력
    - 속성에도 `=`을 붙인 후 변수를 사용할 수 있음
    - 텍스트 중간에 변수를 넣으려면 `#{변수}`를 사용
    - `#{}`의 내부와 `=` 기호 뒷부분은 자바스크립트로 해석하므로 자바스크립트 구문을 써도 됨
- 내부에 직접 변수를 선언할 수도 있음
    - 빼기(-)를 먼저 입력하면 뒤에 자바스크립트 구문을 작성할 수 있음
    - 여기에 변수를 선언하면 다음 줄부터 해당 변수를 사용할 수 있음
- 퍼그는 기본적으로 변수의 특수 문자를 HTML 엔티티로 이스케이프(문법과 관련 없는 문자로 바꾸는 행위), 이스케이프를 원하지 않는다면 = 대신 ≠ 사용

### 6.5.1.3 반복문

- HTML과 다르게 반복문도 사용할 수 있으며, 반복 가능한 변수인 경우에만 해당
- each로 반복문을 돌릴 수 있음
- each 대신 for을 사용할 수 있음
- 반복문 사용 시 인덱스도 가져올 수 있음

### 6.5.1.4 조건문

- 조건문으로 편리하게 분기 처리할 수 있으며 if, else if, else를 사용할 수 있음
- case 문도 가능

### 6.5.1.5 include

- 다른 퍼그나 HTMl 파일을 넣을 수 있음
- header나 footer, 내비게이션처럼 웹을 제작할 때 공통되는 부분을 따로 관리할 수 있어 페이지마다 동일한 HTML을 넣어야 하는 번거로움을 없앰
- include 파일 경로와 같은 형태로 사용

### 6.5.1.6 extends와 block

- 레이아웃을 정할 수 있으며, 공통되는 레이아웃 부분을 따로 관리할 수 있어 좋음
- include와도 함께 사용
- 레이아웃이 될 파일에는 공통된 마크업을 넣되, 페이지마다 달라지는 부분을 block으로 비워둠
- block은 여러 개 만들어도 되며, block [블록명]과 같은 형태로 block을 선언
    - block이 되는 파일에서는 extends 키워드로 레이아웃 파일을 지정하고 block 부분을 넣음
    - block 선언보다 한 단계 더 들여쓰기되어 있어야 함
    - 나중에 익스프레스에서 res.render(’body’)를 사용해 하나의 HTML로 합쳐 렌더링할 수 있음
    - 퍼그 확장자는 생략 가능하며 block 부분이 서로 합쳐짐

## 6.5.2 넌적스

- 넌적스는 퍼그의 HTML 문법 변화에 적응하기 힘든 분에게 유용한 템플릿 엔진
- 파이어폭스를 개발한 모질라에서 만듦
- HTML 문법을 그대로 사용하되 추가로 자바스크립트 문법을 사용할 수 있음
- 파이썬의 템플릿 엔진인 Twig와 문법이 상당히 유사
- 넌적스 설치 : npm i nunjucks

### 6.5.2.1 변수

- res.render 호출 시 보내는 변수를 넌적스가 처리
- 넌적스에서 변수는 {{ }}로 감쌈
- 내부에 변수를 사용할 수도 있음
    - 변수를 선언할 때는 `{% set 변수 = '값' %}`를 사용
- HTML을 이스케이프하고 싶지 않다면 `{{ 변수 | safe }}`를 사용

### 6.5.2.2 반복문

- 넌적스에서는 특수한 문을 {% %} 안에 쓰기 때문에 반복문도 이 안에 작성
- for in문과 endfor 사이에 위치
- 반복문에서 인덱스를 사용하고 싶다면 loop.index라는 특수한 변수를 사용할 수 있음

### 6.5.2.3 조건문

- 조건문은 `{% if 변수 %} {% elif %} {% else %} {% endif %}`로 구성
- case문은 없지만 elif를 통해 분기 처리할 수 있음

### 6.5.2.4 include

- 다른 HTMl 파일을 넣을 수 있음
- header나 footer, 내비게이션처럼 웹 제작 시 공통되는 부분을 따로 관리할 수 있어 페이지마다 동일한 HTML을 넣어야 하는 번거로움을 없앰
- include 파일 경로와 같은 형태로 사용

### 6.5.2.5 extends와 block

- 레이아웃을 정할 수 있으며, 공통되는 레이아웃 부분을 따로 관리할 수 있어 좋음
- include와도 함께 사용
- 레이아웃이 될 파일에는 공통된 마크업을 넣되, 페이지마다 달라지는 부분을 block으로 비워둠
- block은 여러 개 만들어도 됨
- block을 선언하는 방법 : `{% block [블록명] %}`
- `{% endblock%}`으로 블록 종료
- block이 되는 파일에서는 {% extends 경로 %} 키워드로 레이아웃 파일을 지정하고 block 부분을 넣음
- 나중에 익스프레스에서 res.render(’body’)를 사용해 하나의 HTML로 합친 후 렌더링할 수 있음
- 같은 이름의 block 부분이 서로 합쳐짐

## 6.5.3 에러 처리 미들웨어

- 에러 처리 미들웨어는 error라는 템플릿 파일을 렌더링
    - 렌더링 시 res.locals.message와 res.locals.error에 넣어준 값을 함께 렌더링
    - res.render에 변수를 대입하는 것 외에도, res.locals 속성에 값을 대입해 템플릿 엔진에 변수를 주입할 수 있음
- error 객체의 스택 트레이스는 시스템 환경이 production(배포 환경)이 아닌 경우에만 표시됨
    - 배포 환경인 경우에는 에러 메시지만 표시됨 → 에러 스택 트레이스가 노출되면 보안에 취약할 수 있기 때문