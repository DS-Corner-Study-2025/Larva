# 9장 : 웹 API 서버 만들기

- API 서버는 프런트엔드와 분리되어 운영되므로 모바일 서버로도 사용 가능
- 노드를 모바일 서버로 사용하려면 서버를 REST API 구조로 구성

# 10.1 API 서버 이해하기

- **API(Application Programming Interface)** : 다른 애플리케이션에서 현재 프로그램의 기능을 사용할 수 있게 허용하는 접점
- 웹 API는 다른 웹 서비스의 기능을 사용하거나 자원을 가져올 수 있는 창구
- 흔히 API를 ‘열었다’ 또는 ‘만들었다’고 표현하는데, 이는 다른 프로그램에서 현재 기능을 사용할 수 있게 허용했음을 의미
    - 다른 사람에게 정보를 제공하고 싶은 부분만 API를 열어놓고, 제공하고 싶지 않은 부분은 API를 만들지 않는 것
    - API를 열어놓았다 하더라도 모든 사람이 정보를 가져갈 수 있는 것이 아니라 인증된 사람만 일정 횟수 내에서 가져가게 제한을 둘 수도 있음
- **웹 API 서버** : 서버에 API를 올려서 URL을 통해 접근할 수 있게 만든 것
- **크롤링(crawling)** : 웹 사이트가 자체적으로 제공하는 API가 없거나 API 이용에 제한이 있을 때 사용하는 방법, 표면적으로 보이는 웹 사이트의 정보를 일정 주기로 수집해 자체적으로 가공하는 기술
    - 웹 사이트에서 직접 제공하는 API가 아니므로 원하는 정보를 얻지 못할 가능성
    - 웹 사이트에서 제공하길 원치 않는 정보를 수집한다면 법적인 문제가 발생할 수 있음
    - 웹 사이트가 어떤 페이지의 크롤링을 허용하는지 확인하려면 도메인/robots.txt에 접속

# 10.2 프로젝트 구조 갖추기

- 다른 서비스에 NodeBird 서비스의 게시글, 해시태그, 사용자 정보를 JSON 형식으로 제공
- 인증을 받은 사용자에게만 일정한 할당량 안에서 API를 호출할 수 있도록 허용
- nodebird-api 폴더를 만들고 package.json 파일을 생성
    - npm init으로 생성한 후 dependencies들을 설치하거나 package.json 입력

```json
{
  "name": "nodebird-api",
  "version": "0.0.1",
  "description": "NodeBird API 서버",
  "main": "app.js",
  "scripts": {
    "start": "nodemon app",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Zero Cho",
  "license": "ISC",
  "dependencies": {
    "bcrypt": "^5.0.0",
    "cookie-parser": "^1.4.5",
    "dotenv": "^16.0.1",
    "express": "^4.17.1",
    "express-session": "^1.17.1",
    "morgan": "^1.10.0",
    "mysql2": "^2.1.0",
    "nunjucks": "^3.2.1",
    "passport": "^0.6.0",
    "passport-kakao": "1.0.0",
    "passport-local": "^1.0.0",
    "sequelize": "^6.19.2",
    "uuid": "^8.3.2"
  },
  "devDependencies": {
    "nodemon": "^2.0.16"
  }
}
```

- package.json에 적힌 패키지 설치 : `$ npm i`

- 에러 표시 파일 생성 : views 폴더를 만들고 안에 error.html 파일 생성

```html
<h1>{{message}}</h1>
<h2>{{error.status}}</h2>
<pre>{{error.stack}}</pre>
```

```jsx
const express = require('express');
const path = require('path');
const cookieParser = require('cookie-parser');
const passport = require('passport');
const morgan = require('morgan');
const session = require('express-session');
const nunjucks = require('nunjucks');
const dotenv = require('dotenv');

dotenv.config();
const authRouter = require('./routes/auth');
const indexRouter = require('./routes');
const { sequelize } = require('./models');
const passportConfig = require('./passport');

const app = express();
passportConfig();
app.set('port', process.env.PORT || 8002);
app.set('view engine', 'html');
nunjucks.configure('views', {
  express: app,
  watch: true,
});
sequelize.sync({ force: false })
  .then(() => {
    console.log('데이터베이스 연결 성공');
  })
  .catch((err) => {
    console.error(err);
  });

app.use(morgan('dev'));
app.use(express.static(path.join(__dirname, 'public')));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser(process.env.COOKIE_SECRET));
app.use(session({
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
  },
}));
app.use(passport.initialize());
app.use(passport.session());

app.use('/auth', authRouter);
app.use('/', indexRouter);

app.use((req, res, next) => {
  const error = new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
  error.status = 404;
  next(error);
});

app.use((err, req, res, next) => {
  res.locals.message = err.message;
  res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
  res.status(err.status || 500);
  res.render('error');
});

app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기중');
});
```

- 포트 번호를 8002로 했으므로 9장의 NodeBird 앱 서버(8001번 포트) 및 추후에 만들 클라이언트인 NodeCat 서버(4000번 포트)와 같이 실행 가능
    - 콘솔을 하나 더 열어서 서버를 실행

