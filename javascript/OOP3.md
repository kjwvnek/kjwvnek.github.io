# 객체지향 프로그래밍
이번에는 객체지향 프로그래밍을 할 때, 조금 더 '잘' 하기위해서 로버트마틴이 세운 다섯가지 객체지향 설계원칙을 살펴볼 것이다.  
마이클 페더스가 머릿글자를 따서 SOLID 라 명명했다.  

## 용어
이 SOLID 를 살펴보기전에 책임(Responsibility)과 의존(Dependency)과 위임(Delegation)이라는 용어를 살펴보자.  
**책임**은 모듈의 관점에서보면 쉽게 기능이라고 말할 수 있을 것이다. 일반적으로 어떤 모듈은 어떤 기능을 책임지고있다.
**의존**은 어떤 모듈(코드 조각)이 내부에서 다른 모듈을 사용할 때 생기게 된다. 예를 들어서 확인해보자.  
~~~javascript
function LinkedList(value) {
  this.head = new Node(value);
  this.tail = this.head;
}
function Node(value) {
  this.value = value;
  this.next = null;
}
~~~
링크드리스트 자료구조를 구현하다 만 코드이다. 여기서 `LinkedList`는 자기가 해야할 일을 `Node`에게 **위임**했다.  
**위임을 진행**하는 `LinkedList`가 상위수준의 모듈, **위임을 받는** `Node`가 하위수준의 모듈이라 할 수 있을 것이다.
즉, `LinkedList`가 `Node`에게 **의존**하게 되는 것이다.  

## SOLID
- Single Responsibility Principle (단일책임 원칙)  
모든 클래스/함수는 반드시 한가지의 책임(목적)을 가져야한다.
- Open / Closed Principle (개방/폐쇄 원칙)  
모든 소프트웨어 개체는 확장 가능성은 열려있되, 수정 가능성은 닫혀있어야한다.
- Liskov Substitution Principle (리스코프 치환 원칙)  
어떤 타입에서 파생된 타입의 객체가 있을 때, 어떤 타입(파생시킨 타입)의 코드는 변경하지 말아야한다.
- Interface Segregation Principle (인터페이스 분리 원칙)
인터페이스는 아주 작은 추상화 조각 하나만을 바라봐야한다.
- Dependency Inversion Principle (의존성 역전 원칙)  
상위수준 모듈은 하위수준 모듈에 의존해서는 안되며 이 둘은 추상화에 의존해야한다. 또한 추상화된 것은 구체화된 것에 의존해서는 안된다. 구체화된 것이 추상화된 것에 의존해야한다.

### Single Responsibility Principle (단일책임 원칙)
단일책임원칙은 추상화를 진행할 때 단 하나의 책임을 갖도록 추상화해야 한다는 것이다.  
물론 이 부분은 충분히 주관적일 수 있는 영역이라고 생각하지만 원칙적으로 접근하겠다.
~~~javascript
function addCategoryElement() {
  $.get('url.bla.bla', function(categoryData) { // 코드가 길어져서 jQuery Get을 썼지만, XMLHttpRequest 를 활용했다고 생각해보자.
    var categoryElement = document.createElement('div');
    categoryElement.innerHTML = categoryData;
    document.body.appendChild(categoryElement);
  });
}

addCategoryElement();
~~~
위와 같이 "카테고리 데이터를 서버에서 받아서 카테고리 요소를 만들어 body 에 붙여주는 함수"를 추상화했다고 하자.  
어떤 개발자는 저 함수가 하나의 책임을 가진다고 생각할 수도 있지만    
좀 더 단일책임 원칙이 지켜지도록 이런식으로 설계할 수 있지 않을까 생각한다.
~~~javascript
function requestGet(url) {
  return (new Promise(function(resolve) {
    $.get(url, function(data) {
      resolve(data);
    });
  }))
}

function addCategoryElement(categoryData) {
  var categoryElement = document.createElement('div');
  categoryElement.innerHTML = categoryData;
  return categoryElement;
}

requestGet('url').then(addCategoryElement);
~~~

### Open / Closed Principle (개방/폐쇄 원칙)
수정 가능성은 닫되 확장 가능성은 열어야 한다.  
아주 좋은 말이지만 이 말을 어떻게 실천할 수 있을까?  
적절한 의존관계를 설계하고 단일책임의 원칙을 잘 지킴으로써 가능해보인다.  

예를들어 데이터1,2,3 을 가져와서 화면에 띄워주는 기능을 가진 앱이 있다고 생각해보자.
~~~javascript
function getData1() {}
function getData2() {}
function getData3() {}

function app() {
  var data1 = getData1();
  var data2 = getData2();
  var data3 = getData3();
  
  display(data1);
  display(data2);
  display(data3);
  
  // ... 다른 수많은 기능들
  
  function display(data) {}
}
~~~
위와 예제에서 `app`은 `getData1`, `getData2`, `getData3`에 의존하게된다.  
`app`에는 이 기능 뿐 아니라 수많은 기능이 있을텐데 `getData` 함수의 수정가능성이 수많은 수정가능성으로 엮일 가능성이 생긴다.  
따라서 이런식으로 의존관계를 분리하면 수정가능성을 닫고, 확장 가능성을 열 수 있을 것이다.
~~~javascript
var getData = {
  1: function() {},
  2: function() {},
  3: function() {}
};

