# 3장 : 노드 기능 알아보기

# 3.1 REPL 사용하기

- 자바스크립트는 스크립트 언어이므로 미리 컴파일하지 않아도 즉석에서 코드를 실행할 수 있음
- **REPL**(Read Eval Print Loop) : 입력한 코드를 읽고, 해석하고, 결과물을 반환하고, 종료할 때까지 반복하는 콘솔
- 윈도우에서는 명령 프롬프트, 맥이나 리눅스에서는 터미널을 열고 `node`를 입력하고 사용 가능

# 3.2 JS 파일 실행하기

- 자바스크립트 파일을 만들어 실행하기

```powershell
$ node [자바스크립트 파일 경로]
```

# 3.3 모듈로 만들기

- **모듈** : 특정한 기능을 하는 함수나 변수들의 집합
- 모듈은 자체로도 하나의 프로그램이면서 다른 프로그램의 부품으로도 사용할 수 있음
- 보통 파일 하나가 모듈 하나가 되며, 파일별로 코드를 모듈화할 수 있어 관리하기 편함
- 노드에서는 두 가지 형식의 모듈을 제공 : CommonJS 모듈, ECMAScript 모듈

## 3.3.1 CommonJS 모듈

- CommonJS 모듈은 표준 자바스크립트 모듈은 아니지만 노드 생태계에서 가장 널리 쓰임

→ 표준이 나오기 이전부터 쓰였기 때문

```jsx
const odd = 'CJS 홀수입니다.';
const even = 'CJS 짝수입니다.';

module.exports = {
	odd,
	even,
};
```

```jsx
const { odd, even } = require('./var'); // 구조 분해 할당 문법

function checkOddOrEven(num) {
	if(num % 2) { // 홀수이면
		return odd;
	}
	return even;
}

module.exports = checkOddOrEven;
```

- var.js에 변수 두 개를 선언하고, `module.exports`에 변수들을 담은 객체를 대입 → 파일이 모듈로서 기능
- func.js에서 `require` 함수 안에 불러올 모듈의 경로를 작성
    - 다른 폴더에 있는 파일도 모듈로 사용 가능
    - 파일 경로에서 js나 json 같은 확장자는 생략할 수 있음
- 다른 모듈을 사용하는 파일을 다시 모듈로 만들 수 있음
    - `module.exports`에 다시 `checkOddOrEven` 함수를 대입

```jsx
const { odd, even } = require('./var');
const checkNumber = require('./func');

function checkStringOddOrEven(str) {
	if (str.length % 2) { //
		return odd;
	}
	return even;
}

console.log(checkNumber(10));
console.log(checkStringOddOrEven('hello'));
```

- index.js는 var.js와 func.js를 모두 참조
- var.js가 두 번 쓰이는 것처럼, 모듈 하나가 여러 개의 모듈에 사용될 수 있음
- 모듈로부터 값을 불러올 때 변수 이름을 다르게 지정할 수도 있음
- 모듈이 많아지고 모듈 간의 관계가 얽히게 되면 구조를 파악하기 어렵다는 단점
- `exports.odd` 처럼 exports 객체로 모듈을 만들 수 있음
    - module.exports와 exports가 같은 객체를 참조하기 때문
- require : require.cache, require.main
    - require.cache 객체에 파일 이름이 속성명으로 들어옴(속성값으로는 각 파일의 모듈 객체)
    - 한 번 require한 파일은 `require.cache`에 저장 → 다음번에 require할 때는 새로 불러오지 않고 require.cache에 있는 것이 재사용됨
    - require.main은 노드 실행 시 첫 모듈을 가리킴
    - 현재 첫 모듈인지 알아보려면 `require.main === module`을 해보면 됨
    - 첫 모듈의 이름을 알아보려면 `require.main.filename`으로 확인

- **순환 참조(circular dependency)** : module.exports가 함수가 아니라 빈 객체로 표시됨

## 3.3.2 ECMAScript 모듈

- ECMAScript 모듈은 공식적인 자바스크립트 모듈 형식

```jsx
export const odd = 'MJS 홀수입니다.';
export const even = 'MJS 짝수입니다.';
```

```jsx
import { odd, even } from './var.mjs';

function checkOddOrEven(num) {
	if (num % 2) {
		return odd;
	}
	return even;
}

export default checkOddOrEven;
```

```jsx
import { odd, even } from './var.mjs';
import checkNumber from './func.mjs';

function checkStringOddOrEven(str) {
	if (str.length % 2) {
		return odd;
	}
	return even;
}

console.log(checkNumber(10));
console.log(checkStringOddOrEven('hello'));
```

- 파일이 js 대신 mjs 확장자로 변경
- import 시 파일 경로에서 js, mjs와 같은 확장자 생략 불가
- CommonJS와 ES 모듈은 서로 간에 잘 호환되지 않으므로 웬만하면 한 가지 형식만 사용하는 것을 권장

