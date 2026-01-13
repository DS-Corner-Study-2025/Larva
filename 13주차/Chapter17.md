- 최근에는 노드나 웹에서 자바스크립트 대신 타입스크립트로 개발하는 것이 트렌드
- NodeBird 프로젝트를 타입스크립트로 전환해보기

# 17.1 타입스크립트 기본 문법

- **타입스크립트** : 자바스크립트에 명시적으로 타입이 추가된 언어
    - 자바스크립트에도 문자열, 숫자, 불 값, 객체 같은 자료형 타입이 존재
    - 자바스크립트 코드를 작성할 때 명시적으로 타입을 지정하지 않았을 뿐
- 타입스크립트 코드는 tsc라는 컴파일러를 통해 자바스크립트 코드로 변환 가능
- 노드는 자바스크립트만 실행할 수 있으므로, 타입스크립트 코드를 자바스크립트 코드로 변환해야만 실행 가능
    - **디노(deno)**라는 타입스크립트를 실행할 수 있는 런타임이 존재하기는 함
    - 아직 노드보다는 대중적이지 않아 많은 사람이 타입스크립트를 자바스크립트로 변환해 노드와 함께 사용
- 타입스크립트 컴파일러는 typescript 패키지를 설치하면 같이 설치됨
- tsc 명령어를 통해 컴파일러 작동 가능
    - 타입스크립트 코드의 타입을 검사하고 자바스크립트로 변환하는 것이 tsc의 주 기능
    - tsc 명령어는 tsconfig.json에 적힌 설정대로 실행
    - tsconfig.json은 직접 만들어도 되지만 `tsc --init` 명령어를 통해 만드는 것이 좋음

- 타입 스크립트 실습을 위한 폴더를 만들고 폴더 안에 명령어 입력 : `$ npm init -y` → `$ npm i typescript` → `$ npx tsc --init`

```json
{
  "compilerOptions": {
    "target": "es2016",                          /* Set the Javascript language */  
    "module": "commonjs",                        /* Specify what module code is */
    "esModuleInterop": true,                     /* Emit additional JavaScript to */ 
    "forceConsistentCasingInFileNames": true,    /* Ensure that casing is correct */
    "strict": true,                              /* Enable all strict type-checking */
    "skipLibCheck": true                         /* Skip type checking all .d.ts files */
  }
}
```

- tsc는 tsconfig.json에 따라 자바스크립트 결과물을 만들어내므로 설정을 달리하면 결과물도 달라짐
    - target : 결과물의 문법을 어떤 버전의 자바스크립트 코드로 만들어낼지 정함(es2016 : 2016년 자바스크립트 버전)
    - module : 결과물의 모듈 시스템을 어떤 종류로 할지 정함. 노드라면 commonjs를 하면 되고, 최신 브라우저에서는 es2022를 함
    - esModuleInterop : CommonJS 모듈도 ECMAScript 모듈처럼 인식하게 해줌
    - forceConsistentCasingInFileNames : true이면 모듈을 import할 때 파일명의 대소문자가 정확히 일치해야 함(리눅스나 맥에서는 파일명 대소문자를 구분하므로 켜두는 것이 좋음)
    - strict : 엄격한 타입 검사를 할지 지정
    - skipLibCheck : true이면 모든 라이브러리의 타입을 검사하는 대신 내가 직접적으로 사용하는 라이브러리의 타입만 검사해 시간을 아낄 수 있음

- compare.js와 index.ts 파일을 만들어서 자바스크립트와 타입스크립트 비교

```jsx
let a = 'hello';
a = 'world';
```

```tsx
let a  = 'hello';
a = 'world';
```

- ts 파일은 tsc 명령어를 통해 js로 변환 : `$ npx tsc`

```jsx
"use strict";   // tsc에서 추가하는 코드
let a  = 'hello';
a = 'world';
```

```jsx
let a  = 'hello';
a = 123;
```