function display(type) {
  var data = getData[type]();
  // ...
}

function app() {
  display(1);
  display(2);
  display(3);
}
~~~
`app`은 `display`에만 의존하게됨으로써 수정가능성을 축소시켰다. 

### Liskov Substitution Principle (리스코프 치환 원칙)
리스코프 치환의 법칙을 자바스크립트 언어의 관점에서 생각했을 때 이렇게 볼 수 있을 듯하다.  
프로토타입 체이닝을 통해 'Behavior Delegation'을 할 때, 위임받은 메서드를 직접 변경하는 일이 있어서는 안되고 `Overriding`을 하더라도 일관성이 있어야하는 것이다.  
즉, 철저하게 확장의 개념으로써 일관성을 세워야한다.  

아래의 예제를 보자.
~~~javascript
function Person() {}
Person.prototype.hello = function() {
  console.log('hello');
}

function Man() {}
Man.prototype = Object.create(Person);
Man.prototype.constructor = Man;
Man.prototype.hello = function() {
  console.log('Fu**!');
}
Man.prototype.intro = function() {
  console.log("i'm man");
}

function Woman() {}
Woman.prototype = Object.create(Person);
Woman.prototype.constructor = Woman;
Woman.prototype.intro = function() {
  console.log("i'm woman");
}
~~~
이런식으로 설계를 했을 때, `Man.hello()`를 사용할 때 일관성이 사라짐으로써 잉? 할수가 있다.  
우리가 다형성의 개념을 구현하고자 `Overriding`을 할 수는 있을 것이지만 그 메서드의 책임이 달라서는 안된다는 것이다.
 
### Interface Segregation Principle (인터페이스 분리 원칙)
이 원칙은 C++, 자바와 같은 인터페이스 기반의 언어 환경에 맞춰진 원칙으로  
인터페이스에 단일책임 원칙을 적용한 것과 같다.  
인터페이스는 가장 작은 단위로 분리시켜야하고, 클래스가 불필요한 인터페이스를 구현해서는 안된다는 것이다.  

### Dependency Inversion Principle (의존성 역전 원칙)
상위수준 모듈은 하위수준 모듈에 의존해서는 안되며 이 둘은 추상화에 의존해야한다.  
이벤트 회사의 프로그램을 만든다고 생각하자. 이벤트의 방식에 따른 결과를 보여주는 프로그램이 있다. 
~~~javascript
var app = {
  displayEvent: function() {
    body.innerHTML = flowerEvent.display();    
  }
};
var flowerEvent = {
  materials: [],
  display: function() {
    return this.materials.join(',');
  }
};
~~~
이 예제는 `displayEvent`라는 상위모듈이 `createEvent`라는 하위모듈에 의존하고 있는 것을 볼 수 있다.  
하지만 이건 이벤트회사라고 했다. `createEvent` 함수의 수정이 잦을 것이라는 것을 충분히 예측할 수 있다.  
그렇다면 `displayEvent` 상위모듈은 잦은 수정가능성 영향을 받게될 것이다. 이처럼 하위모듈이 상위모듈을 흔드는 것을 **의존성 역전**이라고한다.  

이런 의존성 역전은 인터페이스를 활용함으로써 어느정도 방지할 수 있다.
아래처럼 모듈 그 자체가아닌, 모듈의 추상화에 의존해서 설계함으로써 방지할 수 있다는 뜻이다.
(자바 문법이 잘 기억이 안나서 문법자체는 부정확할수 있지만 이런 느낌이라는 것만 알아두자.)   
~~~Java
public Class App {
  public void displayEvent() {
    // Event 인터페이스의 display 메소드를 씀으로써 FlowerEvent에 대한 의존에서 벗어났다.
    Event event = new FlowerEvent();
    sysout(event.display());
  }
}

public Interface Event {
  public abstract String display;
}

public Class FlowerEvent implements Event {
  private List materials(void);
  
  public FlowerEvent(List materials) {
    this.materials = materials;
  }
  
  // 구현이 강제된다.
  public String display() {
    return materials.toString();
  }
}
~~~
아쉽게도 자바스크립트에서는 인터페이스가 없기 때문에 타입스크립트를 사용할 때 고민해봐야할듯 하다.

## 생각하기
공부하고 생각할수록 객체지향은 추구하는 것이지 정답이 있는건 아니라고 생각한다.  
무조건 분리하고 단일화시키고 인터페이스를 만드는 것은 불필요한 복잡성을 증가시키는 것일 수도 있다.  
이 공부하고 이해한 것들을 베이스로 적절한 선에서 추구하도록 다양한 프로젝트를 해봐야겠다.   