- 도메인 모델을 추가
    - **도메인** : 인터넷 주소

```jsx
const Sequelize = require('sequelize');

class Domain extends Sequelize.Model {
  static initiate(sequelize) {
    Domain.init({
      host: {
        type: Sequelize.STRING(80),
        allowNull: false,
      },
      type: {
        type: Sequelize.ENUM('free', 'premium'), // type 컬럼은 ENUM 속성
        // 무료(free)나 프리미엄(premium) 둘 중 하나만 값으로 입력할 수 있게 함
        // 어겼을 때는 에러 발생
        allowNull: false,
      },
      clientSecret: {
        type: Sequelize.UUID, // UUID : 충돌 가능성이 매우 적은 랜덤한 문자열 
        allowNull: false,
      },
    }, {
      sequelize,
      timestamps: true,
      paranoid: true,
      modelName: 'Domain',
      tableName: 'domains',
    });
  }

  static associate(db) {
    db.Domain.belongsTo(db.User);
  }
};

module.exports = Domain;
```

- 도메인 모델에는 인터넷 주소(host)와 도메인 종류(type), 클라이언트비밀 키(client Secret)가 들어감
- 클라이언트 비밀 키는 다른 개발자들이 NodeBird의 API를 사용할 때 필요한 비밀 키
    - 키가 유출되면 다른 사람을 사칭해 요청을 보낼 수 있으므로, 유출되지 않도록 주의
- 도메인 모델은 사용자 모델과 일대다 관계를 가짐 → 사용자 한 명이 여러 도메인을 소유할 수도 있기 때문

```jsx
const Sequelize = require('sequelize');

class User extends Sequelize.Model {
  static initiate(sequelize) {
    User.init({
      email: {
        type: Sequelize.STRING(40),
        allowNull: true,
        unique: true,
      },
      nick: {
        type: Sequelize.STRING(15),
        allowNull: false,
      },
      password: {
        type: Sequelize.STRING(100),
        allowNull: true,
      },
      provider: {
        type: Sequelize.ENUM('local', 'kakao'),
        allowNull: false,
        defaultValue: 'local',
      },
      snsId: {
        type: Sequelize.STRING(30),
        allowNull: true,
      },
    }, {
      sequelize,
      timestamps: true,
      underscored: false,
      modelName: 'User',
      tableName: 'users',
      paranoid: true,
      charset: 'utf8',
      collate: 'utf8_general_ci',
    });
  }

  static associate(db) {
    db.User.hasMany(db.Post);
    db.User.belongsToMany(db.User, {
      foreignKey: 'followingId',
      as: 'Followers',
      through: 'Follow',
    });
    db.User.belongsToMany(db.User, {
      foreignKey: 'followerId',
      as: 'Followings',
      through: 'Follow',
    });
    db.User.hasMany(db.Domain);
  }
};

module.exports = User;
```

- 로그인 화면 구현
    - 카카오 로그인을 추가하려면 카카오 개발자 사이트에서 http://localhost:8002 도메인을 추가로 등록해야 함

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>API 서버 로그인</title>
    <style>
      .input-group label { width: 200px; display: inline-block; }
    </style>
  </head>
  <body>
    {% if user and user.id %}
      <span class="user-name">안녕하세요! {{user.nick}}님</span>
      <a href="/auth/logout">
        <button>로그아웃</button>
      </a>
      <fieldset>
        <legend>도메인 등록</legend>
        <form action="/domain" method="post">
          <div>
            <label for="type-free">무료</label>
            <input type="radio" id="type-free" name="type" value="free">
            <label for="type-premium">프리미엄</label>
            <input type="radio" id="type-premium" name="type" value="premium">
          </div>
          <div>
            <label for="host">도메인</label>
            <input type="text" id="host" name="host" placeholder="ex) zerocho.com">
          </div>
          <button>저장</button>
        </form>
      </fieldset>
      <table>
        <tr>
          <th>도메인 주소</th>
          <th>타입</th>
          <th>클라이언트 비밀키</th>
        </tr>
        {% for domain in domains %}
          <tr>
            <td>{{domain.host}}</td>
            <td>{{domain.type}}</td>
            <td>{{domain.clientSecret}}</td>
          </tr>
        {% endfor %}
      </table>
    {% else %}
      <form action="/auth/login" id="login-form" method="post">
        <h2>NodeBird 계정으로 로그인하세요.</h2>
        <div class="input-group">
          <label for="email">이메일</label>
          <input id="email" type="email" name="email" required autofocus>
        </div>
        <div class="input-group">
          <label for="password">비밀번호</label>
          <input id="password" type="password" name="password" required>
        </div>
        <div>회원가입은 localhost:8001에서 하세요.</div>
        <button id="login" type="submit">로그인</button>
      </form>
      <script>
        window.onload = () => {
          if (new URL(location.href).searchParams.get('error')) {
            alert(new URL(location.href).searchParams.get('error'));
          }
        };
      </script>
    {% endif %}
  </body>
