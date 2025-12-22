# 11장 : 노드 서비스 테스트하기

- 실제 서비스를 개발 완료한 후, 개발자나 QA들은 자신이 만든 서비스가 제대로 동작하는지 테스트
- 기능이 많다면 일일이 수작업으로 테스트하기에는 작업하기에는 작업량이 너무 많음
- 테스트를 자동화해 프로그램이 프로그램을 테스트하도록 하기도 함
- 테스트 환경과 실제 서비스 환경은 다르므로 테스트하는 데 제약이 따를 수도 있고, 테스트 결과와 실제 동작 결과가 다를 수 있음 → 테스트 환경에서 실제 환경을 최대한 흉내 내서 작업

# 11.1 테스트 준비하기

- 테스트에 사용할 패키지는 **jest**
- 페이스북에서 만든 오픈 소스
- 테스팅에 필요한 툴들을 대부분 갖추고 있어서 편리
- 테스팅 툴은 개발할 때만 사용하므로 **-D** 옵션을 사용 : `$ npm i -D jest`
- package.json에는 test라는 명령어를 등록

```json
{
  "name": "nodebird",
  "version": "0.0.1",
  "description": "익스프레스로 만드는 SNS 서비스",
  "main": "app.js",
  "scripts": {
    "start": "nodemon app",
    "test": "jest"
  },
 ...
}
```

- middlewares 폴더 안에 index.test.js를 만듦
- 테스트용 파일은 파일명과 확장자 사이에 test나 spec을 넣음
- `npm test`로 테스트 코드를 실행

```jsx
test('1 + 1은 2입니다.', () => {
// 첫 번째 인수 : 테스트에 대한 설명, 두 번째 인수 : 테스트 내용
  expect(1 + 1).toEqual(2);
  // expect 함수의 인수 : 실제 코드, toEqual 함수의 인수 : 예상 결괏값
  // toEqual의 인수를 3으로 바꾸면 테스트에 실패
});
```

# 11.2 유닛 테스트

- middlewares/index.js에 있는 isLoggedIn과 isNotLoggedIn 함수를 테스트

```jsx
const { isLoggedIn, isNotLoggedIn } = require('./');

describe('isLoggedIn', () => {
  test('로그인 되어있으면 isLoggedIn이 next를 호출해야 함', () => {
  
	});
  test('로그인 되어있지 않으면 isLoggedIn이 에러를 응답해야 함', () => {
  });
});

describe('isNotLoggedIn', () => {
  test('로그인 되어있으면 isNotLoggedIn이 에러를 응답해야 함', () => {

  });

  test('로그인 되어있지 않으면 isNotLoggedIn이 next를 호출해야 함', () => {

  });
});
```

- **describe** 함수 : 테스트를 그룹화해주는 역할
    - 첫 번째 인수 : 그룹에 대한 설명
    - 두 번째 인수(함수) : 그룹에 대한 내용

- middlewares/index.js 확인하기

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
```

- 실제 코드에서는 익스프레스가 req, res 객체와 next 함수를 인수로 넣어서 사용
    - req 객체는 isAuthenticated 메서드가 존재하고 res 객체에도 status, send, redirect 메서드가 존재
- 가짜 객체와 함수를 만들어서 넣어줌
    - 테스트의 역할은 코드나 함수가 제대로 실행되는지를 검사하고 값이 일치하는지 검사하는 것
    - 테스트 코드의 객체가 실제 익스프레스 객체가 아니어도 됨
    - **모킹(mocking)** : 가짜 객체, 가짜 함수를 넣는 행위

```jsx
const { isLoggedIn, isNotLoggedIn } = require('./');

describe('isLoggedIn', () => { // req, res, next를 모킹
  const res = {
    status: jest.fn(() => res), // 함수의 반환값 지정 : jest.fn(() => 반환값) 사용
    send: jest.fn(), // 함수를 모킹할 때는 jest.fn 메서드 사용
  };
  const next = jest.fn();

  test('로그인 되어있으면 isLoggedIn이 next를 호출해야 함', () => {
    const req = {
      isAuthenticated: jest.fn(() => true), // 로그인 여부를 알려주는 함수
    };
    isLoggedIn(req, res, next);
    expect(next).toBeCalledTimes(1);
  });

  test('로그인 되어있지 않으면 isLoggedIn이 에러를 응답해야 함', () => {
    const req = {
      isAuthenticated: jest.fn(() => false),
    };
    isLoggedIn(req, res, next);
    expect(res.status).toBeCalledWith(403);
    expect(res.send).toBeCalledWith('로그인 필요');
  });
});

