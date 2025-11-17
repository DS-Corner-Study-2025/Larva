# 8장 : 몽고디비

- 몽고디비의 특징 중 하나는 자바스크립트 문법을 사용한다는 것
    - 하나의 언어만 사용하면 되므로 생산성도 매우 높음
    - 몽고디비는 흔히 사용하는 RDBMS가 아니라 특색이 뚜렷한 NoSQL이므로 특징을 잘 알고 사용해야 함

# 8.1 NoSQL vs SQL

- **MySQL**은 SQL을 사용하는 대표적인 데이터베이스
- SQL을 사용하지 않는 **NoSQL**(Not only SQL)이라고 부르는 데이터베이스도 있음
    - 몽고디비는 NoSQL의 대표 주자

| SQL(MySQL) | NoSQL(몽고디비) |
| --- | --- |
| 규칙에 맞는 데이터 입력 | 자유로운 데이터 입력 |
| 테이블 간 JOIN 지원 | 컬렉션 간 JOIN 미지원 |
| 안정성, 일관성 | 확장성, 가용성 |
| 용어(테이블, 로우, 컬럼) | 용어(컬렉션, 다큐먼트, 필드) |
- NoSQL에는 고정된 테이블이 없음
    - 테이블에 상응하는 컬렉션이라는 개념이 있긴 하지만, 컬럼을 따로 정의하지는 않음
    - ex. MySQL은 users 테이블을 만들 때 name, age, married 등의 컬럼과 자료형, 옵션 등을 정의하지만, MySQL은 users 테이블을 만들 때 그냥 users 컬렉션을 만들고 끝(users 컬렉션에는 어떠한 데이터든 들어갈 수 있음)
- 단점이 있음에도 몽고디비를 사용하는 이유 : 확장성과 가용성
    - 데이터의 일관성을 보장해주는 기능이 약한 대신 데이터를 빠르게 넣을 수 있고, 쉽게 여러 서버에 데이터를 분산할 수 있음
- 애플리케이션을 만들 때 꼭 한 가지 데이터베이스만 사용해야 하는 건 아님
    - 많은 기업이 SQL과 NoSQL을 동시에 사용하고 있음
    - 각각 특징이 다르므로 알맞은 곳에 사용하면 됨
    - ex. 항공사 예약 시스템의 경우 비행기 표에 관한 정보가 모든 항공사에 일관성 있게 전달되어야 하므로 예약 처리 부분의 데이터베이스는 MySQL을 사용, 핵심 기능 외의 빅데이터, 메시징, 세션 관리 등에는 확장성과 가용성을 위해 몽고디비를 사용할 수도 있음

# 8.2 몽고디비 설치하기

- 설치 완료 후 서버를 실행하기 전에 반드시 데이터가 저장될 폴더를 먼저 만들어야 함(C:\data\db)
- 콘솔에서 `mongod --ipv6` 명령어를 입력해 몽고디비를 실행

# 8.3 컴퍼스 설치하기

- 몽고디비는 관리 도구로 컴퍼스를 제공
- 컴퍼스를 사용하면 GUI를 통해 데이터를 시각적으로 관리할 수 있어 편리
    - 필수적인 것은 아님

# 8.4 데이터베이스 및 컬렉션 생성하기

- 몽고디비 프롬프트에 접속한 후 진행
- 데이터베이스를 만드는 명령어 : `use [데이터베이스명]`
- 데이터베이스 목록을 확인하는 명령어 : `show dbs`
    - 데이터를 최소 한 개 이상 넣어야 목록에 표시됨
- 현재 사용 중인 데이터베이스를 확인하는 명령어 : `db`
- 컬렉션은 다큐먼트를 넣는 순간 자동으로 생성하기 때문에 따로 생성할 필요는 없지만, 직접 컬렉션을 생성하는 명령어가 있긴 함 : `db.createCollection('users');`

# 8.5 CRUD 작업하기

## 8.5.1 Create(생성)

