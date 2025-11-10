# 7장 : MySQL

- 모든 데이터를 변수에 저장했다는 것은 컴퓨터 메모리에 저장했다는 뜻
- 서버가 종료되면 메모리가 정리되면서 저장했던 데이터도 사라짐
- 이를 방지하기 위해서는 데이터베이스를 사용해야 함
- MySQL : SQL 언어를 사용하는 관계형 데이터베이스 관리 시스템의 대표 주자
- 몽고디비 : NoSQL의 대표주자

# 7.1 데이터베이스란?

- 데이터베이스 : 관련성을 가지며 중복이 없는 데이터들의 집합
- **DBMS(DataBase Management System)** : 데이터베이스를 관리하는 시스템
- 보통 서버의 하드 디스크나 SSD 등의 저장 매체에 데이터를 저장
- 서버에 데이터베이스를 올리면 여러 사람이 동시에 사용 가능
    - 사람들에게 각각 다른 권한을 줘서 어떤 사람은 읽기만 가능하고, 어떤 사람은 모든 작업을 가능하게 함
- **RDBMS(Relational DBMS)** : DBMS 중에서 가장 많이 사용
    - **Oracle**, **MySQL**, **MSSQL** 등
    - SQL이라는 언어를 사용해 데이터를 관리

# 7.4 데이터베이스 및 테이블 생성하기

## 7.4.1 데이터베이스 생성하기

- 데이터베이스를 생성하는 명령어 : `CREATE SCHEMA [데이터베이스명]`
    - MySQL에서 데이터베이스와 스키마는 같은 개념
- `CREATE SCHEMA` 뒤에 `DEFAULT CHARACTER SET utf8mb4 DEFAULT COLLATE utfmb4_general_ci`를 붙여 한글과 이모티콘을 사용할 수 있게 만듦
    - `utf8mb4`여야 데이터베이스에서 한글과 이모티콘을 사용할 수 있음
    - `COLLATE` : 해당 `CHARACTER SET`을 어떤 형식으로 정렬할 것인지를 의미
- SQL 구문을 입력할 때는 마지막에 세미콜론(;)을 붙여야 해당 구문이 실행됨
    - 세미콜론을 붙이지 않으면 프롬프트가 다음 줄로 넘어가서 다른 입력이 들어오기를 계속 기다림
- `CREATE SCHEMA`와 같이 MySQL이 기본적으로 알고 있는 구문은 **예약어**라고 함
    - 예약어는 소문자로 써도 되지만, 가급적 대문자로 쓰는 것이 좋음
    - nodejs와 같은 사용자가 직접 만든 이름과 구분하기 위해서임

## 7.4.2 테이블 생성하기

- 테이블 : 데이터가 들어갈 수 있는 틀
- MySQL 프롬프트에 다음과 같이 입력

```sql
CREATE TABLE nodejs.users ( // 테이블을 생성하는 명령어
id INT NOT NULL AUTO_INCREMENT,
name VARCHAR(20) NOT NULL,
age INT UNSIGNED NOT NULL,
married TINYINT NOT NULL,
comment TEXT NULL,
created_at DATETIME NOT NULL DEFAULT now(), // now 대신 CURRENT_TIMESTAMP 적용 가능
PRIMARY KEY(id),
UNIQUE INDEX name_UNIQUE (name ASC))
COMMENT = '사용자 정보'
ENGINE = InnoDB;
```

- 한 글자라도 잘못 입력하면 에러가 발생하니 조심해야 함
- 컬럼을 정의해두면 앞으로 데이터를 넣을 때 컬럼 규칙에 맞는 정보들만 넣을 수 있음
- 생년월일이나 몸무게와 같이 컬럼으로 만들어두지 않은 정보들은 저장할 수 없음

