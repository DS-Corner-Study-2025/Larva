# 12장 : 웹 소켓으로 실시간 데이터 전송하기

# 12.1 웹 소켓 이해하기

- **웹 소켓** : HTML5에 새로 추가된 스펙으로 실시간 양방향 데이터 전송을 위한 기술
    - HTTP와 다르게 WS라는 프로토콜을 사용
    - 브라우저와 서버가 WS 프로토콜을 지원하면 사용 가능
    - 최신 브라우저는 대부분 웹 소켓을 지원
    - 노드에서는 ws나 [Socket.IO](http://Socket.IO) 같은 패키지를 통해 웹 소켓 사용 가능
- **폴링(polling)** : 웹 소켓이 나오기 이전 HTTP 기술을 사용해 실시간 데이터 전송을 구현한 방법
    - 주기적으로 서버에 새로운 업데이트가 있는지 확인하는 요청(단방향 통신)
    - 업데이트가 있다면 새로운 내용을 가져오는 단순한 방법

- HTML5가 나오면서 웹 브라우저와 웹 서버가 지속적으로 연결된 라인을 통해 실시간으로 데이터를 주고받을 수 있는 웹 소켓이 등장
    - 처음에 웹 소켓 연결이 이뤄지고 나면 그다음부터는 계속 연결된 상태로 있어 따로 업데이트가 있는지 요청을 보낼 필요가 없음
    - 업데이트할 내용이 생겼다면 서버에서 바로 클라이언트에 알림
    - HTTP 프로토콜과 포트를 공유할 수 있으므로 다른 포트에 연결할 필요도 없음
    - 폴링 방식에 비해 성능도 매우 개선됨
- **서버센트 이벤트(Server Sent Events, SSE)**
    - Event Source라는 객체 사용
    - 처음에 한 번만 연결하면 서버가 클라이언트에 지속적으로 데이터 전송
    - 웹 소켓과 비교했을 때 클라이언트에서 서버로는 데이터를 보낼 수 없다는 차이 존재
    - 서버에서 클라이언트로 데이터를 보내는 단방향 통신
    - 주식 차트 업데이트, SNS에서 새로운 게시물 가져오기 등 굳이 양방향 통신을 할 필요가 없는 경우 사용
- **Socket.IO** : 웹 소켓을 편리하게 사용할 수 있도록 도와주는 라이브러리

- 웹 소켓을 지원하지 않는 IE9와 같은 브라우저에서는 알아서 웹 소켓 대신 폴링 방식을 사용해 실시간 데이터 전송을 가능하게 함

# 12.2 ws 모듈로 웹 소켓 사용하기

- ws 모듈로 웹 소켓이 무엇인지 직접 체험
- gif-chat이라는 새로운 프로젝트 생성

```json
{
  "name": "gif-chat",
  "version": "0.0.1",
  "description": "GIF 웹 소켓 채팅방",
  "main": "app.js",
  "scripts": {
    "start": "nodemon app"
  },
  "author": "Zero Cho",
  "license": "ISC",
  "dependencies": {
    "cookie-parser": "^1.4.6",
    "dotenv": "^16.0.1",
    "express": "^4.18.1",
    "express-session": "^1.17.0",
    "morgan": "^1.9.1",
    "nunjucks": "^3.2.1",
    "ws": "^8.2.3"
  },
  "devDependencies": {
    "nodemon": "^2.0.16"
  }
}
```

- 패키지 설치 : `$ npm i`
- .env와 app.js, routes/index.js 파일을 작성

```tsx
COOKIE_SECRET=gifchat
```

```jsx
const express = require('express');
const path = require('path');
const morgan = require('morgan');
const cookieParser = require('cookie-parser');
const session = require('express-session');
const nunjucks = require('nunjucks');
const dotenv = require('dotenv');

dotenv.config();
const indexRouter = require('./routes');

const app = express();
app.set('port', process.env.PORT || 8005);
app.set('view engine', 'html');
nunjucks.configure('views', {
  express: app,
  watch: true,
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

app.use('/', indexRouter);

app.use((req, res, next) => {
  const error =  new Error(`${req.method} ${req.url} 라우터가 없습니다.`);
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

```jsx
const express = require('express');

const router = express.Router();

router.get('/', (req, res) => {
  res.render('index');
});

module.exports = router;
```

- ws 모듈 설치 : `$ npm i ws@8`
- 웹 소켓을 익스프레스 서버에 연결

```jsx
...
dotenv.config();
const webSocket = require('./socket');
const indexRouter = require('./routes');

const app = express();
...
app.use((err, req, res, next) => {
  res.locals.message = err.message;
  res.locals.error = process.env.NODE_ENV !== 'production' ? err : {};
  res.status(err.status || 500);
  res.render('error');
});

const server = app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기중');
});