</html>
```

- 도메인을 등록하는 화면도 포함되어 있음
    - 로그인하지 않았다면 로그인 창이 먼저 뜨고, 로그인한 사용자에게는 도메인 등록 화면을 보여줌

```jsx
const express = require('express');
const { renderLogin, createDomain } = require('../controllers');
const { isLoggedIn } = require('../middlewares');

const router = express.Router();

router.get('/', renderLogin);

router.post('/domain', isLoggedIn, createDomain);

module.exports = router;
```

```jsx
const { v4: uuidv4 } = require('uuid'); // 패키지의 변수나 함수를 불러올 때 이름을 변경
const { User, Domain } = require('../models');

exports.renderLogin = async (req, res, next) => {
  try {
    const user = await User.findOne({
      where: { id: req.user?.id || null }, 
      // where에는 undefined가 들어가면 안 되므로 req.user?.id || null을 사용
      include: { model: Domain },
    });
    res.render('login', {
      user,
      domains: user?.Domains,
    });
  } catch (err) {
    console.error(err);
    next(err);
  }
}

exports.createDomain = async (req, res, next) => {
  try {
    await Domain.create({
      UserId: req.user.id,
      host: req.body.host,
      type: req.body.type,
      clientSecret: uuidv4(), // uuid 중에서 4 버전 사용(36자리 문자열 형식)
    });
    res.redirect('/');
  } catch (err) {
    console.error(err);
    next(err);
  }
};
```

- `GET /`는 접속 시 로그인 화면을 보여줌
- 도메인 등록 라우터는 폼으로부터 온 데이터를 도메인 모델에 저장

- 서버를 실행하고 http://localhost:8002로 접속
    - API를 사용하려면 허가를 받아야 함
    - 로그인 후에는 도메인 등록 화면이 나타남 : 등록한 도메인에서만 API를 사용할 수 있게 하기 위해서
    - 웹 브라우저에서 요청을 보낼 때, 응답을 하는 곳과 도메인이 다르면 CORS(Cross-Origin Resource Sharing) 에러가 발생할 수 있음
    - 브라우저가 현재 웹 사이트에서 함부로 다른  서버에 접근하는 것을 막는 조치
- 발급받은 비밀 키는 [localhost:4000](http://localhost:4000) 서비스에서 NodeBird API를 호출할 때 인증 용도로 사용
    - 비밀 키가 유출되면 다른 사람이 마치 사용자가 호출한 것 마냥 API를 사용할 수 있으므로 조심해야 함

# 10.3 JWT 토큰으로 인증하기

- **JWT(JSON Web Token)** : JSON 형식의 데이터를 저장하는 토큰
- JWT의 구성
    - **헤더(HEADER)** : 토큰  종류와 해시 알고리즘 정보가 들어 있음
    - **페이로드(PAYLOAD)** : 토큰의 내용물이 인코딩된 부분
    - **시그니처(SIGNATURE)** : 일련의 문자열로, 시그니처를 통해 토큰이 변조되었는지 여부를 확인
- JWT에는 내용을 볼 수 있기 때문에 민감한 내용을 넣으면 안됨
- 내용이 노출되는 토큰을 사용하는 이유 : 내용이 들어 있기 떄문
    - 랜덤한 토큰을 받으면 토큰의 주인이 누구인지, 그 사람의 권한은 무엇인지 각 요청마다 체크
    - 이러한 작업은 보통 데이터베이스를 조회해야 하는 복잡한 작업인 경우가 많음
    - JWT 토큰은 JWT 비밀 키를 알지 않는 이상 변조가 불가능
    - 변조한 토큰은 시그니처를 비밀 키를 통해 검사할 때 들통남 → 변조할 수 없으므로 내용물이 바뀌지 않았는지 걱정할 필요 없음
    - 이메일이나 사용자의 권한 등을 넣어두면 데이터베이스 조회 없이도 그 사용자를 믿고 권한 부여 가능
- JWT 토큰의 단점 : 용량이 크다는 것
    - 내용물이 들어 있으므로 랜덤한 토큰을 사용할 때보다 용량이 클 수밖에 없음
    - 요청 때마다 토큰이 오가서 데이터양이 증가
- 랜덤 문자열을 사용해서 매번 사용자 정보를 조회하는 작업의 비용이 더 큰지, 내용물이 들어 있는 JWT 토큰을 사용해서 발생하는 데이터 비용이 더 큰지 비교해서 사용

- 웹 서버에 JWT 토큰 인증 과정을 구현
    - JWT 모듈 설치 : `$ npm i jsonwebtoken`
    - JWT를 사용해서 본격적으로 API를 만들기 : 다른 사용자가 API를 쓰려면 JWT 토큰을 발급받고 인증을 받아야 함 → 대부분의 라우터에 공통되므로 미들웨어에 만들어두는 게 좋음

```tsx
COOKIE_SECRET=nodebirdsecret
KAKAO_ID=5d4daf57becfd72fd9c919882552c4a6
JWT_SECRET=jwtSecret
```

```jsx
exports.isLoggedIn = (req, res, next) => {
  if (req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send('로그인 필요');
  }
};