- 컬럼의 자료형
    - `INT` : 정수를 의미(소수까지 저장하고 싶은 경우 FLOAT이나 DOUBLE 자료형을 사용)
    - `VARCHAR(자릿수)`, `CHAR(자릿수)` : CHAR은 고정 길이이고, VARCHAR은 가변 길이
        
        (ex. CHAR(10)인 경우 반드시 길이가 10인 문자열만 넣어야 하고, VARCHAR(10)인 경우 길이가 0~10인 문자열을 넣을 수 있음, CHAR에 주어진 길이보다 짧은 문자열을 넣는다면 부족한 자릿수만큼 스페이스가 채워짐)
        
    - `TEXT` : 긴 글을 저장할 때 사용(수백 자 이상)
    - `TINYINT` : -128부터 127까지의 정수를 저장할 때 사용, 1 또는 0만 저장한다면 불 값(Boolean)과 같은 역할 가능
    - `DATETIME` : 날짜와 시간에 대한 정보를 담고 있음(날짜 정보만 담는 DATE와 시간 정보만 담는 TIME 자료형도 존재)
    - `NULL`/`NOT NULL` : 빈칸을 허용할지 여부를 묻는 옵션
    - `AUTO_INCREMENT` : 숫자를 저절로 올리겠다는 뜻
    - `UNSIGNED` : 숫자 자료형에 적용되는 옵션, 음수는 무시됨(FLOAT과 DOUBLE에는 적용 불가능)
    - `ZEROFILL` : 숫자의 자릿수가 고정되어 있을 때 사용 가능, 비어 있는 자리에 모두 0을 넣음
    - `DEFAULT` : 해당 컬럼에 값이 없을 때 MySQL이 기본값을 대신 넣음
    - `PRIMARY KEY` : **기본 키**(로우를 대표하는 고유한 값)
    - `UNIQUE INDEX` : 해당 값이 고유해야 하는지에 대한 옵션
    
- 테이블 자체에 대한 설정
    - `COMMENT` : 테이블에 대한 보충 설명을 의미(테이블이 무슨 역할을 하는지)
    - `ENGINE` : 여러 가지가 있지만, MyISAM과 InnoDB가 제일 많이 사용됨

- 테이블을 잘못 만들었을 경우 `DROP TABLE [테이블명]` 명령어를 입력하면 제거되고, 제거 후 다시 테이블을 생성하면 됨

- **외래 키(foreign key)** : 다른 테이블의 기본 키를 저장하는 칼럼
    - `CONSTRAINT [제약조건명] FOREIGN KEY [컬럼명] REFERENCES [참고하는 컬럼명]`으로 외래키를 지정할 수 있음
- `SHOW TABLES` 명령어를 통해 테이블이 제대로 생성되었는지 확인

# 7.5 CRUD 작업하기

- **CRUD** : **Create**, **Read**, **Update**, **Delete**의 첫 글자를 모은 두문자어, 데이터베이스에서 많이 수행하는 네 가지 작업

## 7.5.1 Create(생성)

- 데이터를 생성해서 데이터베이스에 넣는 작업
- 데이터를 넣는 명령어 : `INSERT INTO [테이블명] ([컬럼1], [컬럼2], ….) VALUES ([값1], [값2], …)`

## 7.5.2 Read(조회)

- 데이터베이스에 있는 데이터를 조회하는 작업
- 형식
    - 전체 조회 : `SELECT * FROM [테이블명]`
    - 특정 컬럼만 조회 : `SELECT [컬럼] FROM [테이블명]`
    - 특정 조건을 가진 데이터만 조회 : `SELECT [컬럼] FROM [테이블명] WHERE [조건]`
    - 조건들을 모두 만족 : `AND`
    - 조건들 중 어느 하나라도 만족 : `OR`
    - 정렬 : `ORDER BY [컬럼명] [ASC|DECS]` (ASC : 오름차순, DECS : 내림차순)
    - 조회할 로우 개수 설정 : `LIMIT [숫자]`
    - 건너뛸 숫자 설정 : `OFFSET [건너뛸 숫자]`

## 7.5.3 Update(수정)

- 데이터베이스에 있는 데이터를 수정하는 작업
- 수정 명령어 : `UPDATE [테이블명] SET [컬럼명=바꿀 값] WHERE [조건]`
    - ex. `UPDATE nodejs.users SET comment = '바꿀 내용' WHERE id = 2;`

## 7.5.4 Delete(삭제)

- 데이터베이스에 있는 데이터를 삭제하는 작업
- 삭제 명령어 : `DELETE FROM [테이블명] WHERE [조건]`
    - ex. `DELETE FROM nodejs.users WHERE id = 2;`

# 7.6 시퀄라이즈 사용하기

- **시퀄라이즈(Sequelize)** : MySQL 작업을 쉽게 할 수 있도록 도와주는 라이브러리
    - **ORM**(Object-relational Mapping, 자바스크립트 객체와 데이터베이스의 릴레이션을 매핑해주는 도구)으로 분류됨
    - 사용하는 이유 : 자바스크립트 구문을 알아서 SQL로 바꿔주기 때문
    - SQL 언어를 직접 사용하지 않고도 자바스크립트만으로 MySQL을 조작할 수 있으며, SQL을 몰라도 MySQL을 어느 정도 다룰 수 있게 됨