webSocket(server);
```

- 웹 소켓 로직이 들어 있는 socket.js 파일 작성

```jsx
const WebSocket = require('ws');

module.exports = (server) => {
  const wss = new WebSocket.Server({ server });

  wss.on('connection', (ws, req) => { // 클라이언트가 서버와 웹소켓 연결 시 발생
    const ip = req.headers['x-forwarded-for'] || req.socket.remoteAddress;
              // 클라이언트의 IP를 알아내는 유명한 방법 중 하나
              // 로컬 호스트로 접속한 경우, 크롬에선느 IP가 ::1로 뜸
              // 익스프레스에서는 IP를 확인할 때, proxy-addr 패키지를 사용
    console.log('새로운 클라이언트 접속', ip); // 이벤트 리스너 세 개 연결
    ws.on('message', (message) => { // 클라이언트로부터 메시지가 왔을 때 발생
      console.log(message.toString());
    });
    ws.on('error', (error) => { // 웹 소켓 연결 중 문제가 생겼을 경우 발생
      console.error(error);
    });
    ws.on('close', () => { // 클라이언트와 연결이 끊겼을 때 발생
      console.log('클라이언트 접속 해제', ip);
      clearInterval(ws.interval); // setInterval을 clearInterval로 정리(없다면 메모리 누수 발생)
    });

    ws.interval = setInterval(() => { // 3초마다 모든 클라이언트에게 메시지 전송
      if (ws.readyState === ws.OPEN) { // readyState가 OPEN 상태인지 확인
        ws.send('서버에서 클라이언트로 메시지를 보냅니다.'); // OPEN일 때만 메시지 가능
      }
    }, 3000);
  });
};
```

- ws 모듈을 불러온 후 익스프레스 서버를 웹 소켓 서버와 연결
- 연결 후에는 웹 소켓 서버(wss)에 이벤트 리스너를 붙임(웹 소켓은 이벤트 기반으로 작동한다고 생각)

- 웹 소켓은 단순히 서버에서 설정한다고 해서 작동하는 것이 아니라 클라이언트에서도 웹 소켓을 사용해야 함(양방향 통신이기 때문)
    - views 폴더를 만들고 index.html 파일을 생성해서 script 태그에 웹 소켓 코드 삽입
    - error.html도 함께 작성

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>GIF 채팅방</title>
</head>
<body>
<div>F12를 눌러 console 탭과 network 탭을 확인하세요.</div>
<script>
  const webSocket = new WebSocket("ws://localhost:8005");
  // WebSocket 생성자에 연결할 서버 주소를 넣고 webSocket 객체 생성
  // 서버 주소의 프로토콜이 ws인 것에 주의
  webSocket.onopen = function () { // 서버와 연결이 맺어지는 경우 호출
    console.log('서버와 웹소켓 연결 성공!');
  };
  webSocket.onmessage = function (event) { // 서버로부터 메시지가 오는 경우 호출
    console.log(event.data);
    webSocket.send('클라이언트에서 서버로 답장을 보냅니다');
  };
</script>
</body>
</html>
```

```html
<h1>{{message}}</h1>
<h2>{{error.status}}</h2>
<pre>{{error.stack}}</pre>
```

- 서버를 실행하는 순간 서버는 클라이언트에 3초마다 메시지를 보내고, 클라이언트도 서버로부터 메시지가 오는 순간 바로 답장
- npm start를 통해 서버를 실행한 뒤 http://localhost:8005에 접속해 개발자 도구의 Console 탭 실행
    - 접속하는 순간부터 노드의 콘솔과 브라우저의 콘솔에 3초마다 메시지가 찍힘

# 12.3 [Socket.IO](http://Socket.IO) 사용하기

- ws 패키지는 간단하게 웹 소켓을 사용하고자 할 때 좋음
- 구현하려는 서비스가 좀 더 복잡해진다면 Socket.IO를 사용하는 것이 편함
    - Socket.IO에 편의 기능이 많이 추가되어 있음