## 3.3.3 다이내믹 임포트

- CommonJs 모듈에서는 다이내믹 임포트(dynamic import, 동적 불러오기)가 되는데 ES 모듈에서는 다이내믹 임포트가 불가
- **다이내믹 임포트** : 조건부로 모듈을 불러오는 것
- ES 모듈은 if 문 안에서 import 하는 것이 불가능 → 이럴 때 다이내믹 임포트 사용
    - import는 promise를 반환하므로 await이나 then을 붙여야 함
    - ES 모듈의 최상위 스코프에서는 async 함수 없이도 await할 수 있음

## 3.3.4 __filename, __dirname

- 노드에서는 파일 사이에 모듈 관계가 있는 경우가 많으므로 현재 파일의 경로나 파일명을 알아야 하는 경우가 있음
- 노드는 __filename, __dirname이라는 키워드로 경로에 대한 정보 제시
    - 실행 시 현재 파일명과 현재 파일 경로로 바뀜
- ES 모듈에서는 __filename, __dirname을 사용할 수 없음
    - 대신 `import.meta.url`로 경로를 가져올 수 있음

# 3.4 노드 내장 객체 알아보기

- 노드에서는 기본적인 내장 객체와 내장 모듈을 제공
    - 내장 객체와 내장 모듈은 따로 설치하지 않아도 바로 사용 가능

## 3.4.1 global

- 브라우저의 window와 같은 전역 객체 → 모든 파일에서 접근 가능
- window.open에서 window를 생략하는 것처럼 global을 생략 가능
    - 노드 콘솔에 로그를 기록하는 console객체도 원래는 global.console

## 3.4.2 console

- global 객체 안에 들어있음
- console 객체는 보통 디버깅을 위해 사용
    - 개발 중 변수에 값이 제대로 들어 있는지 확인하기 위해 사용
    - 에러 발생 시 에러 내용을 콘솔에 표시하기 위해 사용
    - 코드 실행 시간을 알아볼 때 사용
    - 대표적으로 `console.log` 메소드가 있음
- **console.time(레이블)** : console.timeEnd(레이블)과 대응되어 같은 레이블을 가진 time과 timeEnd 사이의 시간을 측정
- **console.log(내용)** : 평범한 로그를 콘솔에 표시
    - console.log(내용, 내용, …)처럼 여러 내용을 동시에 표시할 수 있음
- **console.error(에러 내용)** : 에러를 콘솔에 표시
- **console.table(배열)** : 배열의 요소로 객체 리터럴을 넣으면 객체의 속성들이 테이블 형식으로 표현됨
- **console.dir(객체, 옵션)** : 객체를 콘솔에 표시할 때 사용
- **console.trace(레이블)** : 에러가 어디서 발생했는지 추적할 수 있게 함

## 3.4.3 타이머

- 타이머 기능을 제공하는 함수인 setTimeout, setInterval, setImmediate는 global 객체 안에 들어 있음
- **setTimeout(콜백 함수, 밀리초)** : 주어진 밀리초(1000분의 1초) 이후에 콜백 함수를 실행
- **setInterval(콜백 함수, 밀리초)** : 주어진 밀리초마다 콜백 함수를 반복 실행
- **setImmediate(콜백 함수)** : 콜백 함수를 즉시 실행
- **clearTimeout(아이디)** : setTimeout을 취소
- **clearInterval(아이디)** : setInterval을 취소
- **clearImmediate(아이디)** : setImmediate를 취소

## 3.4.4 process

- process 객체는 현재 실행되고 있는 노드 프로세스에 대한 정보를 담고 있음

### 3.4.4.1 process.env

- REPL에 process.env를 입력하면 매우 많은 정보가 출력됨
    - 자세히 보면 이 정보들이 시스템의 환경 변수임을 알 수 있음

```jsx
NODE_OPTIONS = --max-old-space-size=8192 // 노드를 실행할 때의 옵션들을 입력받는 환경 변수
UV_THREADPOOL_SIZE = 8 // 노드에서 기본적으로 사용하는 스레드 풀의 스레드 개수를 조절
```

- process.env는 서비스의 중요한 키를 저장하는 공간으로도 사용됨
    - 서버나 데이터베이스의 비밀번호와 각종 API 키를 코드에 직접 입력하는 것은 위험함
    - 따라서 중요한 비밀번호는 process.env의 속성으로 대체
    
    ```jsx
    const secretID = process.env.SECRET_ID;
    const secretCode = process.env.SECRET_CODE;
    ```
    

### 3.4.4.2 process.nextTick(콜백)

- 이벤트 루프가 다른 콜백 함수들보다 nextTick의 콜백 함수를 우선으로 처리하도록 만듦
- **마이크로태스크(microtask)**

### 3.4.4.3 process.exit(코드)