```jsx
{
	"name" : "learn-sequelize",
	"version" : "0.0.1",
	"description" : "시퀄라이즈를 배우자",
	"main" : "app.js",
	"scripts" : {
		"start" : "nodemon app"
	},
	"author" : "ZeroCho",
	"license" : "MIT"
}
```

- 시퀄라이즈에 필요한 sequelize, sequelize-cli, mysql2 패키지 설치 : `npm i express morgan nunjucks sequelize sequelize-cli mysql2`, `npm i -D nodemon`
    - **sequelize-cli** : 시퀄라이즈 명령어를 실행하기 위한 패키지
    - **mysql2** : MySQL과 시퀄라이즈를 이어주는 드라이버, 자체는 데이터베이스 프로그램이 아님
- 설치 완료 후 `sequelize` init 명령어 호출(전역 설치 없이 명령어로 사용하려면 앞에 npx를 붙임)

```jsx
const Sequelize = require('sequelize');

const env = process.env.NODE_ENV || 'development';
const config = require('../config/config')[env];
const db = {};

const sequelize = new Sequelize(config.database, config.username, config.password, config);
db.sequelize = sequelize;

module.exports = db;
```

## 7.6.1 MySQL 연결하기

- 시퀄라이즈를 통해 익스프레스 앱과 MySQL을 연결

```jsx
const express = require('express');
const path = require('path');
const morgan = require('morgan');
const nunjucks = require('nunjucks');

const {sequelize} = require('./models');

const app = express();
app.set('port', process.env.PORT || 3001);
app.set('view engine', 'html');
unujucks.configure('views', {
	express: app,
	watch: true,
});
sequelize.sync({ force: false })
	.then(() => {{
		console.log('데이터베이스 연결 성공');
	})
	.catch((err) => {
		console.error(err);
	});
	
app,use(morgan('dev'));
app.use(express.staric(path.join(__dirname, 'public')));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));

app.use((req, res, next) => {
	const error = new Error(`${req.method}} ${req.url} 라우터가 없습니다.`);
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

- `require('./models')`는 `require('./models/index.js')`와 같음(폴더 내의 index.js 파일은 require할 때 이름을 생략할 수 있음)
- db.sequelize를 불러와서 sync 메서드를 사용해 서버를 실행할 때 MySQL과 연동되도록 함
- `force: false` : 옵션을 true로 설정하면 서버를 실행할 때마다 테이블을 생성

- MySQL과 연동할 때는 config 폴더 안의 config.json 정보가 사용됨

```jsx
{
	"development": {
		"username": "root",
		"password": "[root 비밀번호]",
		"database": "nodejs",
		"host": "127.0.0.1",
		"dialect": "mysql"
	},
...
}
```

- password 속성에는 MySQL 비밀번호를 입력하고 database 속성에는 nodejs를 입력
- 이 설정은 process.env.NODE_ENV가 development일 때 적용됨
- 배포할 때는 process.env.NODE_ENV를 production으로 설정
- `npm start`로 서버를 실행하면 3001번 포트에서 서버가 돌아감
- `Executing (default): SELECT 1+1 AS result`, `데이터베이스 연결 성공` 이 두 로그가 뜨면 연결이 성공한 것

## 7.6.2 모델 정의하기

- MySQL의 테이블은 시퀄라이즈의 모델과 대응됨
- 시퀄라이즈는 모델과 MySQL의 테이블을 연결해주는 역할
- 시퀄라이즈는 기본적으로 모델 이름은 단수형으로, 테이블 이름은 복수형으로 사용

```jsx
const Sequelize = require('sequelize');

class User extends Sequelize.Model {
	static initiate(sequelize) {
		User.init({
			name: {
				type: Sequelize.STRING(20),
				allowNull: false,
				unique: true,
			},
			age: {
				type: Sequelize.INTEGER.UNSIGNED,
				allowNull: false,
			},
			married: {
				type: Sequelize.BOOLEAN,
				allowNull: false,
			},
			comment: {
				type: Sequelize.TEXT,
				allowNull: true,
			},
			created_at: {
				type: Sequelize.DATE,
				allowNull: false,
				defaultValue: Sequelize.NOW,
			},
		}, {
			sequelize,
			timestamps: false,
			underscored: false,
			modelName: 'User',
			tableName: 'Users',
			paranoid: false,
			charset: 'utf8',
			collate: 'utf8_general_ci',
		});
	}
	