- [Socket.IO](http://Socket.IO) 설치 : `$ npm i [socket.io](http://socket.io) @4`
- ws 패키지 대신 [Socket.IO](http://Socket.IO) 연결

```jsx
const SocketIO = require('socket.io');

module.exports = (server) => {
  const io = SocketIO(server, { path: '/socket.io' });
  // 두 번째 인수로 옵션 객체를 넣어 서버에 관한 여러 가지 설정 가능

  io.on('connection', (socket) => { // 클라이언트가 접속했을 때 발생
    const req = socket.request;
    const ip = req.headers['x-forwarded-for'] || req.socket.remoteAddress;
    console.log('새로운 클라이언트 접속!', ip, socket.id, req.ip);
    // socket.id로 소켓 고유의 아이디를 가져올 수 음
    socket.on('disconnect', () => { // 클라이언트가 연결을 끊었을 때 발생
      console.log('클라이언트 접속 해제', ip, socket.id);
      clearInterval(socket.interval);
    });
    socket.on('error', (error) => { // 통신 과정 중에 에러가 나왔을 때 발생
      console.error(error);
    });
    socket.on('reply', (data) => { // 클라이언트로부터 메시지(사용자가 직접 만든 이벤트)
      console.log(data);
    });
    socket.interval = setInterval(() => { // 3초마다 클라이언트로 메시지 전송
      socket.emit('news', 'Hello Socket.IO'); // 인수 : (이벤트 이름, 데이터)
    }, 3000);
  });
};
```

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>GIF 채팅방</title>
</head>
<body>
<div>F12를 눌러 console 탭과 network 탭을 확인하세요.</div>
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io.connect('http://localhost:8005', {
    path: '/socket.io',
    transports: ['websocket'],
  });
  socket.on('news', function (data) {
    console.log(data);
    socket.emit('reply', 'Hello Node.JS');
  });
</script>
</body>
</html>
```

# 12.4 실시간 GIF 채팅방 만들기

- 사람들이 익명으로 생성하고 자유롭게 참여하면서 GIF 파일을 올릴 수 있는 채팅방 만들기
- 몽고디비와 몽고디비 ODM인 몽구스 사용
- 랜덤 색상을 구현하는 모듈인 color-hash 모듈 설치
- 이미지를 업로드하기 위해 multer 모듈도 설치 : `npm i mongoose multer color-hash@2`
- 채팅방 스키마 생성

```jsx
const mongoose = require('mongoose');

const { Schema } = mongoose;
const roomSchema = new Schema({
  title: {
    type: String,
    required: true,
  },
  max: {
    type: Number, // 수용 인원
    required: true, 
    default: 10, // 기본적으로 10명
    min: 2, // 최소 2명
  },
  owner: {
    type: String,
    required: true,
  },
  password: String, // 비밀번호는 required 속성이 없으므로 꼭 넣지 않아도 됨(공개방)
  createdAt: {
    type: Date,
    default: Date.now,
  },
});

module.exports = mongoose.model('Room', roomSchema);
```

- 채팅 스키마 생성

```jsx
const mongoose = require('mongoose');

const { Schema } = mongoose;
const { Types: { ObjectId } } = Schema;
const chatSchema = new Schema({
  room: {
    type: ObjectId, // Room 스키마와 연결해 Room 컬렉션의 ObjectId가 들어가게 됨
    required: true,
    ref: 'Room',
  },
  user: {
    type: String,
    required: true,
  },
  chat: String, // required 속성이 없음
  gif: String, // required 속성이 없음
  createdAt: {
    type: Date,
    default: Date.now,
  },
});

module.exports = mongoose.model('Chat', chatSchema);
```

- 몽고디비와 연결

```jsx
const mongoose = require('mongoose');

const { MONGO_ID, MONGO_PASSWORD, NODE_ENV } = process.env;
const MONGO_URL = `mongodb://${MONGO_ID}:${MONGO_PASSWORD}@localhost:27017/admin`;

const connect = () => {
  if (NODE_ENV !== 'production') {
    mongoose.set('debug', true);
  }
  mongoose.connect(MONGO_URL, {
    dbName: 'gifchat',
    useNewUrlParser: true,
  }).then(() => {
    console.log("몽고디비 연결 성공");
  }).catch((err) => {
    console.error("몽고디비 연결 에러", err);
  });
};

mongoose.connection.on('error', (error) => {
  console.error('몽고디비 연결 에러', error);
});
mongoose.connection.on('disconnected', () => {
  console.error('몽고디비 연결이 끊겼습니다. 연결을 재시도합니다.');
  connect();
});

module.exports = connect;
```

```tsx
COOKIE_SECRET=gifchat
MONGO_ID=root
MONGO_PASSWORD=nodejsbook
```

- 서버를 실행할 때 몽고디비에 바로 접속할 수 있도록 서버와 몽구스를 연결

```tsx
...
dotenv.config();
const webSocket = require('./socket');
const indexRouter = require('./routes');
const connect = require('./schemas');

const app = express();
app.set('port', process.env.PORT || 8005);
app.set('view engine', 'html');
nunjucks.configure('views', {
  express: app,
  watch: true,
});
connect();
...
```

- html, css 파일 생성

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>{{title}}</title>
  <link rel="stylesheet" href="/main.css">
</head>
<body>
{% block content %}
{% endblock %}
{% block script %}
{% endblock %}
</body>
</html>
```

```html
{% extends 'layout.html' %}

{% block content %}
  <h1>{{message}}</h1>
  <h2>{{error.status}}</h2>
  <pre>{{error.stack}}</pre>
{% endblock %}
```

```css
* { box-sizing: border-box; }
.mine { text-align: right; }
.system { text-align: center; }
.mine img, .other img {
  max-width: 300px;
  display: inline-block;
  border: 1px solid silver;
  border-radius: 5px;
  padding: 2px 5px;
}
.mine div:first-child, .other div:first-child { font-size: 12px; }
.mine div:last-child, .other div:last-child {
  display: inline-block;
  border: 1px solid silver;
  border-radius: 5px;
  padding: 2px 5px;
  max-width: 300px;
}
#exit-btn { position: absolute; top: 20px; right: 20px; }
#chat-list { height: 500px; overflow: auto; padding: 5px; }
#chat-form { text-align: right; }
label[for='gif'], #chat, #chat-form [type='submit'] {
  display: inline-block;
  height: 30px;
  vertical-align: top;
}
label[for='gif'] { cursor: pointer; padding: 5px; }
#gif { display: none; }
table, table th, table td {
  text-align: center;
  border: 1px solid silver;
  border-collapse: collapse;
}
```

```html
{% extends 'layout.html' %}

{% block content %}
<h1>GIF 채팅방</h1>
<fieldset>
  <legend>채팅방 목록</legend>
  <table>
    <thead>
    <tr>
      <th>방 제목</th>
      <th>종류</th>
      <th>허용 인원</th>
      <th>방장</th>
    </tr>
    </thead>
    <tbody>
    {% for room in rooms %}
      <tr data-id="{{room._id}}">
        <td>{{room.title}}</td>
        <td>{{'비밀방' if room.password else '공개방'}}</td>
        <td>{{room.max}}</td>
        <td style="color: {{room.owner}}">{{room.owner}}</td>
        <td>
          <button
            data-password="{{'true' if room.password else 'false'}}"
            data-id="{{room._id}}"
            class="join-btn"
          >입장
          </button>
        </td>
      </tr>
    {% endfor %}
    </tbody>
  </table>
  <div class="error-message">{{error}}</div>
  <a href="/room">채팅방 생성</a>
</fieldset>
<script src="/socket.io/socket.io.js"></script>
<script>
  const socket = io.connect('http://localhost:8005/room', { // 네임스페이스
    path: '/socket.io',
  });

  socket.on('newRoom', function (data) { // 새 방 이벤트 시 새 방 생성
    const tr = document.createElement('tr');
    let td = document.createElement('td');
    td.textContent = data.title;
    tr.appendChild(td);
    td = document.createElement('td');
    td.textContent = data.password ? '비밀방' : '공개방';
    tr.appendChild(td);
    td = document.createElement('td');
    td.textContent = data.max;
    tr.appendChild(td);
    td = document.createElement('td');
    td.style.color = data.owner;
    td.textContent = data.owner;
    tr.appendChild(td);
    td = document.createElement('td');
    const button = document.createElement('button');
    button.textContent = '입장';
    button.dataset.password = data.password ? 'true' : 'false';
    button.dataset.id = data._id; // 버튼에 방 아이디 저장
    button.addEventListener('click', addBtnEvent);
    td.appendChild(button);
    tr.appendChild(td);
    tr.dataset.id = data._id; // tr에 방 아이디 저장
    document.querySelector('table tbody').appendChild(tr); // 화면에 추가
  });

  socket.on('removeRoom', function (data) { // 방 제거 이벤트 시 id가 일치하는 방 제거
    document.querySelectorAll('tbody tr').forEach(function (tr) {
      if (tr.dataset.id === data) {
        tr.parentNode.removeChild(tr);
      }
    });
  });

  function addBtnEvent(e) { // 방 입장 클릭 시
    if (e.target.dataset.password === 'true') { // 비밀방이면
      const password = prompt('비밀번호를 입력하세요');
      location.href = '/room/' + e.target.dataset.id + '?password=' + password;
    } else {
      location.href = '/room/' + e.target.dataset.id;
    }
  }

  document.querySelectorAll('.join-btn').forEach(function (btn) {
    btn.addEventListener('click', addBtnEvent);
  });
</script>
{% endblock %}

{% block script %}
<script>
  window.onload = () => {
    if (new URL(location.href).searchParams.get('error')) {
      alert(new URL(location.href).searchParams.get('error'));
    }
  };
</script>
{% endblock %}
<!--
- io.connect 메서드의 주소가 변경됨
- 주소 뒤에 /room이 붙음 : 네임스페이스(서버에서 /room 네임스페이스를 통해 보낸 데이터만 받을 수 있음)
- 네임스페이스를 여러 개 구분해 주고받을 데이터를 분류할 수 있음
- socket에는 미리 newRoom과 romoveRoom 이벤트를 달아둠(서버에서 웹 소켓으로 해당 이벤트를 발생시키면 이벤트 리스너의 콜백 함수가 실행됨)
-->
```

```html
{% extends 'layout.html' %}

{% block content %}
  <fieldset>
    <legend>채팅방 생성</legend>
    <form action="/room" method="post">
      <div>
        <input type="text" name="title" placeholder="방 제목">
      </div>
      <div>
        <input type="number" name="max" placeholder="수용 인원(최소 2명)" min="2" value="10">
      </div>
      <div>
        <input type="password" name="password" placeholder="비밀번호(없으면 공개방)">
      </div>
      <div>
        <button type="submit">생성</button>
      </div>
    </form>
  </fieldset>
{% endblock %}
```

```html
{% extends 'layout.html' %}

{% block content %}
  <h1>{{title}}</h1>
  <a href="/" id="exit-btn">방 나가기</a>
  <fieldset>
    <legend>채팅 내용</legend>
    <div id="chat-list">
      {% for chat in chats %}
        {% if chat.user === user %}
          <div class="mine" style="color: {{chat.user}}">
            <div>{{chat.user}}</div>
            {% if chat.gif %}}
              <img src="/gif/{{chat.gif}}">
            {% else %}
              <div>{{chat.chat}}</div>
            {% endif %}
          </div>
        {% elif chat.user === 'system' %}
          <div class="system">
            <div>{{chat.chat}}</div>
          </div>
        {% else %}
          <div class="other" style="color: {{chat.user}}">
            <div>{{chat.user}}</div>
            {% if chat.gif %}
              <img src="/gif/{{chat.gif}}">
            {% else %}
              <div>{{chat.chat}}</div>
            {% endif %}
          </div>
        {% endif %}
      {% endfor %}
    </div>
  </fieldset>
  <form action="/chat" id="chat-form" method="post" enctype="multipart/form-data">
    <label for="gif">GIF 올리기</label>
    <input type="file" id="gif" name="gif" accept="image/gif">
    <input type="text" id="chat" name="chat">
    <button type="submit">전송</button>
  </form>
  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io.connect('http://localhost:8005/chat', {
    // 네임스페이스가 /chat(/chat 네임스페이스로 보낸 데이터만 받을 수 있음)
      path: '/socket.io',
    });
    socket.emit('join', new URL(location).pathname.split('/').at(-1));
    socket.on('join', function (data) { // 사용자의 입장에 관한 데이터가 웹 소켓으로 전송될 때 호출
      const div = document.createElement('div');
      div.classList.add('system');
      const chat = document.createElement('div');
      chat.textContent = data.chat;
      div.appendChild(chat);
      document.querySelector('#chat-list').appendChild(div);
    });
    socket.on('exit', function (data) { // 사용자의 퇴장에 관한 데이터가 웹 소켓으로 전송될 때 호출
      const div = document.createElement('div');
      div.classList.add('system');
      const chat = document.createElement('div');
      chat.textContent = data.chat;
      div.appendChild(chat);
      document.querySelector('#chat-list').appendChild(div);
    });
  </script>
{% endblock %}
```

- app.js에 color-hash 패키지를 적용하여 접속한 사용자마다 고유 색상을 부여
    - color-hash 패키지 : 세션 아이디를 HEX 형식의 색상 문자열(#12C6B8 같은)로 바꿔주는 패키지
    - 해시이므로 같은 세션 아이디는 항상 같은 색상 문자열로 바뀜
    - 사용자가 많아질 경우에는 색상이 중복되는 문제가 있을 수 있음

```jsx
...
const dotenv = require('dotenv');
const ColorHash = require('color-hash').default;

dotenv.config();
...
app.use((req, res, next) => {
  if (!req.session.color) {
    const colorHash = new ColorHash();
    req.session.color = colorHash.hex(req.sessionID);
    // 세션에 color 속서이 없을 때는 사용자의 세션 아이디인 req.sessionID를 바탕으로 color 속성 생성
    console.log(req.session.color, req.sessionID);
  }
  next();
});

app.use('/', indexRouter);
...
webSocket(server, app);
```

- 서버의 socket.js에 웹 소켓 이벤트를 연결

```jsx
const SocketIO = require('socket.io');

module.exports = (server, app) => {
  const io = SocketIO(server, { path: '/socket.io' });
  app.set('io', io); // app.set('io', io)로 라우터에서 io 객체를 쓸 수 있게 저장
  const room = io.of('/room'); // of : Socket.IO에 네임스페이스를 부여하는 메서드
  const chat = io.of('/chat');

  room.on('connection', (socket) => { // /room 네임스페이스에 이벤트 리스너를 붙여줌
    console.log('room 네임스페이스에 접속');
    socket.on('disconnect', () => {
      console.log('room 네임스페이스 접속 해제');
    });
  });

  chat.on('connection', (socket) => { // /chat 네임스페이스에 붙인 이벤트 리스너
    console.log('chat 네임스페이스에 접속');

    socket.on('join', (data) => { // data는 브라우저에서 보낸 방 아이디
      socket.join(data); // 네임스페이스 아래 존재하는 방에 접속(방의 아이디를 인수로 받음)
    });

    socket.on('disconnect', () => {
      console.log('chat 네임스페이스 접속 해제');
    });
  });
};
```

- 라우터와 컨트롤러 생성

```jsx
const express = require('express');
const { renderMain, renderRoom, createRoom, enterRoom, removeRoom } = require('../controllers');

const router = express.Router();

router.get('/', renderMain);

router.get('/room', renderRoom);

router.post('/room', createRoom);

router.get('/room/:id', enterRoom);

router.delete('/room/:id', removeRoom);

module.exports = router;
```

```jsx
const Room = require('../schemas/room');
const Chat = require('../schemas/chat');

exports.renderMain = async (req, res, next) => {
  try {
    const rooms = await Room.find({});
    res.render('main', { rooms, title: 'GIF 채팅방' });
  } catch (error) {
    console.error(error);
    next(error);
  }
};

exports.renderRoom = (req, res) => {
  res.render('room', { title: 'GIF 채팅방 생성' });
};

exports.createRoom = async (req, res, next) => { // 채팅방을 만드는 컨트롤러
  try {
    const newRoom = await Room.create({
      title: req.body.title,
      max: req.body.max,
      owner: req.session.color,
      password: req.body.password,
    });
    const io = req.app.get('io'); // app.set()로 저장했던 io 객체를 가져옴
    io.of('/room').emit('newRoom', newRoom);
    // /room 네임스페이스에 연결한 모든 클라이언트에 데이터를 보내는 메서드
    if (req.body.password) { // 비밀번호가 있는 방이면
      res.redirect(`/room/${newRoom._id}?password=${req.body.password}`);
    } else {
      res.redirect(`/room/${newRoom._id}`);
    }
  } catch (error) {
    console.error(error);
    next(error);
  }
};

exports.enterRoom = async (req, res, next) => {
// 채팅방에 접속해 채팅방 화면을 렌더링하는 컨트롤러
  try {
    const room = await Room.findOne({ _id: req.params.id });
    if (!room) {
      return res.redirect('/?error=존재하지 않는 방입니다.');
    }
    if (room.password && room.password !== req.query.password) {
      return res.redirect('/?error=비밀번호가 틀렸습니다.');
    }
    const io = req.app.get('io');
    const { rooms } = io.of('/chat').adapter;
    console.log(rooms, rooms.get(req.params.id), rooms.get(req.params.id));
    if (room.max <= rooms.get(req.params.id)?.size) {
      return res.redirect('/?error=허용 인원이 초과하였습니다.');
    }
    return res.render('chat', {
      room,
      title: room.title,
      chats: [],
      user: req.session.color,
    });
  } catch (error) {
    console.error(error);
    return next(error);
  }
};

exports.removeRoom = async (req, res, next) => { // 채팅방을 삭제하는 컨트롤러
  try {
    await Room.deleteOne({ _id: req.params.id });
    await Chat.deleteMany({ room: req.params.id });
    res.send('ok');
  } catch (error) {
    console.error(error);
    next(error);
  }
};
```

# 12.5 미들웨어와 소켓 연결하기

- 방에 입장할 때와 퇴장할 때 채팅방의 다른 사람에게 시스템 메시지를 보내기
- 모든 사람이 방에서 나가면 방을 DB에서 제거

- 사용자의 이름을 채팅방에 표시

```jsx
...
connect();

const sessionMiddleware = session({
  resave: false,
  saveUninitialized: false,
  secret: process.env.COOKIE_SECRET,
  cookie: {
    httpOnly: true,
    secure: false,
  },
});
...
app.use(cookieParser(process.env.COOKIE_SECRET));
app.use(sessionMiddleware);
...
const server = app.listen(app.get('port'), () => {
  console.log(app.get('port'), '번 포트에서 대기중');
});

webSocket(server, app, sessionMiddleware);
```

```jsx
const SocketIO = require('socket.io');
const { removeRoom } = require('./services');

module.exports = (server, app, sessionMiddleware) => {
  const io = SocketIO(server, { path: '/socket.io' });
  app.set('io', io);
  const room = io.of('/room');
  const chat = io.of('/chat');

  const wrap = middleware => (socket, next) => middleware(socket.request, {}, next);
  // chat 네임스페이스에 웹 소켓이 연결될 때마다 실행
  chat.use(wrap(sessionMiddleware)); // 미들웨어에 익스프레스처럼 req, res, next를 제공

  room.on('connection', (socket) => {
    console.log('room 네임스페이스에 접속');
    socket.on('disconnect', () => {
      console.log('room 네임스페이스 접속 해제');
    });
  });

  chat.on('connection', (socket) => {
    console.log('chat 네임스페이스에 접속');

    socket.on('join', (data) => {
      socket.join(data);
      socket.to(data).emit('join', { // 특정 방에 데이터를 보낼 수 있음
        user: 'system',
        chat: `${socket.request.session.color}님이 입장하셨습니다.`,
      });
    });

    socket.on('disconnect', async () => {
      console.log('chat 네임스페이스 접속 해제');
      const { referer } = socket.request.headers; // 브라우저 주소가 들어있음
      const roomId = new URL(referer).pathname.split('/').at(-1);
      // URL 객체를 사용해 방 아이디를 추출해낼 수 있음
      // 방 아이딘느 pathname의 마지막에 위치하기 때문에 at(-1) 메서드로 가져옴
      const currentRoom = chat.adapter.rooms.get(roomId); // 참여 중인 소켓 정보
      const userCount = currentRoom?.size || 0; // 참여자 수는 size 속성으로 구함
      if (userCount === 0) { // 유저가 0명이면 방 삭제
        await removeRoom(roomId); // 컨트롤러 대신 서비스를 사용
        room.emit('removeRoom', roomId); // removeRoom은 컨트롤러가 아닌 서비스
        console.log('방 제거 요청 성공');
      } else { // 참여자 수가 0명이 아니면
        socket.to(roomId).emit('exit', {
          user: 'system',
          chat: `${socket.request.session.color}님이 퇴장하셨습니다.`,
        });
      }
    });
  });
};
```

- removeRoom 서비스 구성

```jsx
const Room = require('../schemas/room');
const Chat = require('../schemas/chat');

exports.removeRoom = async (roomId) => {
  try {
    await Room.deleteOne({ _id: roomId });
    await Chat.deleteMany({ room: roomId });
  } catch (error) {
    throw error;
  }
};
```

- removeRoom 컨트롤러는 removeRoom 서비스를 가져와 사용

```jsx
const Room = require('../schemas/room');
const Chat = require('../schemas/chat');
const { removeRoom: removeRoomService } = require('../services'); 
...
exports.removeRoom = async (req, res, next) => {
  try {
    await removeRoomService(req.params.id);
    res.send('ok');
  } catch (error) {
    console.error(error);
    next(error);
  }
};
```

# 12.6 채팅 구현하기

- 프런트에서는 서버에서 보내는 채팅 데이터를 받을 소켓 리스너가 필요
- chat.html 파일에 추가
    - socket에 chat 이벤트 리스너를 추가
    - chat 이벤트는 채팅 메시지가 웹 소켓으로 전송될 때 호출됨
    - 채팅 메시지 발송자(data.user)에 따라 내 메시지(mine 클래스)인지 남의 메시지(other 클래스)인지 확인한 후 그에 맞게 렌더링
    - 채팅을 전송하는 폼에 submit 이벤트 리스너도 추가

```html
 <button type="submit">전송</button>
  </form>
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io.connect('http://localhost:8005/chat', {
      path: '/socket.io',
    });
    socket.emit('join', new URL(location).pathname.split('/').at(-1));
    socket.on('join', function (data) {
      const div = document.createElement('div');
      div.classList.add('system');
      const chat = document.createElement('div');
      chat.textContent = data.chat;
      div.appendChild(chat);
      document.querySelector('#chat-list').appendChild(div);
    });
    socket.on('exit', function (data) {
      const div = document.createElement('div');
      div.classList.add('system');
      const chat = document.createElement('div');
      chat.textContent = data.chat;
      div.appendChild(chat);
      document.querySelector('#chat-list').appendChild(div);
    });
    socket.on('chat', function (data) {
      const div = document.createElement('div');
      if (data.user === '{{user}}') {
        div.classList.add('mine');
      } else {
        div.classList.add('other');
      }
      const name = document.createElement('div');
      name.textContent = data.user;
      div.appendChild(name);
      if (data.chat) {
        const chat = document.createElement('div');
        chat.textContent = data.chat;
        div.appendChild(chat);
      } else {
        const gif = document.createElement('img');
        gif.src = '/gif/' + data.gif;
        div.appendChild(gif);
      }
      div.style.color = data.user;
      document.querySelector('#chat-list').appendChild(div);
    });
    document.querySelector('#chat-form').addEventListener('submit', function (e) {
      e.preventDefault();
      if (e.target.chat.value) {
        axios.post('/room/{{room._id}}/chat', {
          chat: this.chat.value,
        })
          .then(() => {
            e.target.chat.value = '';
          })
          .catch((err) => {
            console.error(err);
          });
      }
    });
  </script>
{% endblock %}
```

- 방에 접속하는 부분과 채팅을 하는 부분을 만들기
    - enterRoom 컨트롤러에서 방 접속 시 기존 채팅 내역을 불러오도록 수정
    - 방에 접속할 때는 DB로부터 채팅 내역을 가져오고, 접속 후에는 웹 소켓으로 새로운 채팅 메시지를 받음

```jsx
...
exports.renderRoom = (req, res) => {
  res.render('room', { title: 'GIF 채팅방 생성' });
};

exports.createRoom = async (req, res, next) => {
  try {
    const newRoom = await Room.create({
      title: req.body.title,
      max: req.body.max,
      owner: req.session.color,
      password: req.body.password,
    });
    const io = req.app.get('io');
    io.of('/room').emit('newRoom', newRoom);
    if (req.body.password) { // 비밀번호가 있는 방이면
      res.redirect(`/room/${newRoom._id}?password=${req.body.password}`);
    } else {
      res.redirect(`/room/${newRoom._id}`);
    }
  } catch (error) {
    console.error(error);
    next(error);
  }
};

exports.enterRoom = async (req, res, next) => {
  try {
    const room = await Room.findOne({ _id: req.params.id });
    if (!room) {
      return res.redirect('/?error=존재하지 않는 방입니다.');
    }
    if (room.password && room.password !== req.query.password) {
      return res.redirect('/?error=비밀번호가 틀렸습니다.');
    }
    const io = req.app.get('io');
    const { rooms } = io.of('/chat').adapter;
    console.log(rooms, rooms.get(req.params.id), rooms.get(req.params.id));
    if (room.max <= rooms.get(req.params.id)?.size) {
      return res.redirect('/?error=허용 인원이 초과하였습니다.');
    }
    const chats = await Chat.find({ room: room._id }).sort('createdAt');
    return res.render('chat', {
      room,
      title: room.title,
      chats,
      user: req.session.color,
    });
  } catch (error) {
    console.error(error);
    return next(error);
  }
};
...
exports.sendChat = async (req, res, next) => {
  try {
    const chat = await Chat.create({
      room: req.params.id,
      user: req.session.color,
      chat: req.body.chat,
    });
    req.app.get('io').of('/chat').to(req.params.id).emit('chat', chat);
    res.send('ok');
  } catch (error) {
    console.error(error);
    next(error);
  }
}
```

```jsx
const express = require('express');
const {
  renderMain, renderRoom, createRoom, enterRoom, removeRoom, sendChat,
} = require('../controllers');

const router = express.Router();

router.get('/', renderMain);

router.get('/room', renderRoom);

router.post('/room', createRoom);

router.get('/room/:id', enterRoom);

router.delete('/room/:id', removeRoom);

router.post('/room/:id/chat', sendChat);

module.exports = router;
```

# 12.7 프로젝트 마무리하기

- GIF 이미지를 전송하는 것을 구현하기
- 프런트 화면에서 이미지를 선택해 업로드하는 이벤트 리스너를 추가

```html
document.querySelector('#chat-form').addEventListener('submit', function (e) {
...
		});
		document.querySelector('#gif').addEventListener('change', function (e) {
      console.log(e.target.files);
      const formData = new FormData();
      formData.append('gif', e.target.files[0]);
      axios.post('/room/{{room._id}}/gif', formData)
        .then(() => {
          e.target.file = null;
        })
        .catch((err) => {
          console.error(err);
        });
    });
  </script>
```

- POST /room/{{room._id}}/gif

```jsx
const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');

const {
  renderMain, renderRoom, createRoom, enterRoom, removeRoom, sendChat, sendGif,
} = require('../controllers');
...
router.post('/room/:id/chat', sendChat);

try {
  fs.readdirSync('uploads');
} catch (err) {
  console.error('uploads 폴더가 없어 uploads 폴더를 생성합니다.');
  fs.mkdirSync('uploads');
}
const upload = multer({
  storage: multer.diskStorage({
    destination(req, file, done) {
      done(null, 'uploads/');
    },
    filename(req, file, done) {
      const ext = path.extname(file.originalname);
      done(null, path.basename(file.originalname, ext) + Date.now() + ext);
    },
  }),
  limits: { fileSize: 5 * 1024 * 1024 },
});
router.post('/room/:id/gif', upload.single('gif'), sendGif);

module.exports = router;
```

```jsx
...
exports.sendGif = async (req, res, next) => {
  try {
    const chat = await Chat.create({
      room: req.params.id,
      user: req.session.color,
      gif: req.file.filename,
    });
    req.app.get('io').of('/chat').to(req.params.id).emit('chat', chat);
    res.send('ok');
  } catch (error) {
    console.error(error);
    next(error);
  }
};
```

- uploads 폴더에 사진을 저장하고, 파일명에 타임스탬프(Date.now())를 붙이고, 5MB로 용량을 제한
- 파일이 업로드된 후에는 내용을 데이터베이스에 저장
- 방 안에 있는 모든 소켓에 채팅 데이터를 보냄

- 이미지를 제공할 uploads 폴더를 express.static 미들웨어로 연결
```jsx
...
app.use(express.static(path.join(__dirname, 'public')));
app.use('/gif', express.static(path.join(__dirname, 'uploads')));
app.use(express.json());
...
```