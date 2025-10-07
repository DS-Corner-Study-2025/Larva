# 2.1 ES2015+

- 2015년 ES2015(ECMAScript 2015)가 등장
- ES2015 이상의 자바스크립트를 통틀어 ES2015+라고 함

## 2.1.1 const, let

- 보통 자바스크립트를 배울 때 var로 변수를 선언하는 것부터 시작하지만, 이제 const와 let이 대체

```jsx
if (true) {
		var x = 3;
}
console.log(x); // 3

if (true) {
		const y = 3;
}
console.log(y); // Uncaught ReferenceError: y is not defined
```

- var은 함수 스코프를 가지므로 if문의 블록과 관계없이 접근 가능
- const와 let은 **블록 스코프**를 가지므로 블록 밖에서는 변수에 접근할 수 없음
    - 블록의 범위 : if, while, for, function 등에서 볼 수 있는 중괄호 사이{}
    - 함수 스코프 대신 블록 스코프를 사용함으로써 호이스팅 같은 문제도 해결되고 코드 관리도 수월해짐
- const와 let은 스코프 종류가 다름
    - const : 한 번 값을 할당하면 다른 값을 할당할 수 없음, 초기화할 때 값을 할당하지 않으면 에러 발생
    
    ```jsx
    const a = 0;
    a = 1; // Uncaught TypeError: Assignment to constant variable
    
    let b = 0;
    b = 1; // 1
    
    const c; // Uncaught SyntaxError: Missing initializer in const declaration
    ```
    

## 2.1.2 템플릿 문자열