	static associate(db) {}
};

module.exports = User;
			
```

- User 모델을 만들고 모듈로 exports함
    - User 모델은 Sequelize.Model을 확장한 클래스로 선언
- 모델은 크게 static initiate 메서드와 static associate 메서드로 나뉨
    - static initiate : 테이블에 대한 설정을 함
    - static associate : 다른 모델과의 관계를 적음
- 시퀄라이즈는 알아서 id를 기본 키로 연결하므로 id 컬럼은 적어줄 필요가 없음
- 시퀄라이즈의 자료형은 MySQL의 자료형과는 조금 다름

| MySQL | 시퀄라이즈 |
| --- | --- |
| `VARCHAR(100)` | `STRING(100)` |
| `INT` | `INTEGER` |
| `TINYINT` | `BOOLEAN` |
| `DATETIME` | `DATE` |
| `INT UNSIGNED` | `INTEGER.UNSIGNED` |
| `NOT NULL` | `allowNull: false` |
| `UNIQUE` | `unique: true` |
| `DEFAULT now()` | `defaultValue: Sequelize.NOW` |
- 모델.init 메서드의 두 번째 인수는 테이블 옵션
    - `sequelize` : static initiate 메서드의 매개변수와 연결되는 옵션, de.sequelize 객체를 넣어야 함, 나중에 model/index.js에서 연결
    - `timestamps` : 현재 값이 false로 되어 있음, true이면 시퀄라이즈는 createdAt과 updatedAt 컬럼을 추가
    - `modelName` : 모델 이름을 설정
    - `tableName` : 실제 데이터베이스의 테이블 이름이 됨, 기본적으로는 소문자 및 복수형으로 만듦
    - `paranoid` : true로 설정하면 deletedAt이라는 컬럼이 생김
    - `charset`과 `collate` : 각각 `utf8`과 `utf8_general_ci`로 설정해야 한글이 입력됨, 이모티콘까지 입력할 수 있게 하고 싶다면 `utf8mb4`, `utf8mb4_general_ci`를 각각 입력

## 7.6.3 관계 정의하기

- users 테이블과 comments 테이블이 있다고 할 때
    - 사용자 한 명은 댓글을 여러 개 작성할 수 있으나 댓글 하나에 사용자(작성자)가 여러 명일 수는 없음 → 이러한 관계를 **일대다(1:N)** 관계라고 함(사용자가 1이고, 댓글이 N)
- **일대일(1:1)** 관계 : 사용자 한 명은 자신의 정보를 담고 있는 테이블과만 관계가 있고 정보 테이블도 한 사람 만을 가리킴
- **다대다(N:M)** 관계 : 한 게시글에는 해시태그가 여러 개 달릴 수 있고, 한 해시태그도 여러 게시글에 달릴 수 있음
- MySQL에서는 JOIN이라는 기능으로 여러 테이블 간의 관계를 파악해 결과를 도출
- 시퀄라이즈는 JOIN 기능도 알아서 구현
    - 단, 테이블 가넹 어떠한 관계가 있는지 시퀄라이즈에 알려야 함

### 7.6.3.1 1:N

- 시퀄라이즈에서는 1:N 관계를 `hasMany` 라는 메서드로 표현
    - users 테이블의 로우 하나를 불러올 때 연결된 comments 테이블의 로우들도 같이 불러올 수 있음
- 반대로 `belongsTo` 메서드도 있음
    - comments 테이블의 로우를 불러올 때 연결된 users 테이블의 로우를 가져옴
- 모델 각각의 `static associate` 메서드에 넣음

```jsx
...
	static associate(db) {
		db.User.hasMany(db.Comment, {foreignKey: 'commenter', sourcekey: 'id' });
	}
};
```

```jsx
...
	static assciate(db) {
		db.Comment.belongsTo(db.User, {  foreignKey: 'commenter', targetKey: 'id' });
	}
};
```

- db라는 매개변수를 사용하는 이유 : 최상단에 `const Comment = require('./comment')` 식으로 불러올 경우, **순환 참조 문제**가 발생
    - comment.js에서 user.js를 require하는데 user.js에서도 comment.js를 require하면 문제가 발생하기 때문
    - **순환 참조** : 서로가 서로를 require하는 방식(자바스크립트에서는 **지양**해야 하는 방식)

- 어떤 모델에 hasMany를 쓰고, 어떤 모델에 belongsTo를 쓰는지 구분 : 다른 모델의 정보가 들어가는 테이블에 belongsTo를 사용
- hasMany 메서드에서는 sourceKey 속성에 id를 넣고, belongsTo 메서드에서는 targetKey 속성에 id를 넣음
- foreignKey를 따로 지정하지 않는다면, 이름이 ‘모델명+기본 키’인 컬럼이 모델에 생성됨
    - commenter를 foriegnKey로 직접 넣어주지 않았다면 **user(모델명)+기본 키(id)**가 합쳐진 UserId가 foriegnKey로 생성됨

### 7.6.3.2 1:1

- 1:1 관계에서는 hasMany 메서드 대신 `hasOne` 메서드를 사용

```jsx
db.User.hasOne(db.Info, { foreignKey: 'UserId', sourceKey: 'id' });
db.Info.belongsTo(db.User, { foreignKey: 'UserId', targetKey: 'id' });
```

- 1:1 관계라고 해도 belongsTo와 hasOne이 반대이면 안 됨
    - belongsTo를 사용하는 Info 모델에 UserId 컬럼이 추가되기 때문

### 7.6.3.3 N:M

- 시퀄라이즈에는 N:M 관계를 표현하기 위한 `belongsToMany` 메서드가 있음
- 게시글 정보를 담고 있는 가상의 Post 모델과 해시태그 정보를 담고 있는 가상의 Hashtag 모델이 있다고 할 경우

```jsx
db.Post.belongsToMany(db.Hashtag, { through: 'PostHashtag' });
db.Hashtag.belongsToMany(db.Post, { through: 'PostHashtag' });
```

- 양쪽 모델에 모두 belongsToMany 메서드를 사용하며, N:M 관계 특성상 새로운 모델이 생성됨
    - through 속성에 그 이름을 적으면 됨
    - 새로 생성된 PostHashtag 모델에는 게시글과 해시태그의 아이디가 저장됨

## 7.6.4 쿼리 알아보기

- 시퀄라이즈로 CRUD 작업을 하려면 먼저 시퀄라이즈 쿼리를 알아야 함
    - SQL문을 자바스크립트로 생성하는 것이라 시퀄라이즈만의 방식이 있음
- 쿼리는 프로미스를 반환하므로 then을 붙여 결괏값을 받을 수 있음
- async/await 문법과 같이 사용할 수도 있음

```sql
INSERT INTO nodejs.users (name, age, married, comment) VALUES ('zero', 24, 0, '자기소개 1');
```

```jsx
const { User } = require('../models');
User.create({
	name: 'zero',
	age: 24,
	married: false,
	comment: '자기소개1',
});
```

- 데이터를 넣을 때 MySQL의 자료형이 아니라 시퀄라이즈 모델에 정의한 자료형대로 넣어야 함
    - 시퀄라이즈가 알아서 MySQL 자료형으로 바꿈
    - 자료형이나 옵션에 부합하지 않는 데이터를 넣었을 때는 시퀄라이즈가 에러를 발생시킴

- users 테이블의 모든 데이터를 조회하는 SQL문

```sql
SELECT * FROM nodejs.users;
```

```jsx
User.findAll({});
```

- users 테이블의 데이터 하나만 가져오는 SQL문

```sql
SELECT * FROM nodejs.users LIMIT 1;
```

```jsx
User.findOne({});
```

- attributes 옵션을 사용해서 원하는 컬럼만 가져올 수 있음

```sql
SELECT name, married FROM nodejs.users;
```

```jsx
User.findAll({
	attributes: ['name', 'married'],
});
```

- where 옵션이 조건들을 나열하는 옵션

```sql
SELECT name, age FROM nodejs.users WHERE married = 1 AND age > 30;
```

```jsx
const { Op } = require('sequelize');
const { User } = require('../models');
User.findAll({
	attributes: ['name', 'age'],
	where: {
		married: true,
		age: { [Op.gt]: 30 },
	},
});
```

- MySQL에서는 undefined라는 자료형을 지원하지 않으므로 where 옵션에는 undefined가 들어가면 안됨(빈 값을 넣고자 하면 null을 대신 사용)
- 시퀄라이즈는 자바스크립트 객체를 사용해서 쿼리를 생성해야 하므로 `Op.gt` 같은 특수한 연산자들이 사용됨
    - `Op.gt`(초과), `Op.gte`(이상), `Op.lt`(미만), `Op.lte`(이하), `Op.ne`(같지 않음), `Op.or`(또는), `Op.in`(배열 요소 중 하나), `Op.notIn`(배열 요소와 모두 다름) 등

- 시퀄라이즈의 정렬 방식 : `order` 옵션으로 가능
    - 배열 안에 배열이 있다는 점도 주의(정렬은 꼭 컬럼 하나로만 하는 게 아니라 컬럼 두 개 이상으로 할 수도 있기 때문)

- 조회할 로우 개수 설정
    - LIMIT 1인 경우에는 findAll 대신 findOne 메서드를 사용해도 되지만, 다음과 같이 `limit` 옵션으로 할 수도 있음
    - OFFSET도 역시 `offset` 속성으로 구현 가능

```sql
SELECT id, name FROM users ORDER BY age DESC LIMIT 1 OFFSET 1;
```

```jsx
User.findAll({
	attributes: ['id', 'name'],
	order: ['age', 'DESC'],
	limit: 1,
	offset: 1,
});
```

- 로우를 수정하는 쿼리 : `update` 메서드로 수정 가능

```sql
UPDATE nodejs.users SET comment = '바꿀 내용' WHERE id = 2;
```

```jsx
User.update({
	comment: '바꿀 내용', // 첫 번째 인수 : 수정할 내용
}, {
	where: {id: 2}, // 두 번째 인수 : 어떤 로우를 수정할지에 대한 조건
});
```

- 로우를 삭제하는 쿼리 : `destroy` 메서드로 삭제(where 옵션에 조건들을 적음)

```sql
DELETE FROM nodejs.users WHERE id = 2;
```

```jsx
User.destroy({
	where: {id: 2}, // 두 번째 인수 : 어떤 로우를 수정할지에 대한 조건
});
```

### 7.6.4.1 관계 쿼리

- `findOne`이나 `findAll` 메서들을 호출할 때 프로미스의 결과로 모델을 반환
- `findAll`은 모두 찾는 것이므로 모델의 배열을 반환

```jsx
const user = await User.findOne({});
console.log(user.nick); // 사용자 닉네임
```

- 특정 사용자를 가져오면서 그 사람의 댓글까지 모두 가져오고 싶은 경우

```jsx
const user = await User.findOne({
	include: [{ // 어떤 모델과 관계가 있는지를 배열에 넣어줌
		model: Comment,
	}]
});
console.log(user.Comments); // 사용자 댓글
```

- 배열인 이유 : 다양한 모델과 관계가 있을 수 있기 때문
- 댓글은 여러 개일 수 있으므로 `(hasMany) user.Comments` 로 접근 가능
- 관계를 설정했다면 `getComments`(조회) 외에도 `setComments`(수정), `addComment`(하나 생성), `addComments`(여러 개 생성), `removeComments`(삭제) 메서드를 지원(동사 뒤에 모델의 이름이 붙는 형식)

- 동사 뒤의 모델 이름을 바꾸고 싶다면 관계를 설정할 때 `as` 옵션을 사용할 수 있음

```jsx
// 관계 설정할 때 as로 등록
db.User.hasMany(db.Comment, {foreignKey: 'commenter', sourceKey: 'id', as: 'Answers'
});

