# 9장 : 익스프레스로 SNS 서비스 만들기

# 9.1 프로젝트 구조 갖추기

- nodebird라는 폴더를 만들기
- 항상 package.json을 제일 먼저 생성해야 함
    - npm init 명령어를 콘솔에서 호출해도 되고 직접 만들어도 됨
    - version, desciprtion, author, license는 원하는 대로 자유롭게 수정할 수 있음
    - scripts 부분에 start 속성은 잊지 말고 넣어줘야 함

```json
{
	"name": "nodebird",
	"version": "0.0.1",
	"description": "익스프레스로 만드는 SNS 서비스",
	"main": "app.js",
	"scripts": {
		"start": "nodemon app"
	},
	"author": "ZeroCho",
	"license": "MIT"
}
```

- 시퀄라이즈 설치
    - `npm i sequelize mysql2 seqielize-cli`  : node_moduls 폴더와 package-lock.json이 생성됨
    - `npx sequelize init`  : config, migrations, models, seeders 폴더가 생성됨(npx 명령어를 사용하는 이유 : 전역 설치(npm i -g)를 피하기 위해서)
    - 사용자와 게시물 간, 게시물과 해시태그 간의 관계가 중요하므로 관계형 데이터베이스인 MySQL 선택
- 템플릿 파일을 넣을 views 폴더, 라우터를 넣을 routes 폴더, 정적 파일을 넣을 public 폴더, passport 패키지를 위핸 passport 폴더를 생성
- 익스프레스 서버 코드가 담길 app.js와 설정값들을 담을 .env 파일을 nodebird 폴더 안에 생성

- 필요한 npm 패키지들을 설치하고 app.js를 작성
    - `npm i express cookie-parser express-session morgan multer dotenv nunjucks`
    - `npm i -D nodemon`
- 템플릿 엔진은 넌적스를 사용

```jsx
const express = require('express');
const cookieParser = require('cookie-parser');
const morgan = require('morgan');
const path = require('path');
const session = require('express-session');
const nunjucks = require('nunjucks');
const dotenv = require('dotenv');

dotenv.config();
const pageRouter = require('./routes/page');
const app = express();
app.set('port', process.env.PORT || 8001); // 8001번 포트에 연결
app.set('view engine', 'html');
nunjucks.configure('views', {
	express: app,
	watch: true,
});

app.use(morgan('dev'));
app.use(express.static(path.join(__dirname, 'public')));
app.use(express.json());
app.use(express.urlcoded({ extended: false }));
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

app.use('/', pageRouter); // 라우터

app.use((req, res, next) => { // 404 응답 미들웨어
	const error = new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
	error.status = 404;
	next(error);
});
	
app.use((err, req, res, next) => { // 에러 처리 미들웨어
	res.locals.message = err.message;
	res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
	res.status(err.status || 500);
	res.render('error');
});

app.listen(app.get('port'), () => {
	console.log(app.get('port'), '번 포트에서 대기 중');
});
```

```bash
COOKIE_SECRET=cookiesecret
```

- 하드 코딩된 비밀번호가 유일하게 남아 있는 파일 : **config.json**(시퀄라이즈 설정을 담아 둠)
    - JSON 파일이라 process.env를 사용할 수 없음

- 기본적인 라우터와 템플릿 엔진 만들기
    - routes 폴더 안에는 page.js를, views 폴더 안에는 layout.html, main.html, profile.html, join.html, error.html을 생성
    - public 폴더 안에 디자인을 위한 main.css 생성

```jsx
const express = require('express');
const { renderProfile, renderJoin, renderMain } = require('../controllers/page');

const router = express.Router();

router.use((req, res, next) => {
// 라우터용 미들웨어를 만들어 템플릿 엔진에서 사용할 변수를 res.locals로 설정
// (이유 : 모든 템플릿 엔진에서 공통으로 사용하기 때문)
	res.locals.user = null;
	res.locals.followerCount = 0;
	res.locals.followingCount = 0;
	res.locals.followingIdList = [];
	next();
});
// 컨트롤러 역할을 하는 세 함수 : renderProfile, renderJoin, renderMain
router.get('/profile', renderProfile);

router.get('/join', renderJoin);

router.get('/', renderMain);

module.exports = router;
```

- **컨트롤러** : 라우터 마지막에 위치해 클라이언트에 응답을 보내는 미들웨어
- 프로젝트에 controllers 폴더를 만들고 그 안에 page.js 생성

```jsx
exports.renderProfile = req, res) => { // 정보 페이지를 화면에 렌더링
	res.render('profile', { title: '내 정보 - NodeBird' });
};

exports.renderJoin = (req, res) => { // 회원 가입 페이지를 화면에 렌더링
	res.render('join', { title: '회원 가입 - NodeBird' });
};

exports.renderMain = (req, res, next) => {
// 메인 페이지를 렌더링하면서 넌적스에 twits(게시글 목록)를 전달
	const twits = []; // 지금은 빈 배열, 나중에 값을 넣음
	res.render('main', {
		title: 'NodeBird',
		twits,
	});
};
```

- 실무에서 코드를 편하게 관리하게 위해 컨트롤러를 따로 분리

- 클라이언트 코드 작성

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>{{title}}</title>
    <meta name="viewport" content="width=device-width, user-scalable=no">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <link rel="stylesheet" href="/main.css">
  </head>
  <body>
    <div class="container">
      <div class="profile-wrap">
        <div class="profile">
          {% if user and user.id %} <!-- 렌더링할 때 user가 존재하면 -->
            <div class="user-name">{{'안녕하세요! ' + user.nick + '님'}}</div>
            <div class="half">
              <div>팔로잉</div>
              <div class="count following-count">{{followingCount}}</div>
            </div>
            <div class="half">
              <div>팔로워</div>
              <div class="count follower-count">{{followerCount}}</div>
            </div>
          <input id="my-id" type="hidden" value="{{user.id}}">
          <a id="my-profile" href="/profile" class="btn">내 프로필</a>
          <a id="logout" href="/auth/logout" class="btn">로그아웃</a>
        {% else %} <!-- user가 존재하지 않으면 -->
          <form id="login-form" action="/auth/login" method="post">
            <div class="input-group">
              <label for="email">이메일</label>
              <input id="email" type="email" name="email" required autofocus>
            </div>
            <div class="input-group">
              <label for="password">비밀번호</label>
              <input id="password" type="password" name="password" required>
            </div>
            <a id="join" href="/join" class="btn">회원가입</a>
            <button id="login" type="submit" class="btn">로그인</button>
            <a id="kakao" href="/auth/kakao" class="btn">카카오톡</a>
          </form>
        {% endif %}
        </div>
        <footer>
          Made by&nbsp;
          <a href="https://www.zerocho.com" target="_blank">ZeroCho</a>
        </footer>
      </div>
      {% block content %}
      {% endblock %}
    </div>
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <script>
      window.onload = () => {
        if (new URL(location.href).searchParams.get('error')) {
          alert(new URL(location.href).searchParams.get('error'));
        }
      };
    </script>
    {% block script %}
    {% endblock %}
  </body>