- 컬렉션에 컬럼을 정의하지 않아도 되므로 컬렉션에는 아무 데이터나 넣을 수 있음
- 몽고디비의 자료형
    - 자바스크립트 문법을 사용하므로 자바스크립트의 자료형을 기본적으로 따르지만 추가로 몇 가지 자료형이 더 있음
    - Date나 정규표현식 같은 자바스크립트 객체를 자료형으로 사용할 수 있음
    - Binary Data, ObjectId(기본 키로 쓰이는 값과 비슷한 역할), Int, Long, Decimal, Timestamp, Javascript 등의 추가적인 자료형이 있음
    - Undefined와 Symbol은 몽고디비에서 자료형으로 사용하지 않음
    
    ```bash
    mongosh
    test> use nodejs;
    switched to db nodejs
    nodejs> db.users.insertOne({ name: 'zero', age: 24, married: false, comment: '안녕하세요. 간단히 몽고디비 사용 방법에 대해 알아봅시다.', createdAt: new Date() });
    {
    	acknowledged: true,
    	insetedId: ObjectedId("5a1687007af03c3700826f70")
    }
    nodejs> db.users.insertOne({ name: 'nero', age: 32, married: true, comment: '안녕하세요. zero 친구 nero입니다.', createdAt: new Date() });
    {
    	acknowledged: true,
    	insertId: ObjectId("62fba0deb068d84d69d7c740")
    }
    ```
    
- `db.컬렉션명.insertOne(다큐먼트)`로 다큐먼트를 생성
- `new Date()` : 현재 시간을 입력하라는 뜻
- 명령이 성공적으로 수행되었다면 `acknowledged: true와 insertId: ObjectId("5a1687007af03c3700826f70")`이라는 응답이 옴
    - 실패했다면 에러 내용이 응답으로 옴

## 8.5.2 Read(조회)

- `find({})` : 컬렉션 내의 모든 다큐먼트를 조회하라는 뜻
- 특정 필드만 조회하고 싶다면 find 메서드의 두 번째 인수로 조회할 필드를 넣음
    - `find({}, { _id: 0, name: 1, married: 1 });`
    - 1 또는 true로 표시한 필드만 가져옴
    - _id는 기본적으로 가져오게 되어 있으므로 0 또는 false를 입력해 가져오지 않도록 해야 함