```tsx
let a  = 'hello';
a = 123;
// 에러 발생. 변수 a가 string(문자열) 타입을 갖는데 타입이 number(숫자)인 123 값을 넣음
// ts에서는 변수라 하더라도 고정된 타입을 갖고 있음
// 에러가 발생해도 index.js는 그대로 생성됨 -> 타입스크립트에서 타입 검사가 실패해도 js 변환은 이루어짐
```

- 결과물을 만들어내지 않고 타입 검사만 하고 싶은 경우 : `$ npx tsc --noEmit`

- 타입스크립트에 명시적인 타입을 붙여보기
- 변수와 함수의 매개변수, 반환값에 타입을 붙인다고 생각하면 됨 → 타이핑한다

```jsx
let a = true;
const b = { hello: 'world' };

function add(x, y) { return x + y };
const minus = (x, y) => x - y;
```

```tsx
let a: boolean = true;
const b: { hello: string } = { hello: 'world' };

function add(x: number, y: number): number { return x + y };
const minus = (x: number, y: number): number => x - y;
// 콜론(:) 뒷부분이 타입 자리
// 타입스크립트는 코드로부터 어느 정도 타입을 추론할 수 있으므로 에러가 발생하지 않는다면 타입을 적지 않아도 됨
```

- 타입으로 사용 가능한 값

| 분류 | 설명 | 예시 |
| --- | --- | --- |
| 기본형 | `string`, `number`, `boolean`, `symbol`, `object`, `undefined`, `null`, `bigint` | `const a: string = 'hello';
const b: object = { hello: 'ts' };` |
| 배열 | 타입 뒤에 `[]`를 붙임. 길이가 고정된 배열이면 `[]` 안에 타입을 적음 | `const a: string{] = ['hello', 'ts', 'js'];
const b: [number, string] = [123, 'node'];` |
| 상수 | `1`, `'hello'`, `true` 등의 고정값 | `const a: 1 = 1;` |
| 변수 | `typeof` 변수로 해당 변수의 타입 사용 가능 | `let a = 'hello';
const b: typeof a = 'ts';` |
| 클래스 | 클래스 이름을 그대로 인스턴스의 타입으로 사용 가능 | `class A {}
const a = new A();` |
| type 선언 | 다른 타입들을 조합해서 새로운 타입 생성 가능. 유니언(|), 인터섹션(&) 사용 가능 | `const a: { hello: string } = { hello: 'ts' };
type B = { wow: number };
const b: B = { wow: 123 };
type C = string | number;
let c: C = 123;` |
| 인터페이스 | interface 선언 | `interface A { hello: string, wow: number }
const a: A = { hello: 'ts', wow: 123 }` |

```tsx
let a = true;
const b = { hello: 'world' };

function add(x, y) { return x + y };
const minus = (x, y) => x - y;
```

- 타입을 제거한 뒤 tsc 명령어를 수행하면 add와 minus 함수의 매개변수 x, y에 대해 에러가 발생
    - `Parameter 'x' implicitly has an 'any' type` : 매개변수 x와 y의 타입을 알 수 없어서 any라는 타입으로 지정했다는 내용의 에러

- tsc에서는 타입을 추론할 수 없으면 대부분 any라는 타입을 붙임
    - any는 어떤 타입이 되어도 상관없다는 뜻이므로 타입스크립트를 쓰는 의미가 퇴색됨
    - any 타입이 생기는 것을 최대한 지양해야 함

```tsx
let a: string | number = 'hello'; // 유니언 타이핑
a = 123;

let arr: string[] = []; // 배열 타이핑
arr.push('hello');

interface Inter {
  hello: string;
  world?: number; // 있어도 그만 없어도 그만인 속성
} // 객체를 인터페이스로 타이핑할 수 있음
const b: Inter = { hello: 'interface' };

type Type = {
  hello: string;
  func?: (param?: boolean) => void; // 함수는 이런 식으로 타이핑함
}
const c: Type = { hello: 'type' };

interface Merge {
  x: number,
}
interface Merge {
  y: number,
}
const m: Merge = { x: 1, y: 2 };

export { a }; // 타입스크립트 ECMAScript 모듈을 사용

// string || number : 문자열 또는 숫자 타입
// 객체는 interface나 type으로 타이핑 가능
// Inter와 Type은 각각 interface와 type의 이름으로, 스스로 작명하면 됨(대문자로 시작하는 것이 관습)
// 속성 이름 뒤에 물음표(?)가 붙어 있으면, 그 속성은 있어도 되고 없어도 되는 속성이라는 뜻
// func : 함수를 타이핑하는 방법(매개변수를 타이핑하고 반환값은 => 뒤에 타입을 적음)
// param 매개변수 뒤에 물음표가 붙는데, 있어도 되고 없어도 되는 매개변수라는 뜻
```