</html>
```

```html
{% extends 'layout.html' %}

{% block content %}
    <div class="timeline">
      {% if user %} <!-- user 변수가 존재할 때 : 게시글 업로드 폼을 보여줌 -->
        <div>
          <form id="twit-form" action="/post" method="post" enctype="multipart/form-data">
            <div class="input-group">
              <textarea id="twit" name="content" maxlength="140"></textarea>
            </div>
            <div class="img-preview">
              <img id="img-preview" src="" style="display: none;" width="250" alt="미리보기">
              <input id="img-url" type="hidden" name="url">
            </div>
            <div>
              <label id="img-label" for="img">사진 업로드</label>
              <input id="img" type="file" accept="image/*">
              <button id="twit-btn" type="submit" class="btn">짹짹</button>
            </div>
          </form>
        </div>
      {% endif %}
      <div class="twits">
        <form id="hashtag-form" action="/hashtag">
          <input type="text" name="hashtag" placeholder="태그 검색">
          <button class="btn">검색</button>
        </form>
        {% for twit in twits %}
        <!-- 렌더링 시 twits 배열 안의 요소들을 읽어서 게시글로 만듦 -->
          <div class="twit">
            <input type="hidden" value="{{twit.User.id}}" class="twit-user-id">
            <input type="hidden" value="{{twit.id}}" class="twit-id">
            <div class="twit-author">{{twit.User.nick}}</div>
            {% if not followingIdList.includes(twit.User.id) and twit.User.id !== user.id %} <!-- 나의 팔로잉 아이디 목록에 게시글 작성자의 아이디가 없는 경우 : 팔로우 버튼을 보여줌, 게시글 작성자가 나인 경우 : 나를 팔로우할 수는 없게 함(넌적스 문법) -->
              <button class="twit-follow">팔로우하기</button>
            {% endif %}
            <div class="twit-content">{{twit.content}}</div>
            {% if twit.img %}
              <div class="twit-img"><img src="{{twit.img}}" alt="섬네일"></div>
            {% endif %}
          </div>
        {% endfor %}
      </div>
    </div>
{% endblock %}

{% block script %}
  <script>
    if (document.getElementById('img')) {
      document.getElementById('img').addEventListener('change', function(e) {
        const formData = new FormData();
        console.log(this, this.files);
        formData.append('img', this.files[0]);
        axios.post('/post/img', formData)
          .then((res) => {
            document.getElementById('img-url').value = res.data.url;
            document.getElementById('img-preview').src = res.data.url;
            document.getElementById('img-preview').style.display = 'inline';
          })
          .catch((err) => {
            console.error(err);
          });
      });
    }
    document.querySelectorAll('.twit-follow').forEach(function(tag) {
      tag.addEventListener('click', function() {
        const myId = document.querySelector('#my-id');
        if (myId) {
          const userId = tag.parentNode.querySelector('.twit-user-id').value;
          if (userId !== myId.value) {
            if (confirm('팔로잉하시겠습니까?')) {
              axios.post(`/user/${userId}/follow`)
                .then(() => {
                  location.reload();
                })
                .catch((err) => {
                  console.error(err);
                });
            }
          }
        }
      });
    });
  </script>
{% endblock %}
```

```html
{% extends 'layout.html' %}
<!-- 사용자의 팔로워와 사용자가 팔로잉 중인 목록을 보여줌 -->
{% block content %}
  <div class="timeline">
    <div class="followings half">
      <h2>팔로잉 목록</h2>
      {% if user.Followings %}
        {% for following in user.Followings %}
          <div>{{following.nick}}</div>
        {% endfor %}
      {% endif %}
    </div>
    <div class="followers half">
      <h2>팔로워 목록</h2>
      {% if user.Followers %}
        {% for follower in user.Followers %}
          <div>{{follower.nick}}</div>
        {% endfor %}
      {% endif %}
    </div>
  </div>
{% endblock %}
```

```html
{% extends 'layout.html' %}
<!-- 회원가입하는 폼을 보여줌 -->
{% block content %}
  <div class="timeline">
    <form id="join-form" action="/auth/join" method="post">
      <div class="input-group">
        <label for="join-email">이메일</label>
        <input id="join-email" type="email" name="email"></div>
      <div class="input-group">
        <label for="join-nick">닉네임</label>
        <input id="join-nick" type="text" name="nick"></div>
      <div class="input-group">
        <label for="join-password">비밀번호</label>
        <input id="join-password" type="password" name="password">
      </div>
      <button id="join-btn" type="submit" class="btn">회원가입</button>
    </form>
  </div>
{% endblock %}