- 조회 시 조건을 주려면 첫 번째 인수 객체에 기입
    - `find({ age: { $gt: 0 }, married: true }, { _id: 0, name: 1, married: 1 });`
    - age가 30 초과, married가 true인 조건
    - `$gt` : 특수 연산자. 몽고디비는 자바스크립트 객체를 사용해서 명령어 쿼리를 생성해야 하므로 $gt 같은 특수한 연산자가 사용됨
    - 자주 쓰이는 연산자 : `$gt`(초과), `$gte`(이상), `$lt`(미만), `$lte`(이하), `$ne`(같지 않음), `$or`(또는), `$in`((배열 요소 중 하나)
- 몽고디비에서 OR 연산은 `$or`를 사용
    - age가 30 초과이거나 married가 false인 다큐먼트 : `find({ $or: [{ age: { $gt: 30 } }, { married: false }] }, { _id: 0, name: 1, married: 1 });`
- 정렬은 `sort` 메서드를 사용
    - 나이가 많은 순서대로 정렬
    - -1은 내림차순, 1은 오름차순
    - `find({}, { _id: 0, name: f, age: 1 }.sort({ age: -1 })`
- 조회할 다큐먼트 개수를 설정 : `limit` 메서드 사용
    - `find({}, { _id: 0, name: 1, age: 1 }.sort({ age: -1 }).limit(1)`
- 다큐먼트 개수를 설정하면서 몇 개를 건너뛸지 설정할 수 있음 : `skip` 메서드 사용
    - `find({}, { _id: 0, name: 1, age: 1 }.sort({ age: -1 }).limit(1).skip(1)`

## 8.5.3 Update(수정)

```bash
nodejs> db.users.updateOne({ name: 'nero' }, { $set: { comment: '안녕하세요 이 필드를 바꿔보겠습니다!' } });
{
	acknowledged: true,
	insertedId: null,
	matchedCount: 1,
	modifiedCount: 0,
	upsertedCount: 0
}
```

- 첫 번째 객체는 수정할 다큐먼트를 지정하는 객체
- 두 번째 객체는 수정할 내용을 입력하는 객체
- `$set` : 어떤 필드를 수정할지 정하는 연선자
    - 이 연산자를 사용하지 않고 일반 객체를 넣는다면, 다큐먼트가 통째로 두 번째 인수로 주어진 객체로 수정됨
    - 일부 필드만 수정하고 싶을 때는 반드시 `$set` 연산자를 지정해야 함

## 8.5.4 Delete(삭제)

```bash
nodejs> db.users.deleteOne({ name: 'nero' })
{ acknowledged: true, deletedCount: 1 }
```

- 삭제할 다큐먼트에 대한 정보가 담긴 객체를 첫 번째 인수로 제공
- 성공 시 삭제된 개수(deletedCount)가 반환됨
- `deleteOne`은 하나의 다큐먼트만 수정하므로 여러 건을 삭제하려면 `deleteMany` 메서드를 사용

# 8.6 몽구스 사용하기

- MySQL에 시퀄라이즈가 있다면 몽고디비에는 몽구스가 있음
- 몽구스는 시퀄라이즈와 달리 ODM(Object Document Mapping)이라고 불림
- 몽고디비는 릴레이션이 아니라 다큐먼트를 사용하므로 ORM이 아니라 ODM
- 몽고디비 자체가 이미 자바스크립트인데도 굳이 자바스크립트 객체와 매핑하는 이유 : 몽고디비에 없어서 불편한 기능들을 몽구스가 보완해주기 때문
    - 스키마(schema)라는 것이 생김 : 몽구스는 몽고디비에 데이터를 넣기 전에 노드 서버 단에서 데이터를 한 번 필터링하는 역할
    - MySQL에 있는 JOIN 기능을 populate라는 메서드로 어느 정도 보완 → 관계가 있는 데이터를 쉽게 가져올 수 있음
    - ES2015 프로미스 문법과 강력하고 가독성이 높은 쿼리 빌더를 지원

```json
{
	"name": "learn-mongoose",
	"version": "0.0.1",
	"description": "몽구스를 배우자",
	"main": "app.js",
	"scripts": {
		"start": "nodemon app"
	},
	"author": "ZeroCho",
	"license": "MIT"
}
```

- 몽구스와 필요한 패키지 설치
    - `npm i express morgan nunjucks mongoose`
    - `npm i -D nodemon`

## 8.6.1 몽고디비 연결하기

- 노드와 몽고디비를 몽구스를 통해 연결할 때 몽고디비는 주소를 사용해 연결
- 주소 형식 : `mongodb://[username:password@]host[:port][/[database][?options]]`
    - `[ ]`  부분은 있어도 되고 없어도 됨을 의미

```jsx
const mongoose = require('mongoose');

const connect = () => {
	// 1번
	if (process.env.NODE_ENV !== 'production') {
		mongoose.set('debug', true);
	}
	// 2번
	mongoose.connect('monodb://root:nodejsbook@localhost: 27017 /admin' , {
		dbName: 'nodejs',
		useNewUrlParser: true,
	}).then(() => {
		console.log("몽고디비 연결 성공";
	}).catch((err) => {
		console.log("몽고디비 연결 에러", err);
	});
};
// 3번
mongoose.connection.on('error', (error) => {
	console.error('몽고디비 연결 에러', error);
});
mongoose.connection.on('disconnected', () => {
	console.error('몽고디비 연결이 끊겼습니다. 연결을 재시도합니다.');
	connect();
});

module.exports = connect;
```

- 1번 : 개발 환경일 때만 콘솔을 통해 몽구스가 생성하는 쿼리 내용을 확인할 수 있게 하는 코드
- 2번 : 몽구스와 몽고디비를 연결하는 부분. 몽고디비 주소로 접속을 시도
    - 비밀번호(nodejsbook 부분)를 자신의 비밀번호로 바꿔야 함
    - 접속을 시도하는 주소의 데이터베이스는 admin이지만 실제로 사용할 데이터베이스는 nodejs이므로 두 번째 인수로 dbName 옵션을 줘서 nodejs 데이터베이스를 사용하게 함
    - 마지막 인수로 주어진 콜백 함수를 통해 연결 여부를 확인
- 3번 : 몽구스 커넥션에 이벤트 리스너를 달아둠, 에러 발생 시 에러 내용을 기록하고 재연결 시도

```jsx
const express = require('express');
const path = require('path');
const morgan = require('morgan');
const nunjucks = require('nunjucks');

const connect = require('./schemas');

const app = express();
app.set('port', process.env.PORT || 3002);
app.set('view engine', 'html');
nunjucks.configure('views', {
	express: app,
	watch: true,
});
connect();

app.use(morgan('dev'));
app.use(express.static(path.join(__dirname, 'public')));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

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
	console.log(app.get('port'), '번 포트에서 대기 중');
});
```

- app.js를 만들고 schemas/index.js와 연결
- 서버를 실행하면 3002번 포트에서 서버가 돌아감
- 연결에 실패한 경우 에러 메시지가 로깅됨
    - 에러는 주로 몽고디비 데이터베이스를 실행하지 않았거나 비밀번호가 틀렸을 때 발생

## 8.6.2 스키마 정의하기

- schemas 폴더에 user.js와 comment.js를 만듦

```jsx
const mongoose = require('mongoose');

const { Schema } = mongoose;
const userSchema = new Schema({
	name: {
		type: String, 
		required: true,
		unique: true, // 고유한 값이어야 함
	},
	age: {
		type: Number,
		required: true, // 필수
	},
	married: {
		type: Booelan,
		required: true,
	},
	comment: String,
	createdAt: {
		type: Date,
		defulat: Date.now, // 기본값 : 데이터 생성 당시의 시간
	},
});

module.exports = mongoose.model('User', userSchema);
```

- 몽구스는 알아서 _id를 기본 키로 생성하므로 _id는 적어줄 필요가 없음
- 몽구스 스키마에서는 String, Number, Data, Buffer, Boolean, Mixed, ObjectedId, Array를 값으로 가질 수 있음

```jsx
const mongoose = require('mongoose');

const { Schema } = mongoose;

const { Types: { ObjectId } } = Schema;
const commentSchema = new Schema({
	commenter: {
		type: ObjectId,
		required: true,
		ref: 'User', // commenter 필드에 User 스키마의 사용자 ObjectId가 들어간다는 뜻
	},
	comment: {
		type: String,
		required: true,
	},
	createdAt: {
		type: Date,
		default: Date.now,
	},
});

module.exports = mongoose.model('Comment', commentSchema);
```

## 8.6.3 쿼리 수행하기

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>몽구스 서버</title>
    <style>
      table { border: 1px solid black; border-collapse: collapse; }
      table th, table td { border: 1px solid black; }
    </style>
  </head>
  <body>
    <div>
      <form id="user-form">
        <fieldset>
          <legend>사용자 등록</legend>
          <div><input id="username" type="text" placeholder="이름"></div>
          <div><input id="age" type="number" placeholder="나이"></div>
          <div><input id="married" type="checkbox"><label for="married">결혼 여부</label></div>
          <button type="submit">등록</button>
        </fieldset>
      </form>
    </div>
    <br>
    <table id="user-list">
      <thead>
      <tr>
        <th>아이디</th>
        <th>이름</th>
        <th>나이</th>
        <th>결혼여부</th>
      </tr>
      </thead>
      <tbody>
      {% for user in users %}
      <tr>
        <td>{{user.id}}</td>
        <td>{{user.name}}</td>
        <td>{{user.age}}</td>
        <td>{{ '기혼' if user.married else '미혼'}}</td>
      </tr>
      {% endfor %}
      </tbody>
    </table>
    <br>
    <div>
      <form id="comment-form">
        <fieldset>
          <legend>댓글 등록</legend>
          <div><input id="userid" type="text" placeholder="사용자 아이디"></div>
          <div><input id="comment" type="text" placeholder="댓글"></div>
          <button type="submit">등록</button>
        </fieldset>
      </form>
    </div>
    <br>
    <table id="comment-list">
      <thead>
      <tr>
        <th>아이디</th>
        <th>작성자</th>
        <th>댓글</th>
        <th>수정</th>
        <th>삭제</th>
      </tr>
      </thead>
      <tbody></tbody>
    </table>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <script src="/mongoose.js"></script>
  </body>
</html>
```

```html
<h1>{{message}}</h1>
<h2>{{error.status}}</h2>
<pre>{{error.stack}}</pre>
```

```jsx
// 사용자 이름 눌렀을 때 댓글 로딩
document.querySelectorAll('#user-list tr').forEach((el) => {
  el.addEventListener('click', function () {
    const id = el.querySelector('td').textContent;
    getComment(id);
  });
});
// 사용자 로딩
async function getUser() {
  try {
    const res = await axios.get('/users');
    const users = res.data;
    console.log(users);
    const tbody = document.querySelector('#user-list tbody');
    tbody.innerHTML = '';
    users.map(function (user) {
      const row = document.createElement('tr');
      row.addEventListener('click', () => {
        getComment(user._id);
      });
      // 로우 셀 추가
      let td = document.createElement('td');
      td.textContent = user._id;
      row.appendChild(td);
      td = document.createElement('td');
      td.textContent = user.name;
      row.appendChild(td);
      td = document.createElement('td');
      td.textContent = user.age;
      row.appendChild(td);
      td = document.createElement('td');
      td.textContent = user.married ? '기혼' : '미혼';
      row.appendChild(td);
      tbody.appendChild(row);
    });
  } catch (err) {
    console.error(err);
  }
}
// 댓글 로딩
async function getComment(id) {
  try {
    const res = await axios.get(`/users/${id}/comments`);
    const comments = res.data;
    const tbody = document.querySelector('#comment-list tbody');
    tbody.innerHTML = '';
    comments.map(function (comment) {
      // 로우 셀 추가
      const row = document.createElement('tr');
      let td = document.createElement('td');
      td.textContent = comment._id;
      row.appendChild(td);
      td = document.createElement('td');
      td.textContent = comment.commenter.name;
      row.appendChild(td);
      td = document.createElement('td');
      td.textContent = comment.comment;
      row.appendChild(td);
      const edit = document.createElement('button');
      edit.textContent = '수정';
      edit.addEventListener('click', async () => { // 수정 클릭 시
        const newComment = prompt('바꿀 내용을 입력하세요');
        if (!newComment) {
          return alert('내용을 반드시 입력하셔야 합니다');
        }
        try {
          await axios.patch(`/comments/${comment._id}`, { comment: newComment });
          getComment(id);
        } catch (err) {
          console.error(err);
        }
      });
      const remove = document.createElement('button');
      remove.textContent = '삭제';
      remove.addEventListener('click', async () => { // 삭제 클릭 시
        try {
          await axios.delete(`/comments/${comment._id}`);
          getComment(id);
        } catch (err) {
          console.error(err);
        }
      });
      // 버튼 추가
      td = document.createElement('td');
      td.appendChild(edit);
      row.appendChild(td);
      td = document.createElement('td');
      td.appendChild(remove);
      row.appendChild(td);
      tbody.appendChild(row);
    });
  } catch (err) {
    console.error(err);
  }
}
// 사용자 등록 시
document.getElementById('user-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const name = e.target.username.value;
  const age = e.target.age.value;
  const married = e.target.married.checked;
  if (!name) {
    return alert('이름을 입력하세요');
  }
  if (!age) {
    return alert('나이를 입력하세요');
  }
  try {
    await axios.post('/users', { name, age, married });
    getUser();
  } catch (err) {
    console.error(err);
  }
  e.target.username.value = '';
  e.target.age.value = '';
  e.target.married.checked = false;
});
// 댓글 등록 시
document.getElementById('comment-form').addEventListener('submit', async (e) => {
  e.preventDefault();
  const id = e.target.userid.value;
  const comment = e.target.comment.value;
  if (!id) {
    return alert('아이디를 입력하세요');
  }
  if (!comment) {
    return alert('댓글을 입력하세요');
  }
  try {
    await axios.post('/comments', { id, comment });
    getComment(id);
  } catch (err) {
    console.error(err);
  }
  e.target.userid.value = '';
  e.target.comment.value = '';
});
```

```jsx
...
const connect = require('./schemas');
const indexRouter = require('./routes/index');
const usersRouter = require('./routes/users');
const commentsRouter = require('./routes/comments');