- 타입스크립트에서는 같은 인터페이스를 여러 번 선언하면 하나로 합쳐짐(기존에 선언한 인터페이스를 같은 이름의 인터페이스를 선언해 확장할 수 있다는 뜻)
- 타입스크립트에서는 ECMAScript 모듈을 사용
- 노드 코드를 타입스크립트에서 사용할 때, 노드의 타입은 따로 설치해야 사용 가능 : @types/node 패키지 설치

# 17.2 커뮤니티 타입 정의 적용하기

- NodeBird 프로젝트에 타입스크립트 적용하기
- 타입스크립트를 설치하고 tsconfig.json 파일 생성 : `$ npm i typescript` → `$ npx tsc --init`

```json
{
  "compilerOptions": {
		 "target": "es2016",                                         /* Set the JavaScript */
		 "module": "commonjs",                              /* Specify what module code is */
		 "allowJs": true,
		 "esModuleInterop": true,                         /* Emit additional JavaScript to */
		 "forceConsistentCasingInFileNames": true,        /* Ensure that casing is correct */
		 "strict": true,                       /* Enable all strict type-checking options. */
		 "skipLibCheck": true                       /* Skip type checking all .d.ts files. */
  }
}
```

- server.js와 app.js 파일의 확장자를 ts로 변경
- 모듈 시스템을 CommonJS에서 ECMAScript 모듈로 변경

```tsx
import app from './app';

app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기중');
});
```

```tsx
import express from 'express';
import cookieParser from 'cookie-parser';
import morgan from 'morgan';
import path from 'path';
import session from 'express-session';
import nunjucks from 'nunjucks';
import dotenv from 'dotenv';
import passport from 'passport';

dotenv.config();
import pageRouter from './routes/page';
import authRouter from './routes/auth';
import postRouter from './routes/post';
import userRouter from './routes/user';
import { sequelize } from './models';
import passportConfig from './passport';

const app = express();
passportConfig(); // 패스포트 설정
app.set('port', process.env.PORT || 8001);
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
app.use('/img', express.static(path.join(__dirname, 'uploads')));
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

app.use('/', pageRouter);
app.use('/auth', authRouter);
app.use('/post', postRouter);
app.use('/user', userRouter);

app.use((req, res, next) => {
  const error =  new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
  error.status = 404;
  next(error);
});

const errorHandler = (err, req, res, next) => {
  console.error(err);
  res.locals.message = err.message;
  res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
  res.status(err.status || 500);
  res.render('error');
};
app.use(errorHandler);

export default app;
```