{% block script %}
  <script>
    window.onload = () => {
      if (new URL(location.href).searchParams.get('error')) {
        alert('이미 존재하는 이메일입니다.');
      }
    };
  </script>
{% endblock %}
```

```html
{% extends 'layout.html' %}
<!-- 서버에 에러가 발생했을 때 에러 내역을 보여줌(에러는 콘솔로 봐도 되지만 브라우저 화면으로 보면 좀 더 편리, 단 배포 시에는 에러 내용을 보여주지 않는 게 보안상 좋음 -->
{% block content %}
  <h1>{{message}}</h1>
  <h2>{{error.status}}</h2>
  <pre>{{error.stack}}</pre>
{% endblock %}
```

```css
* { box-sizing: border-box; }
html, body { margin: 0; padding: 0; height: 100%; }
.btn {
  display: inline-block;
  padding: 0 5px;
  text-decoration: none;
  cursor: pointer;
  border-radius: 4px;
  background: white;
  border: 1px solid silver;
  color: crimson;
  height: 37px;
  line-height: 37px;
  vertical-align: top;
  font-size: 12px;
}
input[type='text'], input[type='email'], input[type='password'], textarea {
  border-radius: 4px;
  height: 37px;
  padding: 10px;
  border: 1px solid silver;
}
.container { width: 100%; height: 100%; }
@media screen and (min-width: 800px) {
  .container { width: 800px; margin: 0 auto; }
}
.input-group { margin-bottom: 15px; }
.input-group label { width: 25%; display: inline-block; }
.input-group input { width: 70%; }
.half { float: left; width: 50%; margin: 10px 0; }
#join { float: right; }
.profile-wrap {
  width: 100%;
  display: inline-block;
  vertical-align: top;
  margin: 10px 0;
}
@media screen and (min-width: 800px) {
  .profile-wrap { width: 290px; margin-bottom: 0; }
}
.profile {
  text-align: left;
  padding: 10px;
  margin-right: 10px;
  border-radius: 4px;
  border: 1px solid silver;
  background: lightcoral;
}
.user-name { font-weight: bold; font-size: 18px; }
.count { font-weight: bold; color: crimson; font-size: 18px; }
.timeline {
  margin-top: 10px;
  width: 100%;
  display: inline-block;
  border-radius: 4px;
  vertical-align: top;
}
@media screen and (min-width: 800px) { .timeline { width: 500px; } }
#twit-form {
  border-bottom: 1px solid silver;
  padding: 10px;
  background: lightcoral;
  overflow: hidden;
}
#img-preview { max-width: 100%; }
#img-label {
  float: left;
  cursor: pointer;
  border-radius: 4px;
  border: 1px solid crimson;
  padding: 0 10px;
  color: white;
  font-size: 12px;
  height: 37px;
  line-height: 37px;
}
#img { display: none; }
#twit { width: 100%; min-height: 72px; }
#twit-btn {
  float: right;
  color: white;
  background: crimson;
  border: none;
}
.twit {
  border: 1px solid silver;
  border-radius: 4px;
  padding: 10px;
  position: relative;
  margin-bottom: 10px;
}
.twit-author { display: inline-block; font-weight: bold; margin-right: 10px; }
.twit-follow {
  padding: 1px 5px;
  background: #fff;
  border: 1px solid silver;
  border-radius: 5px;
  color: crimson;
  font-size: 12px;
  cursor: pointer;
}
.twit-img { text-align: center; }
.twit-img img { max-width: 75%; }
.error-message { color: red; font-weight: bold; }
#search-form { text-align: right; }
#join-form { padding: 10px; text-align: center; }
#hashtag-form { text-align: right; }
footer { text-align: center; }
```

- `npm start`로 서버를 실행하고 http://localhost:8001에 접속하면 화면이 나타남

# 9.2 데이터베이스 세팅하기

- 로그인 기능이 있으므로 사용자 테이블이 필요
- 게시글을 저장할 게시글 테이블도 필요
- 해시태그를 사용하므로 해시태그 테이블도 필요
- models 폴더 안에 user.js와 post.js, hashtag.js를 생성

```jsx
// 사용자 정보를 저장하는 모델
const Sequelize = require('sequelize');

class User extends Sequelize.Model {
	static initiate(sequelize) {
		Use.init({
			emai: { // 이메일 저장
				type: Sequelize.STRING(40),
				allowNull: true,
				unique: true,
			},
			nick: { // 닉네임 저장
				type: Sequelize.STRING(15),
				allowNull: false,
			},
			password: { // 비밀번호 저장
				type: Sequelize.STRING(100),
				allowNull: true,
			},
			provider: { // SNS 로그인을 했을 경우에는 provider와 snsId를 저장
				type: Sequelize.ENUM('local', 'kakao'),
				// ENUM : 넣을 수 있는 값을 제한
				// 종류로는 이메일/비밀번호 로그인(local)이나 카카로 로그인(kakao) 둘 중 하나만 선택할 수 있게 함, 어겼을 때 에러 발생
				allowNull: false,
				defaultValue: 'local', // 기본적으로 이메일/비밀번호 로그인이라고 가정
			},
			snsId: {
				type: Sequelize.STRING(30),
				allowNull: true,
			},
		}, {
			sequelize,
			// timestamps와 paranoid가 true로 주어졌으므로 createdAt, updatedAt, deletedAt 컬럼도 생성됨
			timestamps: true,
			underscored: false,
			modelName: 'User',
			tableName: 'users',
			paranoid: true,
			charset: 'utf8',
			collate: 'utf8_general_ci',
		});
	}
	
	static associate(db) {}
};

module.exports = User;
```

```jsx
// 게시글 모델 : 게시글 내용과 이미지 경로를 저장
// 게시글 등록자의 아이디를 담은 컬럼은 나중에 관계를 설정할 때 시퀄라이즈가 알아서 생성 
const Sequelize = require('sequelize');

class Post extends Sequelize.Model {
	static initiate(sequelize) {
		Post.init({
			content: {
				type: Sequelize.STRING(140),
				allowNull: false,
			},
			img: {
				type: Sequelize.STRING(200),
				allowNull: true,
			},
		}, {
			sequelize,
			timestamps: true,
			underscored: false, 
			modelName: 'Post',
			tableName: 'posts',
			paranoid: false,
			charset: 'utf8mb4',
			collate: 'utf8mb4_general_ci',
		});
	}
	
	static associate(db) {}
}

module.exports = Post;
```

```jsx
// 해시태그 모델 : 태그 이름을 저장
// 해시태그 모델을 따로 두는 이유 : 나중에 태그로 검색하기 위해서
const Sequelize = require('sequelize');

class Hashtag extends Sequelize.Model {
	static initiate(sequelize) {
		Hashtag.init({
			title: {
				type: Sequelize.STRING(15),
				allowNull: false,
				unique: true,
			},
		}, {
			sequelize,
			timestamps: true,
			underscored: false,
			modelName: 'Hashtag',
			tableName: 'hashtags',
			paranoid: false,
			charset: 'utf8mb4',
			collate: 'utf8mb4_general_ci',
		});
	}
	
	static associate(db) {}
};

module.exports = Hashtag;
```

- 생성한 모델들을 시퀄라이즈에 등록
- models/index.js에는 시퀄라이즈가 자동으로 생성한 코드들이 존재하는데, 그것을 통째로 바꿔줌

```jsx
const Sequelize = require('sequelize');
const User = require('./user');
const Post = require('./post');
const Hashtag = require('./hashtag');
const env = process.env.NODE_ENV || 'development';
const config = require('../config/config')[env];

const db = {};
const sequelize = new Sequelize(
	config.database, config.username, config.password, config,
);

db.sequelize = sequelize;
db.User = User;
db.Post = Post;
db.Hashtag = Hashtag;

User.initiate(sequelize);
Post.initiate(sequelize);
Hashtag.initiate(sequelize);

User.associate(db);
Post.associate(db);
Hashtag.associate(db);