const app = express();
...
app.use(express.urlencoded({ extended: false }));

app.use('/', indexRouter);
app.use('/users', usersRouter);
app.use('/comments', commentsRouter);

app.use((req, res, next) => {
  const error =  new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
...
```

- 라우터 작성

```jsx
const express = require('express');
const User = require('../schemas/user');

const router = express.Router();

router.get('/', async (req, res, next) => {
  try {
    const users = await User.find({});
    res.render('mongoose', { users });
  } catch (err) {
    console.error(err);
    next(err);
  }
});

module.exports = router;
```

- `GET /`로 접속했을 때의 라우터
    - `User.find({})` 메서드로 모든 사용자를 찾은 뒤, mongoose.html을 렌더링할 때 users 변수로 넣음
    - `find` 메서드는 User 스키마를 require한 뒤 사용할 수 있음
    - 몽고디비의 `db.users.find({})` 쿼리와 같음

- 몽구스도 기본적으로 프로미스를 지원하므로 `async/await`과 `try/catch`문을 사용해서 각각 조회 성공 시와 실패 시의 정보를 얻을 수 있음

```jsx
const express = require('express');
const User = require('../schemas/user');
const Comment = require('../schemas/comment');

const router = express.Router();

router.route('/')
  .get(async (req, res, next) => {
    try {
      const users = await User.find({});
      res.json(users);
    } catch (err) {
      console.error(err);
      next(err);
    }
  })
  .post(async (req, res, next) => {
    try {
      const user = await User.create({
        name: req.body.name,
        age: req.body.age,
        married: req.body.married,
      });
      console.log(user);
      res.status(201).json(user);
    } catch (err) {
      console.error(err);
      next(err);
    }
  });

router.get('/:id/comments', async (req, res, next) => {
  try {
    const comments = await Comment.find({ commenter: req.params.id })
      .populate('commenter');
    console.log(comments);
    res.json(comments);
  } catch (err) {
    console.error(err);
    next(err);
  }
});

module.exports = router;
```

- `GET /users`와 `POST /users` 주소로 요청이 들어올 때의 라우터
    - 각각 사용자를 조회하는 요청과 등록하는 요청을 처리
    - `GET /`에서도 사용자 데이터를 조회했지만 `GET /users`에서는 데이터를 JSON 형식으로 반환한다는 점에서 차이
- 사용자를 등록할 때는 먼저 `모델 .create` 메서드로 저장
    - 몽고디비와 메서드가 다르므로 몽구스용 메서드를 따로 외워야 함
    - 정의한 스키마에 부합하지 않는 데이터를 넣었을 때는 몽구스가 에러를 발생시킴
    - `_id`는 자동으로 생성됨
- `GET /users/:id/comments` 라우터는 댓글 다큐먼트를 조회하는 라우터
    - `find` 메서드에는 옵션이 추가되어 있음
    - 먼저 댓글을 쓴 사용자의 아이디로 댓글을 조회한 뒤 `populate` 메서드로 관련 있는 컬렉션의 다큐먼트를 불러올 수 있음

```jsx
const express = require('express');
const Comment = require('../schemas/comment');

const router = express.Router();

router.post('/', async (req, res, next) => {
  try {
    const comment = await Comment.create({
      commenter: req.body.id,
      comment: req.body.comment,
    });
    console.log(comment);
    const result = await Comment.populate(comment, { path: 'commenter' });
    res.status(201).json(result);
  } catch (err) {
    console.error(err);
    next(err);
  }
});

router.route('/:id')
  .patch(async (req, res, next) => {
    try {
      const result = await Comment.updateOne({
        _id: req.params.id,
      }, {
        comment: req.body.comment,
      });
      res.json(result);
    } catch (err) {
      console.error(err);
      next(err);
    }
  })
  .delete(async (req, res, next) => {
    try {
      const result = await Comment.deleteOne({ _id: req.params.id });
      res.json(result);
    } catch (err) {
      console.error(err);
      next(err);
    }
  });

module.exports = router;
```

- 댓글에 관련된 CRUD 작업을 하는 라우터
- `POST /comments`, `PATCH /comments/:id`, `DELETE /comments/:id`를 등록
    - `POST /comments`
        - 다큐먼트를 등록하는 라우터
        - `Comment.create` 메서드로 댓글을 저장
        - populate 메서드로 프로미스의 결과로 반환된 comment 객체에 다른 컬렉션 다큐먼트를 불러옴
        - `path` 옵션으로 어떤 필드를 합칠지 설정
        - 합쳐진 결과를 클라이언트로 응답
    - `PATCH /comments/:id`
        - 다큐먼트를 수정하는 라우터
        - `updateOne` 메서드 사용
        - updateOne 첫 번째 인수로는 어떤 다큐먼트를 수정할지를 나타낸 쿼리 객체를 제공하고, 두 번째 인수로는 수정할 필드와 값이 들어 있는 객체를 제공
        - 시퀄라이즈와는 인수의 순서가 반대
        - 몽고디비와 다르게 `$set` 연산자를 사용하지 않아도 기입한 필드만 바꿈
    - `DELETE /comments/:id`
        - 다큐먼트를 삭제하는 라우터
        - `deleteOne` 메서드를 사용해 삭제
        - 어떤 다큐먼트를 삭제할지에 대한 조건을 첫 번째 인수에 넣음
- `npm start`로 웹 서버를 실행
- 서버 실행 후 http://localhost:3002에 접속