- 실행 중인 노드 프로세스를 종료
- 서버 환경에서 사용하면 서버가 멈추므로 특수한 경우를 제외하고는 서버에서 잘 사용하지 않음
- 서버 외의 독립적인 프로그램에서는 수동으로 노드를 멈추기 위해 사용
- process.exit 메소드는 인수로 코드 번호를 줄 수 있음
    - 인수를 주지 않거나 0을 줌 : 정상 종료
    - 1을 줌 : 비정상 종료(에러가 발생해서 종료하는 경우에 넣음)

## 3.4.5 기타 내장 객체

- **URL**, **URLSearchParams**
- **AbortController**, **FormData**, **fetch**, **Headers**, **Request**, **Response**, **Event**, **EventTarget** : 브라우저에서 사용하던 API가 노드에도 동일하게 생성됨
- **TextDecoder** : Buffer을 문자열로 바꿈
- **TextEncoder** : 문자열을 Buffer로 바꿈
- **WebAssembly** : 웹어셈블리 처리를 담당

# 3.5 노드 내장 모듈 사용하기

- 노드는 웹 브라우저에서 사용되는 자바스크립트보다 더 많은 기능을 제공
- 운영체제 정보에도 접근할 수 있고, 클라이언트가 요청한 주소에 대한 정보도 가져올 수 있음

## 3.5.1 os

- 웹 브라우저에 사용되는 자바스크립트는 운영체제의 정보를 가져올 수 없지만, 노드는 os 모듈에 정보가 담겨 있어 정보를 가져올 수 있음
- 내장 모듈인 os를 불러오려면 `require('os')` 또는 `require('node:os')`를 하면 됨
    - os라는 파일이 존재하는 것은 아니지만 노드가 알아서 내장 모듈임을 파악해 불러옴
- 대표적인 메소드들
    - **os.arch()** : process.arch와 동일
    - **os.platform()** : process.platform과 동일
    - **os.type()** : 운영체제의 종류를 보여줌
    - **os.uptime()** : 운영체제 부팅 이후 흐른 시간(초)을 보여줌(process.uptime()은 노드의 실행 시간)
    - **os.hostname()** : 컴퓨터의 이름을 보여줌
    - **os.release()** : 운영체제의 버전을 보여줌
    - **os.homedir()** : 홈 디렉터리 경로를 보여줌
    - **os.tmpdir()** : 임시 파일 저장 경로를 보여줌
    - **os.cpus()** :  컴퓨터의 코어 정보를 보여줌
    - **os.freemem()** : 사용 가능한 메모리(RAM)을 보여줌
    - **os.totalmem()** : 전체 메모리 용량을 보여줌
- os 모듈은 주로 컴퓨터 내부 자원에 빈번하게 접근하는 경우 사용됨
    - 일반적인 웹 서비스를 제작할 때는 사용 빈도가 높지 않음
    - 운영체제별로 다른 서비스를 제공하고 싶을 때 os 모듈이 유용

## 3.5.2 path

- 폴더와 파일의 경로를 쉽게 조작하도록 도와주는 모듈
- path 모듈이 필요한 이유 중 하나는 운영체제별로 경로 구분자가 다르기 때문 → 크게 윈도 타입과 POSIX 타입으로 구분
    - **윈도** : C:\Users\ZeroCho처럼 \로 구분
    - **POSIX** : /home/zerocho처럼 /로 구분, 유닉스 기반의 운영체제들로 맥과 리눅스가 속해 있음
- **path.sep** : 경로의 구분자(윈도는 \, POSIX는 /)
- **path.delimiter** : 환경 변수의 구분자, process.env.PATH를 입력하면 여러 개의 경로가 이 구분자로 구분되어 있음(윈도는 세미콜론(;), POSIX는 콜론(:))
- **path.dirname(경로)** : 파일이 위치한 폴더 경로를 보여줌
- **path.extname(경로)** : 파일의 확장자를 보여줌
- **path.basename(경로, 확장자)** : 파일의 이름(확장자 포함)을 표시
- **path.parse(경로)** : 파일 경로를 root, dir, base, ext, name으로 분리
- **path.format(객체)** : path.parse()한 객체를 파일 경로로 합침
- **path.normalize(경로)** : /나 \를 실수로 여러 번 사용했거나 혼용했을 때 정상적인 경로로 변환
- **path.isAbsolute(경로)** : 파일의 경로가 절대경로인지 상대경로인지를 true나 false로 알림
- **path.relative(기준경로, 비교경로)** : 경로를 두 개 넣으면 첫 번째 경로에서 두 번째 경로로 가는 방법을 알림
- **path.join(경로, …)** : 여러 인수를 넣으면 하나의 경로로 합침, 상대경로인 `..(부모 디렉터리)`과 `.(현 위치)` 도 알아서 처리
- **path.resolve(경로, …)** : path.join()과 비슷하지만 차이 존재
- 가끔 윈도에서 POSIX 스타일 경로를 사용할 때가 있고, 그 반대일 때도 있음
    - 윈도에서는 `path.posix.sep`과 같이 사용하고, POSIX에서는 `path.win32.sep`과 같이 사용