describe('isNotLoggedIn', () => {
  const res = { // res 객체는 여러 텍스트에서 사용하는 모양이 같으므로 재활용 가능
    redirect: jest.fn(),
  };
  const next = jest.fn();

  test('로그인 되어있으면 isNotLoggedIn이 에러를 응답해야 함', () => {
    const req = {
    // req 객체는 isAuthenticated 메서드가 다른 모양이기 때문에 각각의 test에 따로 선언
      isAuthenticated: jest.fn(() => true),
    };
    isNotLoggedIn(req, res, next);
    const message = encodeURIComponent('로그인한 상태입니다.');
    expect(res.redirect).toBeCalledWith(`/?error=${message}`);
    // toBeCalledWith(인수) : 특정 인수와 함께 호출되었는지 체크하는 메서드
  });

  test('로그인 되어있지 않으면 isNotLoggedIn이 next를 호출해야 함', () => {
    const req = {
      isAuthenticated: jest.fn(() => false),
    };
    isNotLoggedIn(req, res, next); // 모킹된 객체와 함수를 사용해 isLoggedIn 미들웨어 호출
    expect(next).toHaveBeenCalledTimes(1); // expect로 원하는 내용대로 실행되었는지 체크
    // toHaveBeenCalledTime(숫자) : 정확하게 몇 번 호출되었는지 체크하는 메서드
  });
});
```

- 테스트는 통과할 것
- 작성하지 않은 테스트도 통과 → 테스트한다고 해서 에러가 발생하지 않는다고 단정할 수 없는 이유
- **유닛 테스트(unit test)** : 작은 단위의 함수나 모듈이 의도된 대로 정확히 작동하는지 테스트하는 것
    - 나중에 함수를 수정하면 기존에 작성해둔 테스트는 실패하게 됨
    - 함수가 수정되었을 때 어떤 부분이 고장 나는지를 테스트를 통해 알 수 있음
    - 테스트 코드도 기존 코드가 변경된 것에 맞춰 수정해야 함

- user 컨트롤러 테스트

```jsx
jest.mock('../models/user');
const User = require('../models/user');
const { follow } = require('./user');