- `tsc --noEmit`을 수행하면 크게 세 종류의 에러 발생
    - TS7016 : 설치한 패키지에 타입 정의가 없다는 에러
        - 익스프레스 외에 cookie-parser, morgan, express-session, nunjucks, passport 등이 해당
        - 자바스크립트로 작성되었으므로 tsc에서 타입 분석을 할 수 없고, 이 라이브러리에서 사용하는 변수와 함수의 모든 타입은 any가 되어버림
        - npm의 DefinitelyTyped 패키지에서 제공하는 커뮤니티 타입 정의를 적용하는 방법 사용 가능
        - DefinitelyTyped 패키지는 익스프레스, cookie-parser 같은 라이브러리에 타입 정의를 적용
        - DefinitelyTyped 패키지를 적용하기 전에 먼저 npmjs.com에서 해당 패키지가 타입스크립트로 작성되었는지, 타입스크립트로 작성되지 않았다면 DefinitelyTyped 타이핑은 존재하는지를 확인해야 함
    - TS2322 : `string | string[]`(문자열 또는 문자열의 배열) 타입인 곳에 `string | undefined`(문자열 또는 undefined) 타입을 넣을 수 없다는 의미
        - secret 속성은 `string | string[]` 타입의 값만 허용하는데, `process.env.COOKIE_SECRET`의 타입은 `string | undefined`이기 때문
        - .env는 타입스크립트 파일이 아니라서 그 값이 실제로 존재하는지 알 수 없어 string | undefinded라고 추론한 것
        - `process.env.COOKIE_SECRET`이 존재하고 undefined가 절대 아니라고 tsc에 알리는 방법 : 느낌표 붙이기

```tsx
...
app.use(session({
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET!, // 느낌표
  cookie: {
    httpOnly: true,
    secure: false,
  },
}));
...
// 느낌표를 붙이는 것도 불가피한 경우를 제외하고는 지양하는 것이 좋음
// undefined가 절대 아님을 보증했는데, 실제로는 undefined라면 에러가 발생하기 때문
```

# 17.3 라이브러리 코드 타이핑하기

- 시퀄라이즈와 passport 패키지를 사용한 코드를 타이핑하기
- passport/index.js, passport/localStrategy.js, passport/kakaoStrategy.js를 ts로 교체하고 ECMAScript 모듈 시스템으로 바꾸기

```tsx
import passport from 'passport';
import local from './localStrategy';
import kakao from './kakaoStrategy';
import User from '../models/user';

export default () => {
  passport.serializeUser((user, done) => {
    done(null, user.id);
  });

  passport.deserializeUser((id: number, done) => {
    User.findOne({
      where: { id },
      include: [{
        model: User,
        attributes: ['id', 'nick'],
        as: 'Followers',
      }, {
        model: User,
        attributes: ['id', 'nick'],
        as: 'Followings',
      }],
    })
      .then(user => done(null, user))
      .catch(err => done(err));
  });

  local();
  kakao();
};
```

```tsx
import passport from 'passport';
import { Strategy as LocalStrategy } from 'passport-local';
import bcrypt from 'bcrypt';
import User from '../models/user';

export default () => {
  passport.use(new LocalStrategy({
    usernameField: 'email',
    passwordField: 'password',
  }, async (email, password, done) => {
    try {
      const exUser = await User.findOne({ where: { email } });
      if (exUser) {
        const result = await bcrypt.compare(password, exUser.password);
        if (result) {
          done(null, exUser);
        } else {
          done(null, false, { message: '비밀번호가 일치하지 않습니다.' });
        }
      } else {
        done(null, false, { message: '가입되지 않은 회원입니다.' });
      }
    } catch (error) {
      console.error(error);
      done(error);
    }
  }));
};
```

```tsx
import passport from 'passport';
import { Strategy as KakaoStrategy } from 'passport-kakao';

import User from '../models/user';

export default () => {
  passport.use(new KakaoStrategy({
    clientID: process.env.KAKAO_ID,
    callbackURL: '/auth/kakao/callback',
    clientSecret: '',
  }, async (accessToken, refreshToken, profile, done) => {
    console.log('kakao profile', profile);
    try {
      const exUser = await User.findOne({
        where: { snsId: profile.id, provider: 'kakao' },
      });
      if (exUser) {
        done(null, exUser);
      } else {
        const newUser = await User.create({
          email: profile._json && profile._json.kaccount_email,
          nick: profile.displayName,
          snsId: profile.id,
          provider: 'kakao',
        });
        done(null, newUser);
      }
    } catch (error) {
      console.error(error);
      done(error);
    }
  }));
};
```