- 노드는 require.main 파일을 기준으로 상대경로를 인식
    - require.main과는 다른 디렉터리의 파일이 상대경로를 갖고 있다면 예상과 다르게 동작할 수 있음
    - 이 문제는 path 모듈을 통해 해결할 수 있음
- path 모듈은 앞으로 노드 프로그래밍을 하면서 매우 자주 쓰게 될 모듈 중 하나

## 3.5.3 url

- 인터넷 주소를 쉽게 조작하도록 도와주는 모듈
- url 처리에는 크게 두 가지 방법이 존재
    - WHATWG 방식의 url : 웹 표준을 정하는 단체의 이름, 요즘 사용하는 방식, 브라우저에서도 이 방식을 사용하므로 호환성이 좋음
    - 예전부터 노드에서 사용하던 방식의 url
- url 모듈 안에 URL 생성자가 있음
    - URL은 노드 내장 객체이기도 해서 require할 필요는 없음
    - 이 생성자에 주소를 넣어 객체로 만들면 주소가 부분별로 정리됨 → WHATWG의 url
    - username, password, origin, searchParams 속성이 존재
- **url.format(객체)** : 분해되었던 url 객체를 다시 원래 상태로 조립
- 주소가 host 부분 없이 pathname 부분만 오는 경우, WHATWG 방식은 이 주소를 처리할 수 없음 → new URL처럼 두 번째 인수로 host를 적어줘야 함
- search 부분(쿼리스트링)은 보통 주소를 통해 데이터를 전달할 때 사용됨
    - search는 물음표(?)로 시작하고, 그 뒤에 키=값 형식으로 데이터를 전달
    - 여러 키가 있을 경우에는 &로 구분
    - search 부분을 다루기 위해 searchParams라는 특수한 객체가 생성됨
- **getAll(키)** : 키에 해당하는 모든 값을 가져옴. category 키에는 nodejs와 javascript라는 두 가지 값이 들어 있음
- **get(키)** : 키에 해당하는 첫 번째 값만 가져옴
- **has(키)** : 해당 키가 있는지 없는지를 검사
- **keys()** : searchParams의 모든 키를 반복기(iterator) 객체로 가져옴
- **values()** : searchParams의 모든 값을 반복기 객체로 가져옴
- **append(키, 값)** : 해당 키를 추가, 같은 키의 값이 있다면 유지하고 하나 더 추가
- **set(키, 값)** : append와 비슷하지만 같은 키의 값들을 모두 지우고 새로 추가
- **delete(키)** : 해당 키를 제거
- **toString()** : 조작한 searchParams 객체를 다시 문자열로 만듦, 이 문자열을 search에 대입하면 주소 객체에 반영됨

## 3.5.4 dns

- DNS를 다룰 때 사용하는 모듈
- 주로 도메인을 통해 IP나 기타 DNS 정보를 얻고자 할 때 사용
- ip 주소는 간단하게 dns.lookup이나 dns.resolve(도메인)으로 얻을 수 있음
- A(ipv4 주소), AAAA(ipv6 주소), NS(네임서버), SOA(도메인 정보), CNAME(별칭, 주로 www가 붙은 주소는 별칭인 경우가 많음), MX(메일 서버) 등은 레코드라고 부르는데, 해당 레코드에 대한 정보는 dns.resolve(도메인, 레코드 이름)으로 조회하면 됨

## 3.5.5 crypto

- 다양한 방식의 암호화를 도와주는 모듈

### 3.5.5.1 단방향 암호화

- 단방향 암호화 : 복호화할 수 없는 암호화 방식
    - 복호화 : 암호화된 문자열을 원래 문자열로 되돌려놓는 것
    - 단방향 암호화는한번 암호화하면 원래 문자열을 찾을 수 없음
    - 복호화할 수 없으므로 암호화라고 표현하는 대신 **해시 함수**라고 부르기도 함
- 복호화할 수 없는 암호화가 필요한 이유 : 고객의 비밀번호는 복호화할 필요가 없기 때문
    - 고객의 비밀번호를 암호화해서 데이터베이스에 저장한 뒤 로그인할 때 입력받은 비밀번호를 같은 암호화 알고리즘으로 암호화한 후 데이터베이스의 비밀번호와 비교하면 됨
- 단방향 암호화 알고리즘은 주로 해시 기법을 사용
    - **해시 기법** : 어떠한 문자열을 고정된 길이의 다른 문자열로 바꿔버리는 방식

