# 4장 : http 모듈로 서버 만들기

# 4.1 요청과 응답 이해하기

- 클라이언트에서 서버로 요청을 보내고, 서버에서는 요청의 내용을 읽고 처리한 뒤 클라이언트에 응답을 보냄 → 서버에는 요청을 받는 부분과 응답을 보내는 부분이 있어야 함

```jsx
const http = require('http');

http.createServer((req, res) => {
	// 여기에 어떻게 응답할지 적어줌
});
```

- http 모듈에 createServer 메서드 : 인수로 요청에 대한 콜백 함수를 넣을 수 있으며, 요청이 들어올 때마다 매번 콜백 함수가 실행됨
- createServer의 콜백 부분을 보면 req와 res 매개변수가 있음
    - 보통 request를 줄여 req라 표현하고, response를 줄여 res라 표현
    - req 객체는 요청에 관한 정보들을, res 객체는 응답에 관한 정보들을 담고 있음
- 아직은 코드를 실행해도 아무 일도 일어나지 않음 : 요청에 대한 응답도 넣어주지 않았고 서버와 연결하지도 않았기 때문

```jsx
const http = require('http');

http.createServer((req, res) => {
	res.writeHead(200, { 'Content=Type': 'text/html; charset=utf-8' });
	res.write('<h1>Hello Node!</h1>');
	res.end('<p>Hello Server!</p>');
});
	.listen(8000, () => { // 서버 연결
	console.log('8080번 포트에서 서버 대기 중입니다!');
});
```

- 콘솔에 node server1을 입력하여 서버를 실행
- 서버를 종료하려면 콘솔에서 Ctrl+C를 입력
- createServer 메서드 뒤에 listen 메서드를 붙이고 클라이언트에 공개할 포트 번호와 포트 연결 완료 후 실행될 콜백 함수를 넣음 → 이 파일을 실행하면 서버는 8080번 포트에서 요청이 오기를 기다림
- res 객체에는 3개의 메서드가 있음
    - `res.writeHead` : 응답에 대한 정보를 기록하는 메서드, 첫 번째 인수로 성공적인 요청임을 의미하는 200을, 두 번째 인수로 응답에 대한 정보를 보내는데 이 정보가 기록되는 부분을 헤더(header)라고 함
    - `res.write` : 첫 번째 인수는 클라이언트로 보낼 데이터, 여러 번 호출해서 데이터를 여러 개 보내도 되는데 데이터가 기록되는 부분을 본문(body)라고 함
    - `res.end` : 응답을 종료하는 메서드, 만약 인수가 있다면 그 데이터도 클라이언트로 보내고 응답을 종료
- res.write와 res.end에 일일이 HTML을 적는 것은 비효율적이므로 미리 HTML 파일을 만들어두는 것이 바람직

- HTTP 상태 코드
    - 2XX : 성공을 알리는 상태 코드
    - 3XX : 리다이렉션을 알리는 상태 코드
    - 4XX :요청 오류를 나타냄
    - 5XX : 서버 오류를 나타냄(요청은 제대로 왔지만 서버에 오류가 생겼을 때 발생)

# 4.2 REST와 라우팅 사용하기

- 서버에 요청을 보낼 때는 주소를 통해 요청의 내용을 표현
    - 주소가 /index.html이면 서버의 index.html을 보내달라는 뜻
- 항상 html만 요청할 필요는 없음, css나 js 또는 이미지 같은 파일을 요청할 수도 있고 특정 동작을 행하는 것을 요청할 수도 있음
- 요청의 내용이 주소를 통해 표현되므로 서버가 이해하기 쉬운 주소를 사용하는 것이 좋음 → 여기서 REST가 등장
- **REST(REpresentational State Transfer)** : 서버의 자원을 정의하고 자원에 대한 주소를 지정하는 방법을 가리킴
    - 자원이라고 해서 꼭 파일일 필요는 없음
    - 서버가 행할 수 있는 모든 것들을 통틀어서 의미