module.exports = db;
```

- 모델이 많이 늘어나면 initiate와 associate 부분도 따라서 늘어날 수 있음
- 실무에서는 모델이 100개가 넘어가는 경우도 흔하므로 이럴 때에는 다음과 같이 작성하면 좋음

```jsx
const Sequelize = require('sequelize');
const fs = require('fs');
const path = require('path');
const env = process.env.NODE_ENV || 'development';
const config = require('../config/config')[env];

const db = {};
const sequelize = new Sequelize{
	config.database, config.username, config.password, config,
);

db.sequelize = sequelize;

const basename = path.basename(__filename_);
fs
	.readdirSync(__dirname) // 현재 폴더의 모든 파일을 조회
	.filter(file => { // 숨김 파일, index.js, js 확장자가 아닌 파일 필터링)
		return (file.indexOf('.') !== 0) && (file !== basename) && (file.slice(-3) == '.js');
	})
	.forEach(file => { // 해당 파일의 모델을 불러와서 init
		const model = require(path.join(__dirname, file));
		console.log(file, model.name);
		db[model.name] = modell
		model.initiate(seqielize);
	});
	
Object.keys(db).forEach(modelName => { // associate 호출
	if (db[modelName]associate) {
		db[modelName].associate(db);
	}
});

module.exports = db;
```

- 이 코드는 npx sequelize init 명령어를 수행했을 때 자동으로 생성되는 models/index.js와 거의 비슷
- 모델이 무수히 많더라도 자동으로 시퀄라이즈가 모델을 파악할 수 있게 됨
- 단점 : models 폴더에 미완성 모델이 있을 때 해당 모델도 시퀄라이즈가 읽어들여 연결해버림(미완성 테이블이 생길 수도 있는 것)
- models 폴더에 모델이 아닌 다른 파일을 넣지 않도록 주의해야 함(model.initiate나 model.associate 메서드가 존재하지 않아 에러가 발생하게 됨)

- 각 모델 간의 관계를 associate 함수 내에 정의

```jsx
...
	static associate(db) {
		db.User.hasMany(db.Post);
		db.User.belongsToMany(db.User, {
			foreignKey: 'followingId',
			as: 'Followers',
			through: 'Follow',
		});
		db.User.belongsToMany(db.User, {
			foreignkey: 'followerId',
			as: 'Followings',
			through: 'Follow',
		});
	}
};
```

- User 모델과 Post 모델은 1(User) : N(Post) 관계에 있으므로 hasMany로 연결되어 있음
- user.getPosts, user.addPosts 같은 관계 메서드들이 생성됨

- 같은 모델끼리도 N : M 관계를 가질 수 있음
    - ex. 팔로잉 기능 : 사용자 한 명이 팔로워를 여러 명 가질 수도 있고, 한 사람이 여러 명을 팔로잉할 수도 있음(User 모델과 User 모델 간에 N : M 관계가 있는 것)
- 같은 테이블 간 N : M 관계에서는 모델 이름과 컬럼 이름을 따로 정해야 함
    - through 옵션을 사용해 생성할 모델 이름을 Follow로 정함
- Follow 모델에서 사용자 아이디를 저장하는 칼럼 이름이 둘 다 UserId이면 누가 팔로워이고 누가 팔로잉 중인지 구분되지 않으므로 따로 설정해야 함
    - foreignKey 옵션에 각각 followerId, followingId를 넣어줘서 두 사용자 아이디를 구별
- 같은 테이블 간의 N : M 관계에서는 as 옵션도 넣어야 함(둘 다 User 모델이라 구분되지 않기 때문)
- as는 foreignKey와 반대되는 모델을 가리킴
    - foreignKey가 followerId(팔로워 아이디)이면 as는 Followings(팔로잉)
    - foreignKey가 followingId(팔로잉 아이디)이면 as는 Followers(팔로워)여야함
    - 팔로워(Followers)를 찾으려면 먼저 팔로잉하는 사람의 아이디(followingId)를 찾아야 하는 것이라고 생각하면 됨

- Post 모델 작성

```jsx
...
	static associate(db) {
		db.Post.belongsTo(db.User);
		db.Post.belongsToMany(db.Hashtag, { through: 'PostHashtag' });
	}
};
```

- User 모델과 Post 모델은 1(User) : N(Post) 관계이므로 belongsTo로 연결되어 있음
- 시퀄라이즈는 Post 모델에 User 모델의 id를 가리키는 UserId 컬럼을 추가
- belongsTo는 게시글에 붙음
- post.getUser, post.addUser와 같은 관계 메서드가 생성됨

- Post 모델과 Hashtag 모델은 N : M 관계
- PostHashtag라는 중간 모델이 생기고, 각각 postId와 hashtagId라는 foreignKey도 추가됨
- as는 따로 지정하지 않았으니 post.getHashtags, post.addHashtags, hashtags.getPosts 같은 기본 이름의 관계 메서드들이 생성됨

```jsx
...
	static associate(db) {
		db.Hashtag.belongsToMany(db.Post, { through: 'PostHashtag' });
	}
};
```

- NodeBird의 모델(5개)
    - User, Hashtag, Post : 직접 생성
    - PostHashtag, Follow : 시퀄라이즈가 관계를 파악해 생성
- 자동으로 생성된 모델도 접근할 수 있음
    - `db.sequelize.models.PostHashtag`
    - `db.sequelize.models.Follow`
- 생성한 모델을 데이터베이스 및 서버와 연결(데이터베이스의 이름 : nodebird)
- 시퀄라이즈는 config.json을 읽어 데이터베이스를 생성해주는 기능이 존재

```json
{
	"development": {
		"username": "root",
		"password": "[root 비밀번호]",
		"database": "nodebird",
		"host": "127.0.0.1",
		"dialect": "mysql"
	},
]
```

```bash
$ npx sequelize db:create
```

- 모델을 서버와 연결

```jsx
...
dotenv.config();
const pageRouter = require('./routes/page');
const { sequelize } = require('./models');

const app = express();
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
...
```

- 서버를 실행
- 시퀄라이즈는 테이블 생성 쿼리문에 IF NOT EXISTS를 넣어주므로 테이블이 없을 때 테이블을 자동으로 생성

```bash
$ npm start
```

# 9.3 Passport 모듈로 로그인 구현하기

- Passport 모듈 : 서비스를 사용할 수 있게 해주는 여권 같은 역할
- Passport 관련 패키지들을 설치

```bash
$ npm i passport passport-local passport-kakao bcrypt
```

- Passport 모듈을 app.js와 미리 연결

```jsx
...
const dotenv = require('dotenv');
const passport = require('passport');