- @types/passport-local과 @types/passport-kakao 설치 : `npm i -D @types/passport-local` `@types/passport-kakao@0.2.1`
- `npx tsc --noEmit`을 입력하면 나오는 에러 두 개
    - TS2339 : Error 객체에 status 속성이 없다는 에러와 user 객체에 id 속성이 없다는 에러 → types 폴더를 만들고 index.d.ts 파일 생성(확장자가 d.ts인 것은 해당 파일이 타입 정의만 포함하고 있다는 사실을 알리는 것)
    
    ```tsx
    import IUser from '../models/user';
    
    declare global { // 전역 스코프를 의미
    // global 스코프에 선언되어 있으므로 인터페이스를 확장할 때도 같은 스코프에 선언해야 함
      interface Error {
        status?: number; // status 속성 추가
      }
    
      namespace Express {
        interface User extends IUser {} // User 모델 상속받음
        // 인터페이스는 다른 타입으로부터 상속도 가능
        // User 모델은 클래스이므로 타입으로 사용 가능
        // Express.User는 User 모델 타입이 되어 id 속성이 존재하게 됨 -> 에러 해결
      }
    }
    ```
    
    - TS2769 :  process.env.KAKAO_ID 부분에서 발생 → 뒤에 느낌표를 붙이기
    
    ```tsx
    ...
    export default () => {
      passport.use(new KakaoStrategy({
        clientID: process.env.KAKAO_ID!, // 느낌표 붙이기
        callbackURL: '/auth/kakao/callback',
        clientSecret: '',
      }, async (accessToken, refreshToken, profile, done) => {
      ...
    ```
    

# 17.4 내가 작성한 코드 타이핑하기

- controllers, routes, middlewares와 같이 직접 작성한 코드가 많은 파일 타이핑
- middlewares/index.js를 ts로변경
- 모듈도 ECMAScript 모듈 시스템으로 수정

 

```tsx
const isLoggedIn = (req, res, next) => {
  if (req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send('로그인 필요');
  }
};

const isNotLoggedIn = (req, res, next) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    const message = encodeURIComponent('로그인한 상태입니다.');
    res.redirect(`/?error=${message}`);
  }
};
export { isLoggedIn, isNotLoggedIn };
```

- `npx tsc --noEmit`을 입력하면 에러가 발생하는데, 매개변수인 err, req, res, next를 타이핑해야 함
    - app.ts와 middlewares, controllers에 해당 매개변수가 존재
    - express로부터 타입을 불러와 타이핑할 수 있음

```tsx
import { RequestHandler } from 'express';

const isLoggedIn: RequestHandler = (req, res, next) => {
  if (req.isAuthenticated()) {
    next();
  } else {
    res.status(403).send('로그인 필요');
  }
};

const isNotLoggedIn: RequestHandler = (req, res, next) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    const message = encodeURIComponent('로그인한 상태입니다.');
    res.redirect(`/?error=${message}`);
  }
};
export { isLoggedIn, isNotLoggedIn };

// Request, Response, NextFunction이 해당 매개변수의 타입인지 알 수 있는 이유 : @types/espress 패키지에서 찾아봤기 때문 
```

- 컨트롤러로 타이핑해보기
- controllers 폴더 내부의 파일들을 전부 ts 확장자로변경
- 모듈도 ECMAScript 모듈 시스템으로 수정
- RequestHandler 타입 추가

 

```tsx
import bcrypt from 'bcrypt';
import passport from 'passport';
import User from '../models/user';
import { RequestHandler } from 'express';

const join: RequestHandler = async (req, res, next) => {
  const { email, nick, password } = req.body;
  try {
    const exUser = await User.findOne({ where: { email } });
    if (exUser) {
      return res.redirect('/join?error=exist');
    }
    const hash = await bcrypt.hash(password, 12);
    await User.create({
      email,
      nick,
      password: hash,
    });
    return res.redirect('/');
  } catch (error) {
    console.error(error);
    return next(error);
  }
}

const login: RequestHandler = (req, res, next) => {
  passport.authenticate('local', (authError, user, info) => {
    if (authError) {
      console.error(authError);
      return next(authError);
    }
    if (!user) {
      return res.redirect(`/?loginError=${info.message}`);
    }
    return req.login(user, (loginError) => {
      if (loginError) {
        console.error(loginError);
        return next(loginError);
      }
      return res.redirect('/');
    });
  })(req, res, next); // 미들웨어 내의 미들웨어에는 (req, res, next)를 붙입니다.
};

const logout: RequestHandler = (req, res) => {
  req.logout(() => {
    res.redirect('/');
  });
};

export { login, join, logout };
```