- 주소는 의미를 명확히 전달하기 위해 명사로 구성
    - /user라면 사용자 정보에 관련된 자원을 요청하는 것
    - /post라면 게시글에 관련된 자원을 요청하는 것
- REST에서는 주소 외에도 HTTP 요청 메서드라는 것을 사용
    - `GET` : 서버 자원을 가져오고자 할 때 사용, 요청의 본문에 데이터를 넣지 않고 데이터를 서버로 보내야 한다면 쿼리스트링을 사용
    - `POST` : 서버에 자원을 새로 등록하고자 할 때 사용, 요청의 본문에 새로 등록할 데이터를 넣어 보냄
    - `PUT` : 서버의 자원을 요청에 들어 있는 자원으로 치환하고자 할 때 사용, 요청의 본문에 치환할 데이터를 넣어 보냄
    - `PATCH` : 서버 자원의 일부만 수정하고자 할 때 사용, 요청의 본문에 일부 수정할 데이터를 넣어 보냄
    - `DELETE` : 서버의 자원을 삭제하고자 할 때 사용, 요청의 본문에 데이터를 넣지 않음
    - `OPTIONS` : 요청을 하기 전에 통신 옵션을 설명하기 위해 사용
- 주소 하나가 요청 메서드를 여러 개 가질 수 있음
- HTTP 통신을 사용하면 클라이언트가 누구든 상관없이 같은 방식으로 서버와 소통 가능 → 서버와 클라이언트가 분리되어 있다는 뜻
    - 서버와 클라이언트를 분리하면 추후에 서버를 확장할 때 클라이언트에 구애되지 않아 좋음

# 4.3 쿠키와 세션 이해하기

- 사용자가 누구인지 기억하기 위해 서버는 요청에 대한 응답을 할 때 쿠키라는 것을 같이 보냄
- 쿠키는 유효 기간이 있으며 단순한 ‘키-값’ 의 쌍임
- 서버로부터 쿠키가 오면, 웹 브라우저는 쿠키를 저장해뒀다가 다음에 요청할 때마다 쿠키를 동봉해서 보냄
- 서버는 요청에 들어 있는 쿠키를 읽어서 사용자가 누구인지 파악
- 쿠키는 요청의 헤더(Cookie)에 담겨 전송됨
- 브라우저는 응답의 헤더(Set-Cookie)에 따라 쿠키를 저장

 

```jsx
const http = require('http');

http.createServer((req, res) => {
	console.log(req.url, req.headers.cookie);
	res.writeHead(200, { 'Set-Cookie': 'mycookie=test' });
	res.end('Hello Cookie');
});
	.listen(8083, () => {
		console.log('8083 포트에서 서버 대기 중입니다!');
});
```

- 쿠키명=쿠키값 : 기본적인 쿠키의 값
- `Expires=날짜` : 만료 기한, 이 기한이 지나면 쿠키가 제거됨, 기본값은 클라이언트가 종료될 때까지
- `Max-age=초` : Expires와 비슷하지만 날짜 대신 초를 입력할 수 있음, 해당 초가 지나면 쿠키가 제거됨, Expires보다 우선
- `Domain=도메인명` : 쿠키가 전송될 도메인을 특정할 수 있음, 기본값은 현재 도메인
- `Patch=URL` : 쿠키가 전송될 URL을 특정할 수 있음, 기본값은 ‘/’이고 이 경우 모든 URL에서 쿠키를 전송할 수 있음
- `Secure` : HTTPS일 경우에만 쿠키가 전송됨
- `HttpOnly` : 설정 시 자바스크립트에서 쿠키에 접근할 수 없음, 쿠키 조작을 방지하기 위해 설정하는 것이 좋음

- 세션 쿠키 : 세션을 위해 사용하는 쿠키
- 실제 배포용 서버에서는 보통 세션을 레디스(Redis)나 멤캐시드(Memcached) 같은 데이터베이스에 넣어둠

# 4.4 https와 http2