// 쿼리할 때는
const user = await User.findOne({});
const comments = await user.getAnswers();
console.log(comments); // 사용자 댓글
```

- include나 관계 쿼리 메서드에도 `where`이나 `attributes` 같은 옵션 사용 가능

```jsx
const user = await User.fineOne({
	include: [{
		model: Comment,
		where: {
			id: 1,
		},
		attributes: ['id'],
	}]
});
// 또는
const comments = await user.getComments({
	where: {
		id: 1,
	},
	attributes: ['id'],
});
```

- 관계 쿼리 시 수정, 생성, 삭제

```jsx
const user = await User.findOne({});
const comment = await Comment.create();
await user.addComment(comment);
// 또는
await user.addComment(comment.id);

// 여러 개를 추가할 때는 배열로 추가할 수 있음
const user = await User.findOne({});
const comment1 = await Comment.create();
const comment2 = await Comment.create();
await user.addComment([comment1, comment2]);
```

- 관계 쿼리 메서드의 인수로 추가할 댓글 모델을 넣거나 댓글의 아이디를 넣으면 됨(수정이나 삭제도 마찬가지)

### 7.6.4.2 SQL 쿼리하기

- 시퀄라이즈의 쿼리를 사용하기 싫거나 어떻게 할지 모르겠을 때 직접 SQL문을 통해 쿼리할 수도 있음

```jsx
const [result, metadata] = await sequelize.query('SELECT * from comments');
console.log(result);
```