```tsx
import { RequestHandler } from 'express';
import User from '../models/user';
import Post from '../models/post';
import Hashtag from '../models/hashtag';

const renderProfile: RequestHandler = (req, res) => {
  res.render('profile', { title: '내 정보 - NodeBird' });
};

const renderJoin: RequestHandler = (req, res) => {
  res.render('join', { title: '회원가입 - NodeBird' });
};

const renderMain: RequestHandler = async (req, res, next) => {
  try {
    const posts = await Post.findAll({
      include: {
        model: User,
        attributes: ['id', 'nick'],
      },
      order: [['createdAt', 'DESC']],
    });
    res.render('main', {
      title: 'NodeBird',
      twits: posts,
    });
  } catch (err) {
    console.error(err);
    next(err);
  }
}

const renderHashtag: RequestHandler = async (req, res, next) => {
  const query = req.query.hashtag as string;
  if (!query) {
    return res.redirect('/');
  }
  try {
    const hashtag = await Hashtag.findOne({ where: { title: query } });
    let posts: Post[] = [];
    if (hashtag) {
      posts = await hashtag.getPosts({ include: [{ model: User }] });
    }

    return res.render('main', {
      title: `${query} | NodeBird`,
      twits: posts,
    });
  } catch (error) {
    console.error(error);
    return next(error);
  }
};

export { renderHashtag, renderProfile, renderMain, renderJoin };
```

```tsx
import { RequestHandler } from 'express';
import Post from '../models/post';
import Hashtag from '../models/hashtag';

const afterUploadImage: RequestHandler = (req, res) => {
  console.log(req.file);
  res.json({ url: `/img/${req.file?.filename}` });
};

const uploadPost: RequestHandler = async (req, res, next) => {
  try {
    const post = await Post.create({
      content: req.body.content,
      img: req.body.url,
      UserId: req.user?.id,
    });
    const hashtags: string[] = req.body.content.match(/#[^\s#]*/g);
    if (hashtags) {
      const result = await Promise.all(
        hashtags.map(tag => {
          return Hashtag.findOrCreate({
            where: { title: tag.slice(1).toLowerCase() },
          })
        }),
      );
      await post.addHashtags(result.map(r => r[0]));
    }
    res.redirect('/');
  } catch (error) {
    console.error(error);
    next(error);
  }
};

export { afterUploadImage, uploadPost };
```

```tsx
import { RequestHandler } from 'express';
import User from '../models/user';

const follow: RequestHandler = async (req, res, next) => {
  try {
    const user = await User.findOne({ where: { id: req.user?.id } });
    if (user) {
      await user.addFollowing(parseInt(req.params.id, 10));
      res.send('success');
    } else {
      res.status(404).send('no user');
    }
  } catch (error) {
    console.error(error);
    next(error);
  }
};

export { follow };
```

- Post.create에서 UserId가 사용되는데, 타입스크립트는 Post에 UserId 속성이 있는지 알지 못함
- getPosts나 addHashtags, addFollowing은 관계에 따라 시퀄라이즈가 그때그때 생성하는 메서드이기 때문에 타입 스크립트를 알기 어려움
- 직접 모델에 getPosts와 addHashtags, addFollowing 메서드를 타이핑해야 함
- associate에 적힌 관계를 보고 메서드를 타이핑할 수 있음

- UserId는 다른 모델의 키를 타이핑하는 방법
- ForeignKey 타입에 제네릭으로 User[’id’]를 넣으면 됨

- 라우터 타이핑하기
- routes 폴더 내부의 파일들을 전부 ts 확장자로 변경
- ECMAScript 모듈 시스템으로 바꿈