dotenv.config();
const pageRouter = require('./routes/page');
const { sequelize } = require('./models');
const passportConfig = require('./passport'); // require('./passport/index.js')와 같음

const app = express();
passportConfig(); // 패스포트 설정
app.set('port', process.env.PORT || 8001);
app.set('view engine', 'html');
...
app.use(session({
	resave: false,
	saveUnitialized: false,
	secret: process.env.COOKIE_SECRET,
	cookie: {
		httpOnly: true,
		secure: false,
	},
}));
app.use(passport.initialize());
// passport.initialize 미들웨어 : 요청(req 객체)에 passport 설정을 심음
app.use(passport.session());
// passport.session 미들웨어 : req.session 객체에 passport 정보 저장

app.use('/', pageRouter);
...
// req.session 객체는 express-session에서 생성하는 것이므로 passport 미들웨어는 express-session 미들웨어보다 뒤에 연결해야 함
```

- passport 폴더 내부에 index.js 파일을 만들고 Passport 관련 코드 작성

```jsx
const passport = require('passport');
const local = require('./localStrategy'); // 로컬 로그인 전략에 대한 파일
const kakao = require('./kakaoStrategy'); // 카카오 로그인 전략에 대한 파일
const User = require('../models/user');
// Passport는 로그인 시의 동작을 전략(strategy)이라고 표현

module.exports = () => {
	passport.serializeUser((user, done) => { // 로그인 시 실행됨
	// req.session(세션) 객체에 어떤 데이터를 저장할지 정하는 메서드
		done(null, user.id );
		// 매개변수로 user를 받고 나서 done 함수에 두 번째 인수로 user.id를 넘기고 있음
		// 첫 번째 인수 : 에러가 발생할 때 사용하는 것
		// 두 번째 인수 : 저장하고 싶은 데이터(세션에 사용자 정보를 모두 저장하면 세션의 용량이 커지고 데이터 일관성에 문제가 발생하므로 사용자의 아이디만 저장)
	})
	
	passport.deserializeUser((id, done) => { // 각 요청마다 실행됨
	// passport.session 미들웨어가 호출
	// serializeUser의 done의 두 번째로 인수로 넣었던 데이터가 매개변수가 됨(사용자의 아이디)
		User.fineOne({ where: { id } })
			.then(user => done(null, user)) // 조회한 정보를 req.user에 저장
			.catch(err => done(err));
		});
		
		local();
		kakado();
	};
```

- `serializeUser` : 사용자 정보 객체에서 아이디만 추려 세션에 저장하는 것
- `deserialize User`  : 세션에 저장한 아이디를 통해 사용자 정보 객체를 불러오는 것
- 전체 과정
    1. `/auth/login` 라우터를 통해 로그인 요청이 들어옴
    2. 라우터에서 `passport.authenticate` 메서드 호출
    3. 로그인 전략(LocalStrategy) 수행
    4. 로그인 성공 시 사용자 정보 객체와 함께 `req.login` 호출
    5. `req.login` 메서드가 `passport.serializeUser` 호출
    6. `req.session`에 사용자 아이디만 저장해서 세션 생성
    7. `express-session`에 설정한 대로 브라우저에 `connect.sid` 세션 쿠키 전송
    8. 로그인 완료
- 로그인 이후의 과정
    1. 요청이 들어옴(어떠한 요청이든 상관없음)
    2. 라우터에 요청이 도달하기 전에 `passport.session` 미들웨어가 `passport.deserializeUser` 메서드 호출
    3. `connect.sid` 세션 쿠키를 읽고 세션 객체를 찾아서 `req.session`으로 만듦
    4. `req.session`에 저장된 아이디로 데이터베이스에서 사용자 조회
    5. 조회된 사용자 정보를 `req.user`에 저장
    6. 라우터에서 `req.user` 객체 사용 가능

## 9.3.1 로컬 로그인 구현하기

- 로컬 로그인 : 다른 SNS 서비스를 통해 로그인하지 않고 자체적으로 회원 가입 후 로그인하는 것
- 아이디/비밀번호 또는 이메일/비밀번호를 통해 로그인하는 것

- 회원 가입, 로그인, 로그아웃 라우터가 필요
    - 로그인한 사용자는 회원 가입과 로그인 라우터에 접근하면 안 됨
    - 로그인하지 않은 사용자는 로그아웃 라우터에 접근하면 안 됨
    - 라우터에 접근 권한을 제어하는 미들웨어가 필요

- middlewares 폴더를 만들고 그 안에 index.js 작성

```jsx
exports.isLoggedIn = (req, res, next) => {
	if (req.isAuthenticated()) {
	// 로그인 중이면 isAuthenticated()가 true, 그렇지 않으면 flase
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
// 로그아웃 라우터나 이미지 업로드 라우터 등은 로그인한 사람만 접근할 수 있게 해야 함
// 회원 가입 라우터나 로그인 라우터는 로그인하지 않은 사람만 접근할 수 있게 해야 함
```

- `isLoggedIn`과 `isNotLoggedIn` 미들웨어 생성

```jsx
const express = require('express');
const { isLoggedIn, isNotLoggedIn } = require('../middlewares');
const { renderProfile, renderJoin, renderMain } = require('../controllers/page');
const router = express.Router();

router.use((req, res, next) => }
	res.locals.user = req.user; // 넌적스에서 user 객체를 통해 사용자 정보에 접근 가능
	res.locals.followerCount = 0;
	res.locals.followingCount = 0;
	res.locals.followingIdList = [];
	next();
});

router.get('/profile', isLoggedIn, renderProfile);
router.get('/join', isNotLoggedIn, renderJoin);
...
// 자신의 프로필은 로그인해야 볼 수 있으므로 isLoggedIn 미들웨어를 사용
// req.isAuthenticated()가 true여야 next가 호출되어 res.render가 있는 미들웨어로 넘어감
// false라면 로그인 창이 있는 메인 페이지로 리다이렉트됨

// 회원 가입 페이지는 로그인하지 않은 사람에게만 보여야 함
// isNotLoggedIn 미들웨어로 req.isAuthenticated()가 false일 때만 next를 호출하도록 함

// 로그인 여부로만 미들웨어를 만들 수 있는 것이 아니라 팔로잉 여부, 관리자 여부 등의 미들웨어를
// 만들 수 있으므로 다양하게 활용할 수 있음
```

- 회원 가입, 로그인, 로그아웃 라우터와 컨트롤러 작성

```jsx
const express = require('express');
const passport = require('passport');