exports.isNotLoggedIn = (req, res, next) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    const message = encodeURIComponent('로그인한 상태입니다.');
    res.redirect(`/?error=${message}`);
  }
};

exports.verifyToken = (req, res, next) => {
  try {
    res.locals.decoded = jwt.verify(req.headers.authorization, process.env.JWT_SECRET);
    // 토큰 검증(첫 번째 인수로 토큰을, 두 번째 인수로 토큰의 비밀 키 사용)
    return next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') { // 유효기간 초과
      return res.status(419).json({
        code: 419,
        message: '토큰이 만료되었습니다',
      });
    }
    return res.status(401).json({
      code: 401,
      message: '유효하지 않은 토큰입니다',
    });
  }
};
```

- 토큰의 비밀 키가 일치하지 않는다면 인증을 받을 수 없음 → 에러가 발생해 catch문으로 이동
- 올바른 토큰이더라도 유효 기간이 지난 경우 역시 catch문으로 이동
- 유효 기간 만료 시 419 상태 코드를 응답하는데, 코드는 400번대 숫자 중에서 마음대로 선택 가능
- 인증에 성공한 경우에는 토큰의 내용이 반환되어 res.locals.decoded에 저장됨
    - 토큰의 내용 : 사용자 아이디, 닉네임, 발급자, 유효 기간 등

 

```jsx
const express = require('express');

const { verifyToken } = require('../middlewares');
const { createToken, tokenTest } = require('../controllers/v1');

const router = express.Router();

// POST /v1/token
router.post('/token', createToken);

// POST /v1/test
router.get('/test', verifyToken, tokenTest);

module.exports = router;
```

```jsx
const jwt = require('jsonwebtoken');
const { Domain, User } = require('../models');

exports.createToken = async (req, res) => {
  const { clientSecret } = req.body;
  try {
    const domain = await Domain.findOne({
      where: { clientSecret },
      include: {
        model: User,
        attribute: ['nick', 'id'],
      },
    });
    if (!domain) {
      return res.status(401).json({
        code: 401,
        message: '등록되지 않은 도메인입니다. 먼저 도메인을 등록하세요',
      });
    }
    const token = jwt.sign({ // 첫 번재 인수 : 토큰의 내용(사용자의 아이디와 닉네임을 넣음)
      id: domain.User.id,
      nick: domain.User.nick,
    }, process.env.JWT_SECRET, { // 두 번째 인수 : 토큰의 비밀 키(유출되지 않게 조심)
      expiresIn: '1m', // 세 번째 인수 : 토큰의 설정(유효 기간 1분, 발급자 nodebird)
      issuer: 'nodebird',
    });
    return res.json({
      code: 200,
      message: '토큰이 발급되었습니다',
      token,
    });
  } catch (error) {
    console.error(error);
    return res.status(500).json({
      code: 500,
      message: '서버 에러',
    });
  }
};

exports.tokenTest = (req, res) => {
  res.json(res.locals.decoded);
};
```

- 라우터의 이름은 v1으로, 버전 1이라는 뜻
- 라우터에 버전을 붙인 이유는 한번 버전이 정해진 후에는 라우터를 함부로 수정하면 안 되기 때문
    - 다른 사람이나 서비스가 기존 API를 쓰고 있다는 사실을 항상 염두에 둬야 함
    - API 서버의 코드를 바꾸면 API를 사용 중인 다른 사람에게 영향을 미침
    - 특히 기존에 있던 라우터가 수정되는 순간 API를 사용하는 프로그램들이 오작동할 수 있음
    - 기존 사용자에게 영향을 미칠 정도로 수정해야 한다면 버전을 올린 라우터를 새로 추가하고 이전 API를 쓰는 사람들에게는 새로운 API가 나왔음을 알리는 것이 좋음
    - 이전 API를 없앨 때도 어느 정도 기간을 두고 미리 공지해서 사람들이 다음 API로 충분히 넘어갔을 때 없애는 것이 좋음
- GET /v1/test 라우터 : 사용자가 발급받은 토큰을 테스트해볼 수 있는 라우터
    - 토큰을 검증하는 미들웨어를 거친 후, 검증이 성공했다면 토큰의 내용물을 응답으로 보냄
- 라우터의 응답을 살펴보면 모두 일정한 형식을 갖추고 있음
    - JSON 형태에 code, message 속성이 존재
    - 토큰이 있는 경우 token 속성도 존재
    - 일정한 형식을 갖춰야 응답받는 쪽에서 처리하기 좋음
    - code는 HTTP 상태 코드를 사용해도 되고, 임의로 숫자를 부여해도 됨(일관성만 있다면 문제 없음)
    - code가 200번대 숫자가 아니면 에러이고, 에러의 내용은 message에 담아 보내는 것으로 현재 API 서버의 규칙 설정
- 라우터를 서버에 연결

```jsx
...
dotenv.config();