```jsx
const crypto = require('crypto');

console.log('base64: ', crypto.createHash('sha512').update('비밀번호').digest('base64'));
console.log('hex: ', crypto.createHash('sha512').update('비밀번호').digest('hex'));
console.log('base64: ', crypto.createHash('sha512').update('다른 비밀번호').digest('base64'));
```

- **createHash(알고리즘)** : 사용할 해시 알고리즘을 넣음
    - md5, sha1, sha256, sha512 등이 가능하지만, md5와 sha1은 이미 취약점이 발견됨
    - 현재는 sha512 정도로 충분하지만, 나중에 sha512마저도 취약해지면 더 강화된 알고리즘으로 바꿔야 함
- **update(문자열)** : 변환할 문자열을 넣음
- **digest(인코딩)** : 인코딩할 알고리즘을 넣음
    - base64, hex, latin1이 주로 사용됨
    - 그 중 base64가 결과 문자열이 가장 짧아서 애용됨

- 가끔 nopqrst라는 문자열이 qvew로 변환되어 abcdefgh를 넣었을 때와 똑같은 문자열로 바뀔 때가 있음 → 출력이 발생했다고 표현
    - 해킹용 컴퓨터의 역할은 어떠한 문자열이 같은 출력 문자열을 반환하는지 찾아내는 것
- 현재는 주로 pbkdf2나 bcrypt, scrypt라는 알고리즘으로 비밀번호를 암호화하고 있음
- pbkdf2 : 노드에서 지원, 기존 문자열에 salt라고 불리는 문자열을 붙인 후 해시 알고리즘을 반복해서 적용

```jsx
const crypto = require('crypto');

crypto.randomBytes(64, (err, buf) => {
	const salt = buf.toString('base>64');
	console.log('salt:', salt);
	crypto.pbkdf2('비밀번호', salt, 10000, 64, 'sha>512', (err, key) => {
		console.log('password:', key.toString('base>64'));
		});
	});
```

1. randomBytes() 메서드로 64비트 길이의 문자열을 만듦 → salt
2. pbkdf2() 메서드에는 순서대로 비밀번호, salt, 반복 횟수, 출력 바이트, 해시 알고리즘을 인수로 넣음
    1. 예시에서는 10만 번 반복해서 적용 → sha512로 변환된 결괏값을 다시 sha512로 변환하는 과정을 10만 번 반복
    2. 너무 많이 반복하는 건 아닌지 걱정될 수도 있지만, 1초밖에 걸리지 않음
- randomBytes이므로 매번 실행할 때마다 결과가 달라짐 → salt를 잘 보관하고 있어야 비밀번호도 찾을 수 있음
- pbkdf2는 간단하지만 bcrypt나 scrypt보다 취약하므로 나중에 더 나은 보안이 필요하면 bcrypt나 scrypt 방식을 사용하면 됨

 

### 3.5.5.2 양방향 암호화

- 암호화된 문자열을 복화할 수 있음
- 키(열쇠)라는 것이 사용됨
- 대칭형 암호화에서는 암호를 복호화하려면 암호화할 때 사용한 키와 같은 키를 사용해야 함
- **crypto.createCipheriv(알고리즘, 키, iv)** : 암호화 알고리즘과 키, iv를 넣음
- **cipher.update(문자열, 인코딩, 출력 인코딩)** : 암호화할 대상과 대상의 인코딩, 출력 결과물의 인코딩을 넣음(보통 문자열은 utf8 인코딩을, 암호는 base64를 많이 사용)
- **cipher.final(출력 인코딩)** : 출력 결과물의 인코딩을 넣으면 암호화가 완료
- **cipher.createDecipheriv(알고리즘, 키, iv)** : 복호화할 때 사용. 암호화할 때 사용했던 알고리즘과 키, iv를 그대로 넣어야 함
- **dicipher.update(문자열, 인코딩, 출력 인코딩)** : 암호화된 문장, 그 문장의 인코딩, 복호화할 인코딩을 넣음. createCipheriv의 update()에서 utf8, base64 순으로 넣었다면 createDecipheriv의 update()에서는 base64, utf8 순으로 넣음
- **decipher.final(출력 인코딩)** : 복호화 결과물의 인코딩을 넣음

## 3.5.6 util

- util이라는 이름처럼 각종 편의 기능을 모아둔 모듈
- 계속해서 API가 추가되고 있으며 가끔 중요도가 떨어져 더 이상 사용되지 않고 이후에 사라지는 경우도 있음
- **util.deprecate** : 함수가 deprecated 처리되었음을 알림
    - 첫 번째 인수로 넣은 함수를 사용했을 때 경고 메시지가 출력됨
    - 두 번째 인수로 경고 메시지 내용을 넣음
    - 함수가 조만간 사라지거나 변경될 때 알려줄 수 있어 유용
- **util.promisify** : 콜백 패턴을 프로미스 패턴으로 바꿈
    - 바꿀 함수를 인수로 제공하면 됨
    - 바꿔두면 async/await 패턴까지 사용할 수 있어 좋음