- https 모듈은 웹 서버에 SSL 암호화를 추가
- GET이나 POST 요청을 할 때 오가는 데이터를 암호화해서 중간에 다른 사람이 요청을 가로채더라도 내용을 확인할 수 없게 함

```jsx
const http = require('http');

http.createServer((req, res) => {
	cert: fs.readFileSync('도메인 인증서 경로');
	key: fs.readFileSync('도메인 비밀 키 경로');
	ca: [
		fs.readFileSync('상위 인증서 경로');
		fs.readFileSync('상위 인증서 경로');
	],
}, (res, req) => {
	res.writeHead(200, {'Content-Type' : 'text/html; charset=utf-8' });
	res.write('<h1>Hello Node!</h1>');
	res.end('<p>Hello Server!</p>');
})
	.listen(443, () => {
		console.log('443번 포트에서 서버 대기 중입니다!');
});
```

- createServer 메서드가 인수를 두 개 받음
    - 두 번째 인수는 http 모듈과 같이 서버 로직
    - 첫 번째 인수는 인증서에 관련된 옵션 객체
    - 인증서를 구입하면 pem이나 crt, key 확장자를 가진 파일들을 제공
- **http2 모듈**은 SSL 암호화와 더불어 최신 HTTP 프로토콜인 http/2를 사용할 수 있게 함
    - http/2는 요청 및 응답 방식이 기존 http/1.1보다 개선되어 훨씬 효율적으로 요청을 보냄
    - http/2를 사용하면 웹의 속도도 많이 개선됨

# 4.5 cluster

- cluster 모듈은 기본적으로 싱글 프로세스로 동작하는 노드가 CPU 코어를 모두 사용할 수 있게 해주는 모듈
- 포트를 공유하는 노드 프로세스를 여러 개 둘 수도 있어, 요청이 많이 들어왔을 때 병렬로 실행된 서버의 개수만큼 요청이 분산되게 할 수도 있음 → 서버에 무리가 덜 가게 되는 셈
- ex. 코어가 여덟 개인 서버가 있을 때, 노드는 보통 코어를 하나만 활용
    - 하지만 cluster 모듈을 설정해 코어 하나당 노드 프로세스 하나가 돌아가게 할 수 있음
    - 성능이 꼭 여덟 배가 되는 것은 아니지만, 코어를 하나만 사용할 때에 비해 성능이 개선됨
    - 하지만 메모리를 공유하지 못하는 등의 단점이 존재
    - 세션을 메모리에 저장하는 경우 문제가 될 수 있으며, 이는 레디스 등의 서버를 도입해 해결 가능

```jsx
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;
if (cluster.isMaster) {
	console.log(`마스터 프로세스 아이디: ${process.pid}`);
	// CPU 개수만큼 워커를 생산
	for (let i = 0; i < numCPUs ; i +=1) {
		cluster.fork();
	}
	// 워커가 종료되었을 때
	cluster.on('exit', (worker, code, signal) => {
		console.log(`${worker.process.pid}번 워커가 종료되었습니다.`);
		console.log('code', code, 'signal', signal);
	});
} else {
	// 워커들이 포트에서 대기
	http.createServer((req, res) => {
		res.writeHead(200, { 'Content-type' : 'text/html; charset=utf-8' });
		res.write('<h1>Hello Node!</h1>');
		res.end('<p>Hello Cluster!</P>');
		setTimeout(() => { // 워커가 존재하는지 확인하기 위해 1초마다 강제 종료
			process.exit(1);
		}, 1000);
	}).listen(8086);
	
	console.log(`${process.pid}(번 워커 실행`);
}
```

- 워커 하나가 종료될 때마다 새로운 워커 하나가 생겨남
    - 이러한 방식으로 오류를 처리하려는 것은 좋지 않은 생각
    - 오류 자체의 원인을 찾아 해결해야 함
    - 그래도 예기치 못한 에러로 인해 서버가 종료되는 현상을 방지할 수 있어 클러스터링을 적용해두는 것이 좋음
- Express 모듈은 다른 사람들이 만들어둔 모듈이므로 설치해야 사용 가능