const v1 = require('./routes/v1');
const authRouter = require('./routes/auth');
...

app.use(passport.session());

app.use('/v1', v1);
app.use('/auth', authRouter);
...
```

# 10.4 다른 서비스에서 호출하기

- API를 사용하는 서비스 만들기
- nodebird-api 폴더와 같은 위치에 nodecat이라는 새로운 폴더 생성
    - 별도의 서버이므로 nodebird-api와 코드가 섞이지 않게 주의

```json
{
  "name": "nodecat",
  "version": "0.0.1",
  "description": "NodeBird 2차 서비스",
  "main": "app.js",
  "scripts": {
    "start": "nodemon app"
  },
  "author": "Zero Cho",
  "license": "ISC",
  "dependencies": {
    "axios": "^0.27.2",
    "cookie-parser": "^1.4.6",
    "dotenv": "^16.0.1",
    "express": "^4.18.1",
    "express-session": "^1.17.3",
    "morgan": "^1.10.0",
    "nunjucks": "^3.2.3"
  },
  "devDependencies": {
    "nodemon": "^2.0.16"
  }
}
```

- `$ npm i`
- 서버의 주목적 : nodebird-api를 통해 데이터를 가져오는 것
- 가져온 데이터는 JSON 형태이므로 퍼그나 넌적스 같은 템플릿 엔진으로 데이터를 렌더링할 수 있음
- 서버 파일과 에러를 표시할 파일 생성

```jsx
const express = require('express');
const morgan = require('morgan');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const nunjucks = require('nunjucks');
const dotenv = require('dotenv');

dotenv.config();
const indexRouter = require('./routes');

const app = express();
app.set('port', process.env.PORT || 4000);
app.set('view engine', 'html');
nunjucks.configure('views', {
  express: app,
  watch: true,
});

app.use(morgan('dev'));
app.use(cookieParser(process.env.COOKIE_SECRET));
app.use(session({
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
  },
}));

app.use('/', indexRouter);

app.use((req, res, next) => {
  const error = new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
  error.status = 404;
  next(error);
});

app.use((err, req, res, next) => {
  res.locals.message = err.message;
  res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
  res.status(err.status || 500);
  res.render('error');
});

app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기중');
});
```

```html
<h1>{{message}}</h1>
<h2>{{error.status}}</h2>
<pre>{{error.stack}}</pre>
```

```tsx
COOKIE_SECRET=nodecat
CLIENT_SECRET=78113d93-5da0-4401-a570-d21a86d0851a
```

```jsx
const express = require('express');
const { test } = require('../controllers');

const router = express.Router();

// POST /test
router.get('/test', test);

module.exports = router;
```

```jsx
const axios = require('axios');

exports.test = async (req, res, next) => { // 토큰 테스트 라우터
  try {
    if (!req.session.jwt) { // 세션에 토큰이 없으면 토큰 발급 시도
      const tokenResult = await axios.post('http://localhost:8002/v1/token', {
        clientSecret: process.env.CLIENT_SECRET,
      });
      if (tokenResult.data?.code === 200) { // 토큰 발급 성공
        req.session.jwt = tokenResult.data.token; // 세션에 토큰 저장
      } else { // 토큰 발급 실패
        return res.json(tokenResult.data); // 발급 실패 사유 응답
      }
    }
    // 발급받은 토큰 테스트
    const result = await axios.get('http://localhost:8002/v1/test', {
      headers: { authorization: req.session.jwt },
    });
    return res.json(result.data);
  } catch (error) {
    console.error(error);
    if (error.response?.status === 419) { // 토큰 만료 시
      return res.json(error.response.data);
    }
    return next(error);
  }
};
```

- **GET /test 라우터** : NodeCat 서비스가 토큰 인증 과정을 테스트해보는 라우터
    - 요청이 왔을 때 세션에 발급받은 토큰이 저장되어 있지 않다면 POST http://localhost:8002/v1/token 라우터로부터 토큰을 발급받음

# 10.5 SNS API 서버 만들기

```jsx
const express = require('express');

const { verifyToken } = require('../middlewares');
const { createToken, tokenTest, getMyPosts, getPostsByHashtag } = require('../controllers/v1');

const router = express.Router();

// POST /v1/token
router.post('/token', createToken);

// POST /v1/test
router.get('/test', verifyToken, tokenTest);

// GET /v1/posts/my
router.get('/posts/my', verifyToken, getMyPosts);

// GET /v1/posts/hashtag/:title
router.get('/posts/hashtag/:title', verifyToken, getPostsByHashtag);

module.exports = router;
```

```jsx
const jwt = require('jsonwebtoken');
const { Domain, User, Post, Hashtag } = require('../models');
...
exports.tokenTest = (req, res) => {
  res.json(res.locals.decoded);
};