## 3.5.7 worker_threads

- 노드에서 멀티 스레드 방식으로 작업할 때 사용하는 모듈

```jsx
const {
	Worker, isMainThread, parentPort,
} = require('worker_threads');

if (isMainThread) { // 부모일 때
	const worker = new Worker(__filename);
	worker.on('message', message => console.log('from worker', message));
	worker.on('exit', () => console.log('worker exit'));
	worker.postMessage('ping');
} else { // 워커일 때
	parentPort.on('message', (value) => {
	parentPort.postMessage('pong');
	parentPort.close();
	});
}
```

- `isMainThread`를 통해 현재 코드가 메인 스레드(부모 스레드)에서 실행되는지, 아니면 우리가 생성한 워커 스레드에서 실행되는지 구분
- 부모에서는 워커 생성 후 `worker.postMessage`로 워커에 데이터를 보낼 수 있음
- 워커는 `parentPort.on('message')` 이벤트 리스너로 부모로부터 메시지를 받고 `parentPort.postMessage`로 부모에게 메시지를 보냄
- 부모는 `worker.on('message')`로 메시지를 받음
- 워커에서 on 메서드를 사용할 대는 직접 워커를 종료해야 함
    - `parentPort.close()`를 하면 부모와의 연결이 종료됨

## 3.5.8 child_process

- 노드에서 다른 프로그램을 실행하고 싶거나 명령어를 수행하고 싶을 때 사용하는 모듈
- 이 모듈을 통해 다른 언어의 코드(파이썬 등)를 실행하고 결괏값을 받을 수 있음
- exec과 spawn의 차이
    - exec : 셸을 실행해서 명령어를 수행
    - spawn : 새로운 프로세스를 띄우면서 명령어를 실행
    - 셸을 실행하는지 마는지에 따라 수행할 수 있는 명령어에 차이가 있음

## 3.5.9 기타 모듈들

- **async_hooks** : 비동기 코드의 흐름을 추적할 수 있는 실험적인 모듈
- **dgram** : UDP와 관련된 작업을 할 때 사용
- **net** : HTTP보다 로우 레벨인 TCP나 IPC 통신을 할 때 사용
- **perf_hooks** : 성능 측정을 할 때 console.time보다 더 정교하게 측정
- **querystring** : URLSearchParams가 나오기 이전에 쿼리스트링을 다루기 위해 사용했던 모듈
- **string_decoder** : 버퍼 데이터를 문자열로 바꾸는 데 사용
- **tls** : TLS와 SSL에 관련된 작업을 할 때 사용
- **tty** : 터미널과 관련된 작업을 할 때 사용
- **v8** : v8 엔진에 직접 접근할 때 사용
- **vm** : 가상 머신에 직접 접근할 때 사용
- **wasi** : 웹어셈블리를 실행할 때 사용하는 실험적인 모듈

# 3.6 파일 시스템 접근하기

- fs 모듈은 파일 시스템에 접근하는 모듈(파일을 생성하거나 삭제, 읽거나 쓸 수 있음)

```
저를 읽어주세요.
```

```jsx
const fs = require('fs');

fs.readFile('./readme.txt', (err, data) => {
	if (err) {
		throw err;
	}
	console.log(data);
	console.log(data.toString());
});
```

- fs 모듈을 불러온 뒤 읽을 파일의 경로를 지정
    - 파일의 경로가 현재 파일 기준이 아니라 node 명령어를 실행하는 콘솔 기준이라는 것에 유의
- 파일을 읽은 후에 실행될 콜백 함수도 readFile 메서드의 인수로 같이 넣음
- readFile의 결과물은 버퍼(buffer)라는 형식으로 제공됨
    - 버퍼 : 메모리의 데이터

## 3.6.1 동기 메서드와 비동기 메서드

- setTimeout같은 타이머와 process.nextTick 외에도 노드는 대부분의 메서드를 비동기 방식으로 처리
- 하지만 몇몇 메서드는 동기 방식으로도 사용 가능 → 특히 fs 모듈이 그러한 메서드를 많이 가짐
- 비동기 메서드들은 백그라운드에 해당 파일을 읽으라고만 요청하고 다음 작업으로 넘어감 → 파일 읽기 요청만 세 번을 보내고 console.log(’끝’)을 찍음
- 동기 메서드들은 이름 뒤에 Sync가 붙어 있어 구분하기 쉬움

## 3.6.2 버퍼와 스트림 이해하기

- 파일을 읽거나 쓰는 방식에는 크게 두 가지 방식이 존재 : 버퍼와 스트림
- 버퍼링 : 영상을 재생할 수 있을 때까지 데이터를 모으는 동작
- 스트리밍 : 방송인의 컴퓨터에서 시청자의 컴퓨터로 영상 데이터를 조금씩 전송하는 동작
    - 스트리밍하는 과정에서 버퍼링을 할 수도 있음 → 전송이 너무 느리면 화면을 내보내기까지 최소한의 데이터를 모아야 하고, 영상 데이터가 재생 속도보다 빨리 전송되어도 미리 전송 받은 데이터를 저장할 공간이 필요하기 때문