describe('follow', () => {
  const req = {
    user: { id: 1 },
    params: { id: 2 },
  };
  const res = {
    status: jest.fn(() => res),
    send: jest.fn(),
  };
  const next = jest.fn();

  test('사용자를 찾아 팔로잉을 추가하고 success를 응답해야 함', async () => {
    User.findOne.mockReturnValue({
      addFollowing(id) {
        return Promise.resolve(true);
      }
    });
    await follow(req, res, next);
    expect(res.send).toBeCalledWith('success');
  });

  test('사용자를 못 찾으면 res.status(404).send(no user)를 호출함', async () => {
    User.findOne.mockReturnValue(null);
    await follow(req, res, next);
    expect(res.status).toBeCalledWith(404);
    expect(res.send).toBeCalledWith('no user');
  });

  test('DB에서 에러가 발생하면 next(error) 호출함', async () => {
    const message = 'DB에러';
    User.findOne.mockReturnValue(Promise.reject(message));
    await follow(req, res, next);
    expect(next).toBeCalledWith(message);
  });
});
```

- follow 함수는 async 함수이므로 await을 붙여야 함수가 전부 실행 완료된 후 expect 함수가 실행됨
- follow 컨트롤러 안에는 User라는 모델이 들어 있는데, 실제 데이터베이스와 연결되어 있으므로 User 모델도 모킹해야 함
- jest에서는 모듈도 모킹 가능 → `jest.mock` 메서드 사용
    - 메서드에 모킹할 모듈의 경로를 인수로 넣고 그 모듈을 불러옴
    - 해당 모듈의 메서드는 전부 가짜 메서드가 됨
    - 가짜 메서드에는 `mockReturnValue` 등의 메서드가 생성됨
- 모킹할 때에는 모킹된 결과물이 실제 코드에서 어떤 역할을 할지 미리 예상하는 것이 중요

- 두 번째 테스트에서는 User.findOne이 null을 반환해 사용자를 찾지 못한 상황을 테스트
- 세 번째 테스트에서는 Promise.reject로 에러가 발생하도록 함(DB 연결에 에러가 발생한 상황을 모킹한 것)
- 테스트를 해도 실제 서비스의 실제 데이터베이스에서는 문제가 발생할 수 있는데, 이때는 유닛 테스트 말고 다른 종류의 테스트를 진행해야 함
    - 통합 테스트나 시스템 테스트를 하곤 함

# 11.3 테스트 커버리지

- 유닛 테스트를 작성하다 보면, 전체 코드 중에서 어떤 부분이 테스트되고 어떤 부분이 테스트되지 않는지 궁금해짐
- **커버리지(coverage)** : 전체 코드 중에서 테스트되고 있는 코드의 비율과 테스트되고 있지 않은 코드의 위치를 알려주는 jest의 기능
- 커버리지 기능을 사용하기 위해 package.json에 jest 설정을 입력

```json
{
  "name": "nodebird",
  "version": "0.0.1",
  "description": "익스프레스로 만드는 SNS 서비스",
  "main": "app.js",
  "scripts": {
    "start": "nodemon app",
    "test": "jest",
    "coverage": "jest --coverage"
    // jest 명령어 뒤에 --coverage 옵션을 붙이면 jest가 테스트 커버리지를 분석
  },
```

- 테스트 결과가 출력되고, 추가적으로 표가 하나 더 출력됨
    - 열 : File(파일과 폴더 이름), % Stmt(구문 비율), % Branch(if문 등의 분기점 비율), % Funcs(함수 비율), % Lines(코드 줄 수 비율), Uncovered Line #s(커버되지 않은 줄 위치)
    - 퍼센티지가 높을수록 많은 코드가 테스트 되었다는 뜻
    - 명시적으로 테스트하고 require한 코드만 커버리지 분석이 된다는 점에 주의
    - All files라 하더라도 실제 모든 코드를 테스트한 것은 아닐 수도 있음

- 테스트 커버리지를 올리기 위해 테스트를 작성

```jsx
const Sequelize = require('sequelize');
const User = require('./user');
const config = require('../config/config')['test'];
const sequelize = new Sequelize(
  config.database, config.username, config.password, config,
);

describe('User 모델', () => {
  test('static init 메서드 호출', () => {
    expect(User.initiate(sequelize)).toBe(undefined);
  });
  test('static associate 메서드 호출', () => {
    const db = {
      User: {
        hasMany: jest.fn(),
        belongsToMany: jest.fn(),
      },
      Post: {},
    };
    User.associate(db);
    expect(db.User.hasMany).toHaveBeenCalledWith(db.Post);
    expect(db.User.belongsToMany).toHaveBeenCalledTimes(2);
  });
});
```

- initiate와 associate 메서드가 제대로 호출되는지 테스트
- db 객체는 모킹
- models 폴더에 모델이 아닌 테스트 파일을 생성했으므로 models/index.js를 수정해야 함
    - 모델을 시퀄라이즈와 자동으로 연결할 때 test 파일들도 걸러내도록 함

```jsx
...
fs
  .readdirSync(__dirname) // 현재 폴더의 모든 파일을 조회
  .filter(file => { // 숨김 파일, index.js, js 확장자가 아닌 파일 필터링
    return (file.indexOf('.') !== 0) && (file !== basename) && (file.slice(-3) === '.js');
  })
  .forEach(file => { // 해당 파일의 모델 불러와서 init
    const model = require(path.join(__dirname, file));
    console.log(file, model.name);
    db[model.name] = model;
    model.initiate(sequelize);
  });
...
```

- 다시 테스트를 수행하면 성공
- 테스트 커버리지가 대폭 올라간 것을 볼 수 있음
- 테스트 커버리지를 높이는 것에 집착하기보다는 필요한 부분 위주로 올바르게 테스트하는 것이 좋음

# 11.4 통합 테스트

- **통합 테스트(integration test)** : 모두 유기적으로 잘 작동하는지 테스트하는 것
- supertest 설치 : `$ npm i -D supertest`
- supertest 패키지를 사용해 auth.js의 라우터들을 테스트
    - supertest를 사용하기 위해서는 app 객체를 모듈로 만들어 분리해야 함
    - app.js 파일에서 app 객체를 모듈로 만든 후, server.js에서 불러와 listen하기
    - server.js는 app의 포트 리스닝만 담당
    - 서버 실행 부분을 분리했으므로 app.js에서는 순수한 서버 코드만 남게 됨

```jsx
...
app.use((err, req, res, next) => {
  console.error(err);
  res.locals.message = err.message;
  res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
  res.status(err.status || 500);
  res.render('error');
});

module.exports = app;
```

```jsx
const app = require('./app');

app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기중');
});
```

```jsx
{
  "name": "nodebird",
  "version": "0.0.1",
  "description": "익스프레스로 만드는 SNS 서비스",
  "main": "app.js",
  "scripts": {
    "start": "nodemon server", // npm start 명령어도 바뀐 파일에 맞게 변경
    "test": "jest",
    "coverage": "jest --coverage"
  },
  ...
```

- 테스트용 데이터베이스도 설정
- 통합 테스트에서는 데이터베이스 코드를 모킹하지 않으므로 데이터베이스에 실제로 테스트용 데이터가 저장됨
- 실제 서비스 중인 데이터베이스에 테스트용 데이터가 들어가면 안 되므로, 테스트용 데이터베이스를 따로 만드는 것이 좋음
- config/config.json에서 test 속성을 수정
- 테스트 환경에서는 test 속성의 정볼르 사용해 데이터베이스에 연결

```jsx
{
  "development": {
    "username": "root",
    "password": "nodejsbook",
    "database": "nodebird",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": "nodejsbook",
    "database": "nodebird_test",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "database_production",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

- 콘솔에 nodebird_test 데이터베이스를 생성하는 명령어 입력 : `$ npx sequelize db:create —env test`

- 테스트 코드 작성 : routes/auth.test.js 파일을 작성

```jsx
const request = require('supertest');
const { sequelize } = require('../models');
const app = require('../app');

beforeAll(async () => {
// beforeAll 함수 추가(모든 테스트를 실행하기 전에 수행해야 할 코드를 넣는 공간)
  await sequelize.sync(); // 데이터베이스에 테이블 생성
});

describe('POST /join', () => { // 회원가입 테스트
  test('로그인 안 했으면 가입', (done) => {
    request(app)
      .post('/auth/join')
      .send({
        email: 'zerohch0@gmail.com',
        nick: 'zerocho',
        password: 'nodejsbook',
      })
      .expect('Location', '/')
      .expect(302, done);
  });
});

describe('POST /join', () => { // 로그인한 상태에서 회원 가입을 시도하는 경우를 테스트
// 로그인한 상태여야 회원 가입을 테스트할 수 있으므로 로그인 요청과 회원 가입 요청이 순서대로 이뤄져야 함
  const agent = request.agent(app); // agent를 만들어서 하나 이상의 요청에서 재사용 가능
  beforeEach((done) => { // 각각의 테스트 실행에 앞서 먼저 실행되는 부분
    agent // agent 객체로 로그인을 먼저 수행
      .post('/auth/login')
      .send({
        email: 'zerohch0@gmail.com',
        password: 'nodejsbook',
      })
      .end(done); // 함수가 마무리되었음을 알림
  });

  test('이미 로그인했으면 redirect /', (done) => {// 로그인된 agent로 회원 가입 테스트를 진행
    const message = encodeURIComponent('로그인한 상태입니다.');
    agent
      .post('/auth/join')
      .send({
        email: 'zerohch0@gmail.com',
        nick: 'zerocho',
        password: 'nodejsbook',
      })
      .expect('Location', `/?error=${message}`)
      .expect(302, done);
  });
});

describe('POST /login', () => {
  test('가입되지 않은 회원', (done) => {
    const message = encodeURIComponent('가입되지 않은 회원입니다.');
    request(app)
      .post('/auth/login')
      .send({
        email: 'zerohch1@gmail.com',
        password: 'nodejsbook',
      })
      .expect('Location', `/?error=${message}`)
      .expect(302, done);
  });

  test('로그인 수행', (done) => { 
    request(app)
      .post('/auth/login')
      .send({
        email: 'zerohch0@gmail.com',
        password: 'nodejsbook',
      })
      .expect('Location', '/')
      .expect(302, done);
  });

  test('비밀번호 틀림', (done) => {
    const message = encodeURIComponent('비밀번호가 일치하지 않습니다.');
    request(app)
      .post('/auth/login')
      .send({
        email: 'zerohch0@gmail.com',
        password: 'wrong',
      })
      .expect('Location', `/?error=${message}`)
      .expect(302, done);
  });
});

describe('GET /logout', () => {
  test('로그인 되어있지 않으면 403', (done) => {
    request(app)
      .get('/auth/logout')
      .expect(403, done);
  });

  const agent = request.agent(app);
  beforeEach((done) => {
    agent
      .post('/auth/login')
      .send({
        email: 'zerohch0@gmail.com',
        password: 'nodejsbook',
      })
      .end(done);
  });

  test('로그아웃 수행', (done) => {
    agent
      .get('/auth/logout')
      .expect('Location', `/`)
      .expect(302, done);
  });
});

afterAll(async () => {
  await sequelize.sync({ force: true });
});
```

- 다른 라우터 중에서도 로그인해야 접근할 수 있는 라우터가 있을 것
- 마찬가지로 beforeEach같은 함수에서 미리 로그인한 agent를 마련하면 됨

# 11.5 부하 테스트

- **부하 테스트(load test)** : 서버가 얼마만큼의 요청을 견딜 수 있는지(또는 수용할 수 있는지) 테스트하는 방법
- 코드에 문법적, 논리적 문제가 없더라도 서버의 하드웨어 제약 때문에 서비스가 중단될 수 있음
- 대표적인 것이 OOM(Out of Memory) 문제
    - 서버는 접속자들의 정보를 저장하기 위해 각각의 접속자마다 일정한 메모리를 할당
    - 이렇게 사용하는 메모리의 양이 증가하다가 서버의 메모리 용량을 초과하게 되면 문제가 발생
    - 부하 테스트를 통해 이를 어느 정도 예측 가능
- artillery를 설치하고 npm start 명령어로 서버를 실행 : `$ npm i -D artillery` → `$ npm start`
- 새로운 콘솔을 하나 더 띄운 후 명령어를 입력 : `npx artillery quick --count 100 -n 50 http://localhost:8001`
    - http://local:8001에 빠르게 부하 테스트를 하는 방법
    - `--count` 옵션 : 가상의 사용자 수를 의미
    - `-n` : 요청 횟수를 의미
    - 100명의 가상 사용자가 50번씩 요청을 보내므로 총 5,000번의 요청이 서버로 전달됨
    - 실제 서비스를 할 때 5,000번의 요청은 그렇게 많은 양의 요청이 아님
    - 다만, 절대적인 숫자가 중요한 것이 아니라 하나의 요청이 얼마나 많은 작업을 하는지가 더 중요

- 부하 테스트를 할 때 단순히 한 페이지에만 요청을 보내는 것이 아니라 실제 사용자의 행동을 모방해 시나리오를 작성할 수 있음
- 이때는 JSON 형식의 설정 파일을 작성해야 함

```json
{
  "config": { 
    "target": "http://localhost:8001", // target을 현재 서버로 잡음
    "http": {
      "timeout": 30 // 요청이 30초 이내에 처리되지 않으면 실패로 간주
    },
    "phases": [
      {
        "duration": 30, // 30초 동안 매초 20명의 사용자를 생성하도록 함
        "arrivalRate": 20
      }
    ]
  },
  "scenarios": [ // 가상 사용자들이 어떠한 동작을 할지 적음
    {
      "flow": [
        {
          "get": { // 먼저 메인 페이지(GET /)에 접속
            "url": "/"
          }
        },
        {
          "post": {
            "url": "/auth/login", // 로그인을 한 후
            "json": {
              "email": "zerohch0@gmail.com", // 로그인 요청의 본문으로
              "password": "Wpfhsms0!!" // emial과 password를 JSON 형식으로 보냄
            },
            "followRedirect": false // 로그인 성공 후 redirect 동작은 테스트에서 무시
          }
        },
        {
          "get": {
            "url": "/hashtag?hashtag=nodebird" // 해시태그 검색 수행
          }
        }
      ]
    }
  ]
}
```

- `npx artillery run` 파일명 명령어로 부하 테스트를 실행
    - 600명의 접속자가 각각 세 번의 요청을 보내서(한 번의 redirect는 무시) 총 1,800번의 요청이 전송됨
    - 각각의 요청이 모두 데이터베이스에 최소 한 번씩 접근하므로 서버와 DB에 상당한 무리가 갈 수 있음
    - 서버의 사양을 업그레이드하거나, 서버를 여러 개 두거나, 코드를 더 효율적으로 개선하는 방법이 있음
- 보통의 경우는 요청-응답 시 데이터베이스에 접근할 때 시간이 가장 많이 소요됨
- 서버는 여러 대로 늘리기 쉽지만, 데이터베이스는 늘리기 어려우므로 하나의 데이터베이스에 많은 요청이 몰리곤 함
    - 최대한 데이터베이스에 접근하는 요청을 줄이면 좋음 → 반복적으로 가져오는 데이터는 캐싱을 하는 등
- 서버의 성능과 네트워크 상황에 따라 다르지만 arrivalRate를 줄이거나 늘려서 자신의 서버가 어느 정도의 요청을 수용할 수 있을지 체크해보는 것이 좋음
- 한 번만 테스트하는 게 아니라 여러 번 같은 설정값으로 테스트해 평균치를 내보는 게 좋음
- 다만, artillery만으로는 네트워크가 느린 것인지, 서버가 느린 것인지, DB가 느린 것인지까지는 파악할 수 없음 → 좀 더 정교한 모니터링과 성능 측정을 원한다면 datadog이나 newrelic 같은 상용 서비스를 적용해보면 좋음