exports.getMyPosts = (req, res) => {
  Post.findAll({ where: { userId: res.locals.decoded.id } })
    .then((posts) => {
      console.log(posts);
      res.json({
        code: 200,
        payload: posts,
      });
    })
    .catch((error) => {
      console.error(error);
      return res.status(500).json({
        code: 500,
        message: '서버 에러',
      });
    });
};

exports.getPostsByHashtag = async (req, res) => {
  try {
    const hashtag = await Hashtag.findOne({ where: { title: req.params.title } });
    if (!hashtag) {
      return res.status(404).json({
        code: 404,
        message: '검색 결과가 없습니다',
      });
    }
    const posts = await hashtag.getPosts();
    return res.json({
      code: 200,
      payload: posts,
    });
  } catch (error) {
    console.error(error);
    return res.status(500).json({
      code: 500,
      message: '서버 에러',
    });
  }
};
```

- GET /posts/my 라우터와 GET /posts/hashtag/:title 라우터를 추가 : 내가 올린 포스트와 해시태그 검색 결과를 가져오는 라우터
- 사용하는 측(NodeCat)에서 위의 API를 이용하는 코드를 추가(토큰을 발급받는 부분이 반복되므로 이를 함수로 만들어 재사용하는 것이 좋음)

```jsx
const express = require('express');
const { searchByHashtag, getMyPosts } = require('../controllers');

const router = express.Router();

router.get('/myposts', getMyPosts);

router.get('/search/:hashtag', searchByHashtag);

module.exports = router;
```

```jsx
...
API_URL=http://localhost:8002/v1
ORIGIN=http://localhost:4000
```

```jsx
const axios = require('axios');

const URL = process.env.API_URL;
axios.defaults.headers.origin = process.env.ORIGIN; // origin 헤더 추가

const request = async (req, api) => { // NodeBird API에 요청을 보내는 함수
  try {
    if (!req.session.jwt) { // 세션에 토큰이 없으면
      const tokenResult = await axios.post(`${URL}/token`, {
        clientSecret: process.env.CLIENT_SECRET,
      });
      req.session.jwt = tokenResult.data.token; // 세션에 토큰 저장
    }
    return await axios.get(`${URL}${api}`, {
      headers: { authorization: req.session.jwt },
    }); // API 요청
  } catch (error) {
    if (error.response?.status === 419) { // 토큰 만료시 토큰 재발급 받기
      delete req.session.jwt;
      return request(req, api);
    } // 419 외의 다른 에러면
    throw error;
  }
};

exports.getMyPosts = async (req, res, next) => { // API를 통해 자신이 작성한 포스트를 JSON 형식으로 가져오는 라우터
  try {
    const result = await request(req, '/posts/my');
    res.json(result.data);
  } catch (error) {
    console.error(error);
    next(error);
  }
};

exports.searchByHashtag = async (req, res, next) => { // API를 이용해 해시태그를 검색
  try {
    const result = await request(
      req, `/posts/hashtag/${encodeURIComponent(req.params.hashtag)}`,
    );
    res.json(result.data);
  } catch (error) {
    if (error.code) {
      console.error(error);
      next(error);
    }
  }
};
```

# 10.6 사용량 제한 구현하기

- 인증된 사용자라고 해도 API를 과도하게 사용하면 API 서버에 무리가 감
- 일정 기간 내에 API를 사용할 수 있는 횟수를 제한해 서버의 트래픽을 줄이는 것이 좋음
- 이러한 기능 또한 npm에 패키지로 만들어져 있음 → **express-rate-limit** 패키지
- 패키지 설치 : `$ npm i express-rate-limit`

- verifyToken 미들웨어 아래에 apiLimiter 미들웨어와 deprecated 미들웨어 추가

```jsx
const jwt = require('jsonwebtoken');
const rateLimit = require('express-rate-limit');

exports.isLoggedIn = (req, res, next) => {
  if (req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send('로그인 필요');
  }
};

exports.isNotLoggedIn = (req, res, next) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    const message = encodeURIComponent('로그인한 상태입니다.');
    res.redirect(`/?error=${message}`);
  }
};

exports.verifyToken = (req, res, next) => {
  try {
    res.locals.decoded = jwt.verify(req.headers.authorization, process.env.JWT_SECRET);
    return next();
  } catch (error) {
    if (error.name === 'TokenExpiredError') { // 유효기간 초과
      return res.status(419).json({
        code: 419,
        message: '토큰이 만료되었습니다',
      });
    }
    return res.status(401).json({
      code: 401,
      message: '유효하지 않은 토큰입니다',
    });
  }
};

exports.apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1분
  max: 10,
  handler(req, res) {
    res.status(this.statusCode).json({
      code: this.statusCode, // 기본값 429
      message: '1분에 열 번만 요청할 수 있습니다.',
    });
  },
});