- ES2015 문법에 새로운 문자열이 생김
    - 큰따옴표나 작은따옴표로 감싸는 기존 문자열과 달리 백틱(`)으로 감쌈
    - 문자열 안에 변수를 넣을 수 있음
    
    ```jsx
    var num1 = 1;
    var num2 = 2;
    var result = 3;
    var string1 = num1 + ' 더하기 ' + num2 + '는 \'' + result + '\'';
    console.log(string1); // 1 더하기 2는 '3'
    ```
    
    - 문자열 string1은 띄어쓰기와 변수, 더하기 기호로 가독성이 좋지 않고 작은따옴표를 이스케이프(escape)해야 해서 코드가 지저분
    
    ```jsx
    const num3 = 1;
    const num4 = 2;
    const result2 = 3;
    const string2 = `${num3} 더하기 ${num4}는 ' ${result2}'`;
    console.log(string2); // 1더하기 2는 '3'
    ```
    
    - 기존 따옴표 대신 백틱을 사용하기 때문에 큰따옴표나 작음따옴표와 함께 사용할 수 있음

## 2.1.3 객체 리터럴

- oldObject 객체에 동적으로 속성을 추가

```jsx
var sayNode = function() {
		console.log('Node');
};
var es = 'ES';

var oldObject = {
		sayJS: fuction() {
				console.log('JS');
		},
		sayNode: sayNode,
};
oldObject[es + 6] = 'Fantastic';
oldObject.sayNode(); // Node
oldObject.sayJS(); // JS
console.log(oldObject.ES6); // Fantastic
```

- newObject 객체로 바꾸어 개선한 코드

```jsx
const sayNode = () = function() {
		console.log('Node');
};
const es = 'ES';

const newObject = {
		sayJS() {
				console.log('JS');
		},
		sayNode,
		[es + 6]: 'Fantastic',
};
newObject.sayNode(); // Node
newObject.sayJS(); // JS
console.log(newObject.ES6); // Fantastic
```

- `sayJS` 같은 객체의 메서드에 함수를 연결할 때 콜론(:)과 function을 붙이지 않아도 됨
- `sayNode: sayNode` 처럼 속성명과 변수명이 동일한 경우에는 한 번만 써도 됨 → 코드의 중복을 피할 수 있음
- 객체의 속성명을 만드려면 예전에는 객체 리터럴 바깥에서 해야 했지만, ES2015 문법에서는 객체 리터럴 안에 동적 속성을 선언할 수 있음

## 2.1.4 화살표 함수

```jsx
function add1(x, y) {
		return x + y;
}

const add2 = (x, y) => {
		return x + y;
}

const add3 = (x, y) => x + y;

const add4 = (x, y) => (x + y);

function not1(x) {
		return !x;
}

const not2 = x => !x;
```

- add1, add2, add3, add4와 not1, not2는 각각 같은 기능을 함
- 화살표 함수에서는 function 선언 대신 ‘⇒’ 기호로 함술르 서넝ㄴ
- 화살표 함수에서 내부에 return문밖에 없는 경우 return문을 줄일 수 있음
    - 중괄호 대신 return할 식을 바로 적음
    - 소괄호로 감쌀 수도 있음
- 매개변수가 한 개이면 매개변수를 소괄호로 묶어주지 않아도 됨

```jsx
var relationship1 = {
		name: 'zero',
		friends: ['nero', 'hero', 'xero'],
		logFriends: function () {
				var that = this; // relationship을 가리키는 this를 that에 저장
				this.friends.forEach(function (friend) {
						console.log(that.name, friend);
				});
		},
};
relationship1.logFriends();

const relationship2 = {
		name: 'zero',
		friends: ['nero', 'hero', 'xero'],
		logFriends() {
				this.friends.forEach(friend => {
						console.log(this.name, friend);
				});
		},
};
relationship2.logFriends();
```

- relationship1.logFriends() 안의 forEach문에서는 function 선언문을 사용
    - 각자 다른 함수 스코프의 this를 가지므로 that이라는 변수를 사용해서 relationship1에 간접적으로 접근
- relationship2.logFriends() 안의 forEach문에서는 화살표 함수를 사용
    - 바깥 스코프인 logFriends()의 this를 그대로 사용할 수 있음(상위 스코프의 this를 그대로 물려받는 것)

## 2.1.5 구조 분해 할당

- 구조 분해 할당을 사용하면 객체와 배열로부터 속성이나 요소를 쉽게 꺼낼 수 있음
- 객체의 속성을 같은 이름의 변수에 대입하는 코드

```jsx
var candyMachine = {
		status: {
				name: 'node',
				count: 5,
		},
		getCandy: function() {
				this.status.count--;
				return this.status.count;
		},
};
var getCandy = candyMachine.getCandy;
var count = candyMachine.status.count;
```

- 수정한 코드

```jsx
const candyMachine = {
		status: {
				name: 'node',
				count: 5,
		},
		getCandy() {
				this.status.count--;
				return this.status.count;
		},
};
const { getCandy, status: { count } } = candyMachine;
```

- candyMachine 객체 안의 속성을 찾아 변수와 매칭

- 배열에 대한 구조 분해 할당 문법

```jsx
var array = ['nodejs', {}, 10, true];
var node = array[0];
var obj = array[1];
var bool = array[3];

// 이렇게 바꿀 수 있음
const array = ['nodejs', {}, 10, true];
const [node, obj, , bool] = array;
```

- 위치를 맞춰줌
- 구조 분해 할당 문법도 코드 줄 수를 상당히 줄여주므로 유용

## 2.1.6 클래스

- 클래스 문법도 추가되었지만 여전히 프로토타입 기반으로 동작

```jsx
var Human = function(type) {
		this.type = type || 'human';
};

Human.isHuman = function(human) {
		return human instanceof Human;
}

Human.prototype.breathe = function() {
		alert('h-a-a-a-m');
};

var Zero = function(type, firstName, lastName) {
		Human.apply(this, arguments);
		this.firstName = firstName;
		this.lastName = lastName;
};

Zero.prototype = Object.create(Human.prototype);
Zero.prototype.constructor = Zero; // 상속하는 부분
Zero.prototype.sayName = function() {
		alert(this.firstName + ' ' + this.lastName);
};
var oldZero = new Zero('human', 'Zero', 'Cho');
Human.isHuman(oldZero); // true
```

- Human 생성자 함수가 있고 그 함수를 Zero 생성자 함수가 상속
- Zero 생성자 함수를 보면 상속받기 위한 코드가 상당히 난해

- 클래스 기반 코드로 수정

```jsx
class Human {
		constructor(type = 'human') {
				this.type = type;
		}
		
		static isHuman(human) {
				return human instanceof Human;
		}
		
		breathe() {
				alert('h-a-a-a-m');
		}
}

class Zero extends Human {
		constructor(type, firstName, lastName) {
				super(type);
				this.firstName = firstName;
				this.lastName = lastName;
		}
		
		sayName() {
				super.breathe();
				alert(`${this.firstName} ${this.lastName}`);
		}
}

const newZero = new Zero('human', 'Zero', 'Cho');
Human.isHuman(newZero); // true
```

- 전반적으로 class 안으로 그룹화됨
    - 생성자 함수는 constructor 안으로 들어감
    - `Human.isHuman`과 같은 클래스 함수는 static 키워드로 전환
    - 프로토타입 함수들도 모드 class 블록 안에 포함되어 어떤 함수가 어떤 클래스 소속인지 확인하기 쉬움
- 이렇게 클래스 문법으로 바뀌었더라도 자바스크립트는 프로토타입 기반으로 동작한다는 것을 명심!

## 2.1.7 프로미스

- 자바스크립트와 노드에서는 이벤트 리스너를 활용할 때 콜백 함수를 자주 사용
- ES2015부터는 자바스크립트와 노드의 API들이 콜백 대신 프로미스(Promise) 기반으로 재구성
- 프로미스 규칙

```jsx
const condition = true; // true이면 resolve, false이면 reject
const promise = new Promise((resolve, reject) => {
		if (condition) {
				resolve('성공');
		} else {
				reject('실패');
		}
});

// 다른 코드가 들어갈 수 있음
promise
		.then((message) => {
				console.log(message); // 성공(resolve)한 경우 실행
		})
		.catch((error) => {
				console.log(error); // 실패(reject)한 경우 실행
		})
		.finally(() => { // 끝나고 무조건 실행
				console.log('무조건');
		});
```

- 먼저 프로미스 객체를 생성
    - new Promise로 프로미스를 생성할 수 있음
    - 안에 resolve와 reject를 매개 변수로 갖는 콜백 함수를 넣음
- promise 변수에 then과 catch 메서드를 붙일 수 있음
    - 프로미스 내부에서 resolve가 호출되면 then이 실행되고, reject가 호출되면 catch가 실행됨
    - finally 부분은 성공/실패 여부와 상관없이 실행됨
- resolve와 reject에 넣어준 인수는 각각 then과 catch의 매개변수에서 받음
    - `resolve('성공')`이 호출되면 then의 message가 ‘성공’이 되고, `reject('실패')`가 호출되면 catch의 error가 ‘실패’가 됨
- 프로미스는 실행은 바로 하되 결과값은 나중에 받는 객체
    - 결과값은 실행이 완료된 후 then이나 catch 메서드를 통해 받음
    - new Promise는 바로 실행되지만, 결과값은 then을 붙였을 때 받음

- then이나 catch에서 다시 다른 then이나 catch를 붙일 수 있음
    - 이전 then의 return 값을 다음 then의 매개변수로 넘김
    - 프로미스를 return한 경우 프로미스가 수행된 후 다음 then이나 catch가 호출됨

```jsx
promise
		.then((message) => {
				return new Promise((resolve, reject) => {
						resolve(message);
				});
		})
		.then((message2) => {
				console.log(message2);
				return new Promise((resolve, reject) => {
						resolve(message2);
				});
		})
		.then((message3) => {
				console.log(messsage3);
		})
		
		.catch((error) => {
				console.error(error);
		});
```

- 처음 then에서 message를 resolve하면 다음 then에서 message2로 받을 수 있음
- 다시 message2를 resolve한 것을 다음 then에서 message3으로 받음
    - 단, then에서 new Promise를 return해야 다음 then에서 받을 수 있음

- 콜백을 프로미스로 바꿀 수 있음

```jsx
function findAndSaveUser(Users) {
		Users.findOne({}, (err, user) => { // 첫 번째 콜백
				if (err) {
						return console.error(err);
				}
				user.name = 'zero';
				user.save((err) => { // 두 번째 콜백
						if (err) {
								return console.error(err);
						}
						Users.findOne({ gender: 'm' }, (err, user) => { // 세 번째 콜백
						// 생략
						});
				});
		});
}
```

## 2.1.8 async/await

```jsx
async function findAndSaveUser(Users) {
		let user = await Users.findOne({});
		user.name = 'zero';
		user = await user.save();
		user = await Users.findOne({ gender: 'm' });
		// 생략
}
```

- 함수 선언부를 일반 함수 대신 async function으로 교체한 후, 프로미스 앞에 await을 붙임 → 해당 프로미스가 resolve될 때까지 기다린 뒤 다음 로직으로 넘어감
    - ex. `await Users.findOne({})`이 resolve될 때까지 기다린 다음에 user 변수를 초기화하는 것

## 2.1.9 Map/Set

- Map은 객체와 유사하고 Set은 배열과 유사
- Map은 속성들 간의 순서를 보장하고 반복문을 사용할 수 있음
    - 속성명으로 문자열이 아닌 값도 사용할 수 있고 size 메서드를 통해 속성의 수를 쉽게 알 수 있다는 점에서 일반 객체와 다름

```jsx
const m = new Map();
m.set('a', 'b');
m.set(3, 'c');
const d = {};
m.set(d, 'e');

m.get(d);
console.log(m.get(d));

m.size;
console.log(m.size);

for (const [k, v] of m) {
		console.log(k, v);
}

m.forEach((v, k) => {
		console.log(k, v);
});

m.has(d);
console.log(m.has(d)); // true

m.delete(d);
m.clear();
console.log(m.size); // 0
```

- Set은 중복을 허용하지 않는다는 것이 가장 큰 특징
    - 배열 구조를 사용하고 싶으나 중복을 허용하고 싶지 않을 때 Set을 대신 사용
- 기존 배열에서 중복을 제거하고 싶을 때도 Set을 사용
    - new Set(배열)을 하는 순간 배열의 중복된 요소들이 제거됨
    - Set을 배열로 되돌리려면 Array.from(Set)을 하면 됨

```jsx
const s = new Set();
s.add(false); // add(요소)로 Set에 추가
s.add(1);
s.add('1');
s.add(1); // 중복이므로 무시됨
s.add(2);

console.log(s.size); // 중복이 제거되어 4

s.has(1); // has(요소)로 요소 존재 여부 확인
console.log(s.has(1)); // true

for (const a of s) {
		console.log(a); // false 1 '1' 2
}

s.forEach((a) => {
		console.log(a); // false 1 '1' 2
})

s.delete(2); // delete(요소)로 요소를 제거
s.clear(); // clear()로 전부 제거

// 기존 배열에서 중복 제거
const arr = [1, 3, 2, 7, 2, 6, 3, 5];

const s = new Set(arr);
const result = Array.from(s);
console.log(result); // 1, 3, 2, 7, , 5
```

## 2.1.10 널 병합/옵셔널 체이닝

- `??` : 널 병합(nullish coalescing) 연산자
    - 주로 || 연산자 대용으로 사용됨
    - falsy 값(0, ‘’, false, NaN, null, undefined) 중 null과 undefined만 따로 구분

```jsx
const a = 0;
const b = a || 3; // || 연산자는 falsy 값이면 뒤로 넘어감
console.log(b); // 3

const c = 0;
const d = c ?? 3; // ?? 연산자는 null과 undefined일 때만 뒤로 넘어감
console.log(d); // 0;

const e = null;
const f = e ?? 3;
console.log(f); // 3

const g = undefined;
const h = g ?? 3;
console.log(h); // 3
```

- `?.` : 옵셔널 체이닝(optional chaining) 연산자
    - null이나 undefined의 속성을 조회하는 경우 에러가 발생하는 것을 막음
    - 일반적인 속성뿐만 아니라 함수 호출이나 배열 요소 접근에 대해서도 에러가 발생하는 것을 방지할 수 있음
    - 자바스크립트 프로그래밍을 할 때 발생하는 TypeError: Can not read properties of undefined 또는 null 에러의 발생 빈도를 획기적으로 낮출 수 있음

```jsx
const a = {};
a.b; // a가 객체이므로 문제없음

const c = null;
try {
		c. d;
} catch (e) {
		console.error(e); // TypeError: Cannot read properties of null (reading 'd')
}
c?.d; // 문제없음

try {
		c.f();
} catch (e) {
		console.error(e); // TypeError: Cannot read properties of null (reading 'f')
}
c?.f(); // 문제없음

try {
		c[0];
} catch (e) {
		console.error(e); // TypeError: Cannot read properties of null (reading '0')
}
c?.[0]; // 문제없음
```

# 2.2 프런트엔드 자바스크립트

- HTML에서 script 태그 안에 작성하는 부분

## 2.2.1 AJAX

- 비동기적 웹서비스를 개발할 때 사용하는 기법
- 요즘에는 JSON을 많이 사용
- 페이지 이동 없이 서버에 요청을 보내고 응답을 받는 기술
- 보통 AJAX 요청은 jQuery나 axios 같은 라이브러리를 이용해서 보냄

- GET 요청 보내기
    - axios.get 함수의 인수로 요청을 보낼 주소를 넣기
    - axios.get도 내부에 new Promise가 들어 있으므로 then과 catch를 사용할 수 있음

- POST 방식의 요청 보내기
    - 데이터를 서버로 보낼 수 있음
    - axios.post를 사용

## 2.2.2 FormData

- HTML form 태그의 데이터를 동적으로 제어할 수 있는 기능
- FormData 생성자로 객체를 만듦
    - 생성된 객체의 append 메서드로 키-값 형식의 데이터를 저장할 수 있음
    - append 메서드를 여러 번 사용해서 키 하나에 여러 개의 값을 추가해도 됨
    - has 메서드는 주어진 키에 해당하는 값이 있는지 여부를 알림
    - get 메서드는 주어진 키에 해당하는 값 하나를 가져옴
    - getAll 메서드는 주어진 키에 해당하는 모든 값을 가져옴
    - delete는 현재 키를 제거하는 메서드이고, set은 현재 키를 수정하는 메서드
- axios로 폼 데이터를 서버에 보냄

## 2.2.3 encodeURIComponent, decodeURIComponent

- AJAX 요청을 보낼 때 서버가 한글 주소를 이용하지 못하는 경우에 사용하는 window 객체의 메서드
- 브라우저뿐만 아니라 노드에서도 사용할 수 있음
- 보낼 때 encodeURIComponent, 받는 쪽에서decodeURIComponent를 사용

 

## 2.2.4 데이터 속성과 dataset

- 프론트엔드에 데이터를 내려보낼 때 첫 번째로  고려해야 할 점은 보안
    - 프론트엔드에 민감한 데이터를 내려보내면 안 됨(비밀번호 등)
- 데이터 속성 : 프론트엔드에 데이터를 내려보낼 때 HTML과 관련된 데이터를 저장하는 공식적인 방법
    - HTML 태그의 속성으로, `data-`로 시작하는 것들을 넣음
    - 이 데이터들을 이용해 서버에 요청을 보내게 됨
    - 장점 : 자바스크립트로 쉽게 접근할 수 있음
    - dataset에 데이터를 넣어도 HTML 태그에 반영됨