```tsx
import express from 'express';
import passport from 'passport';

import { isLoggedIn, isNotLoggedIn } from '../middlewares';
import { join, login, logout } from '../controllers/auth';

const router = express.Router();

// POST /auth/join
router.post('/join', isNotLoggedIn, join); 

// POST /auth/login
router.post('/login', isNotLoggedIn, login);

// GET /auth/logout
router.get('/logout', isLoggedIn, logout);

// GET /auth/kakao
router.get('/kakao', passport.authenticate('kakao'));

// GET /auth/kakao/callback
router.get('/kakao/callback', passport.authenticate('kakao', {
  failureRedirect: '/?loginError=카카오로그인 실패',
}), (req, res) => {
  res.redirect('/'); // 성공 시에는 /로 이동
});

export default router;
```

```tsx
import express from 'express';
import { isLoggedIn, isNotLoggedIn } from '../middlewares';
import {
  renderProfile, renderJoin, renderMain, renderHashtag,
} from '../controllers/page';

const router = express.Router();

router.use((req, res, next) => {
  res.locals.user = req.user;
  res.locals.followerCount = req.user?.Followers?.length || 0;
  res.locals.followingCount = req.user?.Followings?.length || 0;
  res.locals.followingIdList = req.user?.Followings?.map(f => f.id) || [];
  next();
});

router.get('/profile', isLoggedIn, renderProfile);

router.get('/join', isNotLoggedIn, renderJoin);

router.get('/', renderMain);

router.get('/hashtag', renderHashtag);

export default router;
```

```tsx
import express from 'express';
import multer from 'multer';
import path from 'path';
import fs from 'fs';

import { afterUploadImage, uploadPost } from '../controllers/post';
import { isLoggedIn } from '../middlewares';

const router = express.Router();

try {
  fs.readdirSync('uploads');
} catch (error) {
  console.error('uploads 폴더가 없어 uploads 폴더를 생성합니다.');
  fs.mkdirSync('uploads');
}

const upload = multer({
  storage: multer.diskStorage({
    destination(req, file, cb) {
      cb(null, 'uploads/');
    },
    filename(req, file, cb) {
      const ext = path.extname(file.originalname);
      cb(null, path.basename(file.originalname, ext) + Date.now() + ext);
    },
  }),
  limits: { fileSize: 5 * 1024 * 1024 },
});

// POST /post/img
router.post('/img', isLoggedIn, upload.single('img'), afterUploadImage);

// POST /post
const upload2 = multer();
router.post('/', isLoggedIn, upload2.none(), uploadPost);

export default router;
```

```tsx
import express from 'express';

import { isLoggedIn } from '../middlewares';
import { follow } from '../controllers/user';

const router = express.Router();

// POST /user/:id/follow
router.post('/:id/follow', isLoggedIn, follow);

export default router;
```

- `npx tsc --noEmit` 명령어를 입력하면 마찬가지로 타입스크립트가 User 모델에 Followers와 Followings 속성이 있는지 파악할 수 없어 에러가 발생
- User 모델 수정

 

```tsx
import Sequelize, {
  CreationOptional, InferAttributes, InferCreationAttributes, Model,
  BelongsToManyAddAssociationMixin,
  NonAttribute,
} from 'sequelize';
import Post from './post';

class User extends Model<InferAttributes<User>, InferCreationAttributes<User>> {
...
	declare deletedAt: CreationOptional<Date>;

  declare Followers?: NonAttribute<User[]>;
  declare Followings?: NonAttribute<User[]>;
  declare addFollowing: BelongsToManyAddAssociationMixin<User, number>;
...

// Follwers와 Followings는 원래 존재하는 속성이 아니므로 물음표를 붙임
// 타입도 NonAttribute로 기본 속성이 아님을 표시
```

- 타입스크립트 문법을 익힌 후에는 라이브러리 공식 문서를 참고하며 타입을 분석하는 연습이 필요
- 에러 메시지가 친절한 편이므로 에러 메시지를 참조하며 타이핑하면 됨