exports.deprecated = (req, res) => {
  res.status(410).json({
    code: 410,
    message: '새로운 버전이 나왔습니다. 새로운 버전을 사용하세요.',
  });
};
```

- apiLimiter 미들웨어를 라우터에 넣으면 라우터에 사용량 제한이 걸림
- 미들웨어의 옵션으로는 windowMs(기준 시간), max(허용 횟수), handler(제한 초과 시 콜백 함수) 등이 존재
- 현재 설정은 1분에 한 번 호출 가능하게 되어 있으며, 사용량 제한을 초과하면 429 상태 코드와 함께 허용량을 초과했다는 응답 전송
- deprecated 미들웨어는 사용하면 안 되는 라우터에 붙여줌

| 응답 코드 | 메시지 |
| --- | --- |
| 200 | JSON 데이터입니다. |
| 401 | 유효하지 않은 토큰입니다. |
| 410 | 새로운 버전이 나왔습니다. 새로운 버전을 사용하세요. |
| 419 | 토큰이 만료되었습니다. |
| 429 | 1분에 한 번만 요청할 수 있습니다. |
| 500- | 기타 서버 에러 |
- 사용량 제한이 추가되었으므로 기존 API 버전과 호환되지 않음 → 새로운 v2 라우터를 만들기
- 기본적으로는 v1과 동일

```jsx
const express = require('express');

const { verifyToken, apiLimiter } = require('../middlewares');
const { createToken, tokenTest, getMyPosts, getPostsByHashtag } = require('../controllers/v2');

const router = express.Router();

// POST /v2/token
router.post('/token', apiLimiter, createToken);

// POST /v2/test
router.get('/test', apiLimiter, verifyToken, tokenTest);

// GET /v2/posts/my
router.get('/posts/my', apiLimiter, verifyToken, getMyPosts);

// GET /v2/posts/hashtag/:title
router.get('/posts/hashtag/:title', apiLimiter, verifyToken, getPostsByHashtag);

module.exports = router;
```

```jsx
...
exports.createToken = async (req, res) => {
  const { clientSecret } = req.body;
  try {
    const domain = await Domain.findOne({
      where: { clientSecret },
      include: {
        model: User,
        attribute: ['nick', 'id'],
      },
    });
    if (!domain) {
      return res.status(401).json({
        code: 401,
        message: '등록되지 않은 도메인입니다. 먼저 도메인을 등록하세요',
      });
    }
    const token = jwt.sign({
      id: domain.User.id,
      nick: domain.User.nick,
    }, process.env.JWT_SECRET, {
      expiresIn: '30m', // 30분
      issuer: 'nodebird',
    });
    return res.json({
      code: 200,
      message: '토큰이 발급되었습니다',
      token,
    });
  } catch (error) {
    console.error(error);
    return res.status(500).json({
      code: 500,
      message: '서버 에러',
    });
  }
};

...
```

- 토큰 유효 기간을 30분으로 늘렸고, 라우터에 사용량 제한 미들웨어를 추가
- 기존 v1 라우터를 사용할 때는 경고 메시지를 띄워줌

```jsx
const express = require('express');

const { verifyToken, deprecated } = require('../middlewares');
const { createToken, tokenTest, getMyPosts, getPostsByHashtag } = require('../controllers/v1');

const router = express.Router();

router.use(deprecated);

// POST /v1/token
router.post('/token', createToken);
...
```

- 새로 만든 라우터를 서버와 연결

```jsx
...
const v1 = require('./routes/v1');
const v2 = require('./routes/v2');
const authRouter = require('./routes/auth');
...
app.use('/v1', v1);
app.use('/v2', v2);
app.use('/auth', authRouter);
...
```

- 사용자 입장(NodeCat)으로 돌아와서 새로 생긴 버전을 호출(버전만 v1에서 v2로 변경)

```tsx
...
API_URL=http://localhost:8002/v2
...
```

- 만약 v2로 바꾸지 않고 v1을 계속 사용한다면 410 에러 발생
- 1분에 한 번보다 더 많이 API를 호출하면 429 에러 발생

- 사용량을 1분에 열 번으로 수정

```tsx
...
exports.apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1분
  max: 10,
  handler(req, res) {
    res.status(this.statusCode).json({
      code: this.statusCode, // 기본값 429
      message: '1분에 열 번만 요청할 수 있습니다.',
    });
  },
});
...
```

# 10.7 CORS 이해하기

- NodeCat의 프런트에서 nodebird-api의 서버 API를 호출
- routes/index.js에 프런트 화면을 렌더링하는 라우터 추가

```tsx
const express = require('express');
const { searchByHashtag, getMyPosts, renderMain } = require('../controllers');

const router = express.Router();

router.get('/myposts', getMyPosts);

router.get('/search/:hashtag', searchByHashtag);

router.get('/', renderMain);

module.exports = router;
```

```tsx
...
exports.renderMain = (req, res) => {
  res.render('main', { key: process.env.CLIENT_SECRET });
};
```

```html
<!DOCTYPE html>
<html>
  <head>
    <title>프론트 API 요청</title>
  </head>
  <body>
  <div id="result"></div>
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  <script>
    axios.post('http://localhost:8002/v2/token', {
      clientSecret: '{{key}}',
    })
      .then((res) => {
        document.querySelector('#result').textContent = JSON.stringify(res.data);
      })
      .catch((err) => {
        console.error(err);
      });
  </script>
  </body>