- 노드는 파일을 읽을 때 메모리에 파일 크기만큼 공간을 마련해두며 파일 데이터를 메모리에 저장한 뒤 사용자가 조작할 수 있도록 함 → 이때 메모리에 저장된 데이터가 바로 버퍼
- Buffer 객체는 여러 가지 메서드를 제공
    - **from(문자열)** : 문자열을 버퍼로 바꿀 수 있음, length 속성은 버퍼의 크기를 알림, 바이트 단위
    - **toString(버퍼)** : 버퍼를 다시 문자열로 바꿀 수 있음. 이때 base64나 hex를 인수로 넣으면 해당 인코딩으로 변환 가능
    - **concat(배열)** : 배열 안에 든 버퍼들을 하나로 합침
    - **alloc(바이트)** : 빈 버퍼를 생성, 바이트를 인수로 넣으면 해당 크기의 버퍼가 생성됨
- readFile 방식의 버퍼가 편리하기는 하지만 문제점도 존재
    - 만약 용량이 100MB인 파일이 있으면, 읽을 때 메모리에 100MB의 버퍼를 만들어야 함, 이 작업을 동시에 열 개만 해도 1GB에 달하는 메모리가 사용됨, 서버처럼 몇 명이 이용할지 모르는 환경에서는 메모리 문제 발생 가능
    - 모든 내용을 버퍼에 다 쓴 후에야 다음 동작으로 넘어가므로 파일 읽기, 압축, 파일 쓰기 등의 조작을 연달아 할 때 매번 전체 용량을 버퍼로 처리해야 다음 단계로 넘어갈 수 있음
- 그래서 버퍼의 크기를 작게 만들고 여러 번에 걸쳐 나눠 보내는 방식이 등장
    - ex. 버퍼 1MB를 만든 후 100MB 파일을 100번에 걸쳐 나눠 보내는 것
    - 메모리 1MB로 100MB 파일을 전송할 수 있음 ⇒ **스트림**
- 파일을 읽는 스트림 메서드로는 **createReadStream**이 존재

## 3.6.3 기타 fs 메서드 알아보기

- fs는 파일 시스템을 조작하는 다양한 메서드를 제공
- **fs.access(경로, 옵션, 콜백)** : 폴더나 파일에 접근할 수 있는지를 체크
    - `F_OK` : 파일 존재 여부 체크
    - `R_OK` : 읽기 권한 여부 체크
    - `W_OK` : 쓰기 권한 여부 체크
    - `ENOENT` : 파일/폴더나 권한이 없을 경우 발생하는 에러 코드
- **fs.mkdir(경로, 콜백)** : 폴더를 만드는 메서드, 이미 폴더가 있다면 에러가 발생하므로 먼저 access 메서드를 호출해서 확인하는 것이 중요
- **fs.open(경로, 옵션, 콜백)** : 파일의 아이디를 가져오는 메서드, 파일이 없다면 파일을 생성한 뒤 그 아이디를 가져옴
- **fs.rename(기존 경로, 새 경로, 콜백)** : 파일의 이름을 바꾸는 메서드, 기존 파일 위치와 새로운 파일 위치를 적으면 됨
- **fs.readdir(경로, 콜백)** : 폴더 안의 내용물을 확인
- **fs.unlink(경로, 콜백)** : 파일을 지울 수 있음
- **fs.rmdir(경로, 콜백)** : 폴더를 지울 수 있음, 폴더 안에 파일들이 있다면 에러가 발생하므로 먼저 내부 파일을 모두 지우고 호출해야 함

## 3.6.4 스레드 풀 알아보기

- fs 외에도 내부적으로 스레드 풀을 사용하는 모듈 : crypto, zlib, dns.lookup 등
- 윈도라면 명령 프롬프트에 `SET UV_THREADPOOL_SIZE=1`을, 맥과 리눅스라면 터미널에 `UV_THREADPOOL_SIZE=1`을 입력한 후 다시 `node threadpool` 명령어를 입력하면 작업이 순서대로 실행됨 → 스레드 풀 개수를 하나로 제한했기 때문

# 3.7 이벤트 이해하기