const { isLoggedIn, isNotLoggedIn } = require('../middlewares');
const { join, login, logout } = require('../controllers/auth);

const router = express.Router();

// POST /auth/join
router.post('/join', isNotLoggedIn, join);

// POST /auth/login
router.post('/login', isNotLoggedIn, login);

// GET /auth/logout
router.get('/logout', isLoggedIn, logout);

module.exports = router;
```

```jsx
const bcrypt = require('bcrypt');
const passport = require('passport');
const User = require('../models/user');

// 1 : 회원 가입 컨트롤러
// 기존에 같은 이메일로 가입한 사용자가 있는지 조회
// 있다면 회원 가입 페이지로 되돌려보냄(주소 뒤에 에러를 쿼리스트링으로 표시)
// 없다면 비밀번호를 암호화하고 사용자 정보를 생성
exports.join = async(req, res, next) => }
	const { email, nick, password } = req.body;
	try {
		const exUser = await User.findOne({ where: { email } });
		if (exUser) {
			return res.redirect('/join?error=exist');
		}
		const bash = await bcrypt.hash(password, 12); // 회원 가입 시 비밀번호는 암호화해서 저장
		// bcrypt의 두 번째 인수 : pbkdf2의 반복 횟수와 비슷한 기능
		// 숫자가 커질수록 비밀번호를 알아내기 어려워지지만 암호화 시간도 오래 걸림
		// 12 이상을 추천하며, 31까지 사용 가능
		// 프로미스를 지원하는 함수이므로 await을 사용
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

// 2 : 로그인 컨트롤러
// 로그인 요청이 들어오면 passport.authenticate('local') 미들웨어가 로컬 로그인 전략 수행
exports.login = (req, res, next) => {
	passport.authenticate('local', (authError, user, info) => {
	// 전략이 성공하거나 실패하면 authenticate 메서드의 콜백 함수가 실행됨
	// 콜백 함수의 첫 번째 매개변수 값이 있다면 실패한 것
	// 두 번째 매개변수 자리는 사용자 정보(이 자리에 값이 있다면 성공한 것, 이 값으로 req.login 메서드 호출)
	// Passport는 req 객체에 login, logout 메서드를 추가
	// req.login은 passport.serializeUser를 호출하고, req.login에 제공하는 user 객체가 serializeUser로 넘어가게 됨
	// 이때, connect.sid 세션 쿠키가 브라우저에서 전송됨
		if (authError) {
			console.error(authError);
			return next(authError);
		}
		if (!user) {
			return res.redirect(`/?error=${info.message}`);
		}
		return req.login(user, (loginError) => {
			if (loginError) {
				console.error(loginError);
				return next(loginError);
			}
			return res.redirect('/');
		});
	})(req, res, next); // 미들웨어 내의 미들웨어에는 (req, res, next)를 붙임
};

// 3 : 로그아웃 컨트롤러
// req.logout 메서드는 req.user 객체와 req.session 객체를 제거
// req/logout 메서드는 콜백 함수를 인수로 받고, 세션 정보를 지운 후 콜백 함수가 실행됨
// 콜백 함수에서는 메인 페이지로 되돌아가면 됨
exports.logout = (req, res) => {
	req.logout(() => {
		res.redirect('/');
	});
};
```

- 나중에 app.js와 연결할 때 `/auth` 접두사를 붙일 것이므로 라우터의 주소는 각각 `/auth/join`, `/auth/login`, `/auth/logout`이 됨

- passport-local 모듈에서 Strategy 생성자를 불러와 그 안에 전략 구현

```jsx
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcrypt');

const User = require('../models/user');

module.exports = () => {
// 1
// LocalStrategy 생성자의 첫 번째 인수로 주어진 객체는 전략에 관한 설정을 하는 곳
	passport.use(new LocalStrategy({
		usernameField: 'email', // 일치하는 로그인 라우터의 req.body 속성명을 적음
		passwordField: 'password', // 일치하는 로그인 라우터의 req.body 속성명을 적음
		passReqToCallback" false,
		
// 2 : 실제 전략을 수행하는 async 함수(LocalStrategy 생성자의 두 번째 인수로 들어감)
// 첫 번째 인수에서 넣어준 email과 password는 각각 async 함수의 첫 번째와 두 번째 매개변수
// 세 번째 매개변수인 done 함수는 passport.authenticate의 콜백 함수
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
}
```

- 사용자 데이터베이스에서 일치하는 이메일이 있는지 찾은 후, 있다면 bcrypt의 compare 함수로 비밀번호를 비교
- 비밀번호까지 일치한다면 done 함수의 두 번째 인수로 사용자 정보를 넣어 보냄
    - 두 번째 인수를 사용하지 않는 경우 : 로그인에 실패했을 때 뿐
- done 함수의 첫 번째 인수를 사용하는 경우 : 서버 쪽에서 에러가 발생했을 때
- done 함수의 세 번째 인수를 사용하는 경우 : 로그인 처리 과정에서 비밀번호가 일치하지 않거나 존재하지 않는 회원인 경우(사용자 정의 에러가 발생했을 때)
- done이 호출된 후에는 다시 passport.authenticate의 콜백 함수에서 나머지 로직이 실행됨
- 로그인이 성공했다면 메인 페이지로 리다이렉트되면서 로그인 폼 대신 회원 정보가 뜸

## 9.3.2 카카오 로그인 구현하기

- 카카오 로그인 : 로그인 인증 과정을 카카오에 맡기는 것
- 사용자는 번거롭게 새로운 사이트에 회원 가입하지 않아도 돼서 좋고, 서비스 제공자는 로그인 과정을 안심하고 검증된 SNS에 맡길 수 있어 좋음
- SNS 로그인의 특징 : 회원 가입 절차가 따로 없음
    - 처음 로그인 할 때는 회원 가입 처리를 해야 하고, 두 번째 로그인부터는 로그인 처리를 해야 함
    - SNS 로그인 전략은 로컬 로그인 전략보다 다소 복잡

- passport-kakao 모듈로부터 Strategy 생성자를 불러와 전략을 구현

```jsx
const passport = require('passport');
const KakaoStrategy = require('passport-kakao').Strategy;

const User = require('../models/user');

// 1 : 카카오 로그인에 대한 설정을 함
module.exports = () => {
	passport.use(new KakaoStrategy({
		clientID: process.env.KAKAO_ID, // 카카오에서 발급해주는 아이디(노출되지 않아야 함)
		callbackURL: '/auth/kakao/callback', // 카카오로부터 인증 결과를 받을 라우터 주소

// 2 : 먼저 기존에 카카오를 통해 회원 가입한 사용자가 있는지 조회
	}, async (accessToken, refreshToken, profile, done) => {
		console.log('kakao profile', profile);
		try {
			const exUser = await User.findOne({
				where: { snsId: profile.id, provider: 'kakao' },
			});
			if (exUser) {
// 있는 경우(이미 회원 가입되어 있는 경우) 사용자 정보와 함께 done 함수를 호출하고 전략을 종료
				done(null, exUser);
				
// 3 : 없다면 회원 가입 진행
			} else {
// 카카오에서는 인증 후 callbackURL에 적힌 주소로 accessToken, refreshToken과 profile을 보냄
// profile 객체에서 원하는 정보를 꺼내 와 회원 가입을 진행
				const newUser = await User.create({
					email: profile._json?.kakao_account?.email,
					// profile의 속성이 undefined일 수도 있어 옵셔널 체이닝 문법 사용
					nick: profile.displayName,
					snsId: profile.id,
					provider: 'kakao',
				});
				done(null, newUser); // 사용자를 생성한 뒤 done 함수를 호출
			}
		} catch (error) {
			console.error(error);
			done(error);
		}
	}));
};
```

- 카카오 로그인 라우터 생성(로그아웃 라우터 아래에 추가)

```jsx
...
router.get('/logout', isLoggedIn, logout);

// GET /auth/kakao
router.get('/kakao', passport.authenticate('kakao'));

// GET /auth/kakao/callback
router.get('/kakao/callback', passport.authenticate('kakao', {
	failureRedirect: '/?error=카카오로그인 실패'.
}), (req, res) => {
	res.redirect('/'); // 성공 시에는 /로 이동
});

module.exports = router;
```

- GET /auth/kakao로 접근하면 카카오 로그인 과정이 시작됨
- GET /auth/kakao에서 로그인 전략(KakaoStrategy)을 수행
    - 처음에는 카카오 로그인 창으로 리다이렉트하여 로그인 후 성공 여부를 GET /auth/kakao/callback으로 받음
- 로컬 로그인과 다른 점 : passport.authenticate 메서드에 콜백 함수를 제공하지 않음
    - 카카오 로그인은 로그인 성공 시 내부적으로 req.login을 호출하므로 우리가 직접 호출할 필요 없음
    - 콜백 함수 대신 로그인에 실패했을 때 어디로 이동할지를 failureRedirect 속성에 적음
    - 성공 시에도 어디로 이동할지를 다음 미들웨어에 적음

- 추가한 auth 라우터를 app.js에 연결

```jsx
...
const pageRouter = require('./routes/page');
const authRouter = require('./routes/auth');
const { sequelize } = require('./models');
...
app.use('/', pageRouter);
app.use('/auth', authRouter);
...
```

- kakaoStrategy.js에서 사용하는 `clientID`를 [https://developers.kakao.com](https://developers.kakao.com에) 에 접속해 발급받아야 함

# 9.4 multer 패키지로 이미지 업로드 구현하기

- 패키지 설치 : `npm i multer`
- 이미지를 어떻게 저장할 것인지는 서비스의 특성에 따라 달라짐
    - NodeBird 서비스는 input 태그를 통해 이미지를 선택할 때 바로 업로드를 진행
    - 업로드된 사진 주소를 다시 클라이언트에 알림
    - 게시글을 저장할 때는 데이터베이스에 직접 이미지 데이터를 넣는 대신 이미지 경로만 저장
    - 이미지는 서버 디스크(uploads 폴더)에 저장

```jsx
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');

const { afterUploadImage, uploadPost } = require('../controllers/post');
const { isLoggedIn } = require('../middlewares');

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

// POST /post/img : 이미지 하나를 업로드받은 뒤 이미지의 저장 경로를 클라이언트로 응답
// static 미들웨어가 /img 경로의 정적 파일을 제공하므로 클라이언트에서 업로드한 이미지에 접근 가능
router.post('/img', isLoggedIn, upload.single('img'), afterUploadImage);

// POST /post : 게시글 업로드를 처리하는 라우터
// 이전 라우터에서 이미지를 업로드했다면 이미지 주소도 req.body.url로 전송됨
// 이미지 주소가 온 것이지, 이미지 데이터 자체가 온 것은 아님
// 이미지는 이미 POST /post/img 라우터에서 저장됨
const upload2 = multer();
router.post('/', isLoggedIn, upload2.none(), uploadPost);

module.exports = router;
```

```jsx
const { Post, Hashtag } = require('../models');

exports.afterUploadImage = (req, res) => {
	console.log(req.file);
	res.json({ url: `/img/${req.file.filename}` });
};

exports.uploadPost = async (req, res, next) => {
	try {
		const post = await Post.create({
			content: req.body.content,
			img: req.body.url,
			UserId: req.user.id,
		});
		const hashtags = req.body.content.match(/#[^\s#]*/g); // 해시태그를 정규표현식으로 추출
		if (hashtages) {
			const result = await Promise.all(
				hashtags.map(tag => {
					return Hashtag.findOrCreate({ // 저장할 때 사용하는 메서드
					// 데이터베이스에 해시태그가 존재하면 가져오고, 존재하지 않으면 생성한 후 가져옴
						where: {title: tag.slice(1).toLowerCase() }, // 해시태그에서 #을 떼고 소문자로 바꿈
					})
				}),
			);
			await post.addHashtags(result.map(r => r[0])); // 해시태그 모델들을 게시글과 연결
			// 결괏값인 [모델, 생성 여부] 중 모델만 추출해냄
		}
		res.redirect('/');
	} catch (error) {
		console.error(error);
		next(error);
	}
};
```

- 메인 페이지 로딩 시 메인 페이지와 게시글을 함께 로딩

```jsx
const { User, Post } = require('../models');

exports.renderProfile = (req, res) => {
	res.render('profile', { title: '내 정보 - NodeBird });
};

exports.renderJoin = (req, res) => {
	res.render('join', { title: '회원 가입 - NodeBird });
};

exports.renderMain = async (req, res, next) => {
	try {
		const posts = await Post.findAll({
			include: {
				model: User,
				attributes: ['id', 'nick'], // 조회할 때 게시글 작성자의 아이디와 닉네임을 JOIN해서 제공
			},
			order: [['createdAt', 'DESC']], // 게시글의 순서는 최신순으로 정렬
		});
		res.render('main', {
			title: 'NodeBird',
			twits: posts, // 데이터베이스에서 게시글을 조회한 뒤 결과를 twits에 넣어 렌더링
		});
	} catch (err) {
		console.error(err);
		next(err);
	}
}
```

# 9.5 프로젝트 마무리하기

- 다른 사용자를 팔로우하는 기능을 만들기 위해 routes/user.js와 controllers/user.js를 작성

```jsx
const express = require('express');

const { isLoggedIn } = require('../middlewares');
const { follow } = require('../controllers/user');

const router = express.Router();

// POST /user/:id/follow에서 :id 부분이 req.params.id가 됨
router.post('/:id/follow', isLoggedIn, follow);

module.exports = router;
```

```jsx
const User = require('../models/user');

exports.follow = async (req, res, next) => {
	try {
		const user = await User.findOne({ where: { id: req.user.id } });
		// 팔로우할 사용자를 데이터베이스에서 조회
		if (user) { // req.user.id가 follwerId, req.params.id가 followingId
			await user.addFollowing(parseInt(req.params.id, 10)); // 현재 로그인한 사용자와의 관계 지정
			res.send('success');
		} else {
			res.status(404).send('no user');
		}
	} catch (error) {
		console.error(error);
		next(error);
	}
};
```

```jsx
...
	passport.deserializeUser((id, done) => {
		User.findOne({ // 세션에 저장된 아이디로 사용자 정보를 조회할 때 팔로잉, 팔로워 목록도 같이 조회
			where: { id },
			include: [{
				model: User,
				attributes: ['id', 'nick'], 
				// 계속 attributes를 지정하는 이유 : 실수로 비밀번호를 조회하는 것을 방지
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
...
```

```jsx
// 팔로잉/팔로워 숫자와 팔로우 버튼 표시
...
router.use((req, res, next) => {
// 로그인한 경우에는 req.user가 존재하므로 팔로잉/팔로워 수와 팔로잉 아이디 리스트를 넣음
	res.locals.user = req.user;
	res.locals.followerCount = req.user?.Followers?.length || 0;
	res.locals.followerCount = req.user?.Followings?.length || 0;
	res.locals.followingIdList = req.user?.Followings?.map(f => f.id) || [];
	// 팔로잉 아이디 리스트를 넣는 이유 : 팔로잉 아이디 리스트에 게시글 작성자의 아이디가 존재하지 않으면 팔로우 버튼을 보여주기 위해
	next();
});
...
```

```jsx
const express = require('express');
const { isLoggedin, isNotLoggedIn } = require('../middlewares');
const {
	renderProfile, renderJoin, renderMain, renderHashtag,
} = require('../controllers/page');

const router = express.Router();
...
router.get('hashtag', renderHashtag);

module.exports = router;
```

```jsx
// 해시태그로 조회하는 GET /hashtag 라우터
// 쿼리스트링으로 해시태그 이름을 받고 해시태그 값이 없는 경우 메인 페이지로 돌려보냄
const { User, Post, Hashtag } = require('../models');
...
exports.renderHashtag = async (req, res, next) => {
	const query = req.query.hashtag;
	if (!query) {
		return res.redirect('/');
	}
	try {
		const hashtag = await Hashtag.findOne({ where: { title: query } });
		// 데이터베이스에서 해당 해시태그가 존재하는지 검색
		let posts = [];
		if (hashtag) { // 해시태그가 있다면 모든 게시글을 가져옴
			posts = await hashtag.getPosts({ include: [{ model: User }] }); 
			// include를 이용해 작성자 정보를 합침
		}
		
		return res.render('main', {
			title: `${query} | NodeBird`,
			twits: posts, // 메인 페이지를 렌더링하면서 전체 게시글 대신 조회된 게시글만 랜더링
		});
	} catch (error) {
		console.error(error);
		return next(error);
	}
};
```

```jsx
// routes/post.js와 routes/user.js를 app.js에 연결
...
const pageRouter = require('./routes/page');
const authRouter = require('./routes/auth');
const postRouter = require('./routes/post');
const userRouter = require('./routes/user');
const { sequelize } = require('./models');
const passportConfig = require('./passport');
...
app.use(morgan('dev'));
app.use(express.static(path.join(__dirname, 'public')));
app.use('/img', express.static(path.join(__dirname, 'uploads')));
// 업로드한 이미지를 제공할 라우터(/img)도 express.static 미들웨어로 uploads 폴더와 연결
// express.static은 여러 번 사용 가능
app.use(express.json());
...
app.use('/', pageRouter);
app.use('/auth', authRouter);
app.use('/post', postRouter);
app.use('/user', userRouter);
...
```

## 9.5.1 스스로 해보기

- 팔로잉 끊기(시퀄라이즈의 destroy 메서드와 라우터 활용)
- 프로필 정보 변경하기(시퀄라이즈의 update 메서드와 라우터 활용)
- 게시글 좋아요 누르기 및 좋아요 취소하기(사용자-게시글 모델 간 N : M 관계 정립 후 라우터 활용)
- 게시글 삭제하기(등록자와 현재 로그인한 사용자가 같을 때, 시퀄라이즈의 destroy 메서드와 라우터 활용)
- 사용자 이름을 누르면 그 사용자의 게시글만 보여주기
- 매번 데이터베이스를 조회하지 않도록 deserializeUser 캐싱하기(객체 선언 후 객체에 사용자 정보 저장, 객체 안에 캐싱된 값이 있으면 조회)

## 9.5.2 핵심 정리

- 서버는 요청에 응답하는 것이 핵심이므로 요청을 수락하든 거절하든 상관 없이 반드시 응답해야 함
    - 한 번만 응답해야 에러가 발생하지 않음
- 개발 시 서버를 매번 수동으로 재시작하지 않으려면 nodemon을 사용하는 것이 좋음
- dotenv 패키지와 .env 파일로 유출되면 안 되는 비밀 키를 관리
- 라우터는 routes 폴더에, 데이터베이스는 models 폴더에, html 파일은 views 폴더에 각각 구분해서 저장하면 프로젝트 규모가 커져도 관리하기 쉬움
- 라우터에서 응답을 보내는 미들웨어 : 컨트롤러
    - 컨트롤러도 따로 분리하면 코드를 관리할 때 편함
- 데이터를 구성하기 전에 데이터 간 관계 파악 필요
- middlewares/index.js처럼 라우터 내에 미들웨어를 사용할 수 있음
- Passport의 인증 과정 기억하기
    - serializeUser와 deserializeUser가 언제 호출되는지 파악하고 있어야 함
- 프론트엔드 form 태그의 인코딩 방식이 multipart일 때는 multer와 같은 multipart 처리용 패키지를 사용하는 것이 좋음