</html>
```

- `clientSecret`의 `{{key}}` 부분이 넌적스에 의해 실제 키로 치환돼서 렌더링됨
- 실제 서비스에서는 서버에서 사용하는 비밀 키와 프런트에서 사용하는 비밀 키를 따로 두는 게 좋음 → 보통 서버에서 사용하는 비밀 키가 더 강력하기 때문
- 프런트에서 사용하는 비밀 키는 모든 사람에게 노출된다는 단점도 존재 → 데이터베이스에서 clientSecret 외에 frontSecret 같은 컬럼을 추가해 따로 관리하는 것을 권장

- http://localhost:4000에 접속하면 에러가 발생하여 제대로 동작하지 않음 : Access-Control-Allow-Origin이라는 헤더가 없다는 내용의 에러
    - 브라우저와 서버의 도메인이 일치하지 않으면 기본적으로 요청이 차단됨
    - 브라우저에서 서버로 요청을 보낼 때만 이 현상이 발생하고, 서버에서 서버로 요청을 보낼 때는 발생하지 않음
    - 현재 요청을 보내는 클라이언트(localhost:4000)와 요청을 받는 서버(localhost:8002)의 도메인이 다름 → **CORS(Cross-Origin Resource Sharing)** 문제
- CORS 문제를 해결하려면 응답 헤더에 Access-Control-Allow-Origin 이라는 헤더를 넣어야 함
    - 이 헤더는 클라이언트 도메인의 요청을 허락하겠다는 의미를 갖고 있음
    - res.set 메서드로 직접 넣어도 되지만, npm에 편하게 설치할 수 있는 패키지가 있음 → **cors**
- 응답 헤더를 조작하려면 NodeCat이 아니라 NodeBird API 서버에서 바꿔야 함 → 응답은 API 서버가 보내는 것이기 때문
    - NodeBird API에 cors 모듈을 설치하면 됨 : `$ npm i cors`
- v2.js에 적용

```jsx
const express = require('express');
const cors = require('cors');

const { verifyToken, apiLimiter, corsWhenDomainMatches } = require('../middlewares');
const { createToken, tokenTest, getMyPosts, getPostsByHashtag } = require('../controllers/v2');

const router = express.Router();
router.use(cors({
	credentials: true, // 이 옵션을 활성화해야 다른 도메인 간에 쿠키가 공유됨
}));
...
```

- 다시 http://localhost:4000에 접속하면 토큰이 발급된 것을 볼 수 있음
- 토큰이 발급되지 않고 429 에러가 발생한다면, 이전 절에서 적용한 사용량 제한 때문에 그런 것이므로 제한이 풀릴 때까지 다시 시도
- 새로운 문제가 발생 : 요청을 보내는 주체가 클라이언트라서 비밀 키(proecess.env.CLIENT_SECRET)가 모두에게 노출됨 → 처음에 비밀 키 발급 시 허용한 도메인을 적게 함
    - 호스트와 비밀 키가 모두 일치할 때만 CORS를 허용하게 수정

```jsx
const express = require('express');

const { verifyToken, apiLimiter, corsWhenDomainMatches } = require('../middlewares');
const { createToken, tokenTest, getMyPosts, getPostsByHashtag } = require('../controllers/v2');

const router = express.Router();

router.use(corsWhenDomainMatches);

// POST /v2/token
router.post('/token', apiLimiter, createToken);
...
```

```jsx
const jwt = require('jsonwebtoken');
const rateLimit = require('express-rate-limit');
const cors = require('cors');
const { Domain } = require('../models');
...
exports.corsWhenDomainMatches = async (req, res, next) => {
  const domain = await Domain.findOne({
    where: { host: new URL(req.get('origin')).host },
  });
  if (domain) {
    cors({ 
      origin: req.get('origin'), // origin 속성 추가(허용할 도메인만 따로 적으면 됨)
      credentials: true,
    })(req, res, next); // 미들웨어의 작동 방식 커스터마이징
  } else {
    next();
  }
};
```

- 도메인 모델로 클라이언트의 도메인(req.get(’origin’))과 호스트가 일치하는 것이 있는지 검사
- http나 https 같은 프로토콜을 떼어낼 때는 주소를 URL 객체로 만들어서 host 속성을 사용
    - 일치하는 것이 있다면 CORS를 허용해서 다음 미들웨어로 보내고, 일치하는 것이 없다면 CORS 없이 next를 호출
- 현재 클라이언트와 서버에서 같은 비밀 키를 써서 문제가 될 수 있음 → 다양한 환경의 비밀 키를 발급하는 카카오처럼 환경별로 키를 구분해서 발급하는 것이 바람직
    - 카카오의 경우 REST API 키가 서버용 비밀 키이고, 자바 스크립트 키가 클라이언트용 비밀 키