- createStream 같은 경우 내부적으로 알아서 data와 end 이벤트를 호출하지만, 우리가 직접 이벤트를 만들 수도 있음
- event 모듈로 만든 객체는 이벤트 관리를 위한 메서드를 갖고 있음
    - **on(이벤트명, 콜백)** : 이벤트 이름과 이벤트 발생 시의 콜백을 연결, 이렇게 연결하는 동작을 이벤트리스닝이라고 함, event2처럼 이벤트 하나에 이벤트 여러 개를 달아줄 수도 있음
    - **addListener(이벤트명, 콜백)** : on과 기능이 같음
    - **emit(이벤트명)** : 이벤트를 호출하는 메서드, 이벤트 이름을 인수로 넣으면 미리 등록해뒀던 이벤트 콜백이 실행됨
    - **once(이벤트명, 콜백)** : 한 번만 실행되는 이벤트
    - **removeAllListeners(이벤트명)** : 이벤트에 연결된 모든 이벤트 리스너를 제거
    - **removeListener(이벤트명, 리스너)** : 이벤트에 연결된 리스너를 하나씩 제거
    - **off(이벤트명, 콜백)** : 노드 10 버전에서 추가된 메서드, removeListener와 기능이 같음
    - **listenerCount(이벤트명)** : 현재 리스너가 몇 개 연결되어 있는지 확인

# 3.8 예외 처리하기

- 멀티 스레드 프로그램에서는 스레드 하나가 멈추면 그 일을 다른 스레드가 대신하지만 노드의 메인 스레드는 하나 뿐이므로 그 하나를 소중히 보호해야 함
- 메인스레드가 에러로 인해 멈춘다는 것은 스레드를 갖고 있는 프로세스가 멈춘다는 뜻이고, 전체 서버도 멈춘다는 뜻과 같음
- 프로세스가 멈추지 않도록 에러를 잡는 방법 : 에러가 발생할 것 같은 부분을 try/catch문으로 감싸줌
- 노드 자체에서 잡아주는 에러 : .fs.unlink로 존재하지 않는 파일을 지울 때 에러가 발생하지만 노드 내장 모듈의 에러는 실행 중인 프로세스를 멈추지 않음
- 에러가 발생했을 때 에러를 throw하면 노드 프로세스가 멈춰버림 → throw하는 경우에는 반드시 try/catch 문으로 throw한 에러를 잡아야 함
- 예측이 불가능한 에러를 처리하는 방법 : uncaughtException 이벤트 리스너를 사용
    - 하지만 노드 공식 문서에서는 uncaughtException 이벤트를 최후의 수단으로 사용할 것을 명시
    - 노드는 uncaughtException 이벤트 발생 후 다음 동작이 제대로 동작하는지를 보증하지 않음 → uncaughtException은 단순히 에러 내용을 기록하는 정도로 사용하고, 에러를 기록한 후 process.exit()으로 프로세스를 종료하는 것이 좋음
    - 에러가 발생하는 코드를 수정하지 않는 이상, 프로세스가 실행되는 동안 에러는 계속 발생할 것

## 3.8.1 자주 발생하는 에러들

- `node: command not found` : 노드를 설치했지만 이 에러가 발생하는 이유는 환경 변수가 제대로 설정되어 있지 않은 것, 환경 변수에는 노드가 설치된 경로가 포함되어야 함
- `ReferenceError: 모듈 is not defined` : 모듈을 require했는지 확인
- `Error: Cannot find module 모듈명` : 해당 모듈을 require했지만 설치하지 않음, npm i 명령어로 설치
- `Error [ERR_MODULE_NOT_FOUND]` : 존재하지 않는 모듈을 불러오려 할 때 발생
- `Error : Can’t set headers after they are sent` : 요청에 대한 응답을 보낼 때 응답을 두 번 이상 보냄. 요청에 대한 응답은 한 번만 보내야 함
- `FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed- JavaScript heap out of memory` : 코드를 실행할 때 메모리가 부족해서 스크립트가 정상적으로 작동하지 않는 경우, 코드가 잘못 구현되었을 확률이 높으므로 코드를 점검해야 함
- `UnhandledPromiseRejectionWarning: Unhandled promise rejection` : 프로미스 사용 시 catch 메서드를 붙이지 않으면 발생, 항상 catch를 붙여 에러가 발생하는 상황에 대비
- `EADDRINUSE 포트 번호` : 해당 포트 번호에 이미 다른 프로세스가 연결되어 있음. 그 프로세스를 종료하거나 다른 포트 번호를 사용해야 함(그 프로세스는 노드 프로세스일 수도 있고 다른 프로그램일 수도 있음)
- `EACCES 또는 EPERM` : 노드가 작업을 수행하는 데 권한이 충분하지 않음
- `EJSONPARSE` : package.json 등의 JSON 파일에 문법 오류가 있을 때 발생
- `ECONNREFUSED` : 요청을 보냈으나 연결이 성립하지 않을 때 발생
- `ETARGET` : pakage.json에 기록한 패키지 버전이 존재하지 않을 때 발생
- `ETIMEOUT` : 요청을 보냈으나 응답이 시간 내에 오지 않을 때 발생
- `ENOENT: no such file or directory` : 지정한 폴더나 파일이 존재하지 않는 경우