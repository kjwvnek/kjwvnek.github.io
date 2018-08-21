# 객체지향 프로그래밍
이전 글에 이어서 객체지향 프로그래밍의 4가지 특징 중 Inheritance(상속) / Polymorphism(다형성)을 다뤄보자.

## Inheritance(상속)
흔히 상속한다는 말은 부모가 자식에게 무언가를 물려주는 것을 이야기한다.  

이번에도 예를 먼저 들어보자. 우리가 자동차 딜러회사의 프로그램을 개발한다고 생각해보자.  
이 자동차 회사는 현대자동차, BMW, 벤츠 이 세가지 차량을 판다고 했을 때 추상화를 먼저 진행해보자.  
가장 먼저 누구나 '자동차' 라는 것을 추상화할 것이다. 그리고 '현대자동차', 'BMW', '벤츠' 이 세가지 개념도 마찬가지로 추상화할 수 있을 것이다.
  
그럼 '자동차' 와 '나머지 세가지'는 동등한 Depth 의 추상화라고 볼 수 있을까?  
아니다. 분명히 '자동차'는 '나머지 세가지'보다 더 공통적인 영역, 즉 부모의 개념으로 볼 수 있을 것이다.  
객체지향에서 이런 부모/자식 관계를 확인할 때 흔히 'IS-A' 관계를 활용한다.  
'현대자동차' is a '자동차' 이것은 말이 되지만 '자동차' is a '현대자동차' 이건 말이 되지 않는다.  
즉 이 관계를 통해 '현대자동차'가 '자동차'의 자식이 된다고 생각할 수 있다.  

따라서 자식은 당연히 부모의 추상화한 특징을 그대로 이어받아야한다. '현대자동차'는 '자동차'이기도 하기때문이다.  
이런 특징을 코드로 구현하면 어떻게 구현할 수 있을까?

### 자바스크립트 프로토타입을 이용한 상속의 구현
자바스크립트 언어의 특징을 조금 더 살펴보기 위해 class 키워드를 사용하지 말아보자.  
또한 프로토타입 기반 언어(자바스크립트)에서 상속이 클래스기반의 언어(자바)와 어떻게 다른지 생각해보자.  
~~~javascript
function Car(engine, fuelEfficiency) {
  if (!(this instanceof Car)) {
    throw new Error("it's constructor function");
  }
  
  var instance = this;
  instance.engine = engine;
  instance.fuelEfficiency = fuelEfficiency;
  return instance;
}
Car.prototype.move = function() {
  this.engine -= this.fuelEfficiency;
}

function HyundaiCar() {
  if (!(this instanceof HyundaiCar)) {
    throw new Error("it's constructor function");
  }
  
  var instance = Car.call(this, arguments);
  instance.navigation = 'Hyundai Navigation AI Object';
  return instance;
}
HyundaiCar.prototype = Object.create(Car.prototype);
HyundaiCar.prototype.constructor = HyundaiCar;
HyundaiCar.prototype.getRoute = function() {
  return this.navigation;
}
~~~
위의 예제를 살펴보면 `HyundaiCar` 는 `Car` 의 자식으로써 `engine` 과 `fuelEfficiency`, `move()` 를 상속받았다.  
이런식으로 더 상위 추상화 대상을 하위 추상화 대상이 상속받음으로써, 객체지향의 기본 중 하나인 "공통점으로 묶되, 불필요한 공통점을 줄여라"를 달성할 수 있다.
    
위에서 우리는 `HyundaiCar` 생성자함수 안에서 `Car`의 생성자함수를 먼저 호출함으로써 Car 의 속성들을 가져왔고  
**프로토타입 체이닝**을 통한 **Behavior Delegation(행동 위임)** 을 통해서 메서드들을 사용할 수 있게 만들었다.

### 자바 클래스를 이용한 상속의 구현
이쯤에서 클래스 상속의 개념을 지원하는 자바에서 구현했을 때는 어떻게 다를까? 를 확인해보자.  
~~~java
public Class Car {
  // 멤버변수
  private int engine;
  private int fuelEfficiency;
  
  // 생성자함수
  public Car(int engine, int fuelEfficiency) {
    this.engine = engine;
    this.fuelEfficiency = fuelEfficiency;
  }
  
  // 메소드
  public void move() {
    engine -= fuelEfficiency;
  }
}

public Class HyundaiCar extends Car {
  // 멤버변수
  private HyundaiCarNavigation navigation;
  
  // 생성자함수
  public HyundaiCar() {
    this.navigation = new HyundaiCarNavigation();
  }
  
  // 메소드
  public Route getRoute() {
    return navigation.getRoute();
  }
}
~~~
Java 에서 클래스는 어떤 객체의 모양을 미리 정해둔 '설계도'와 같다. 또한 객체는 이런 설계도를 통해서만 만들어진다.  
Java 언어는 정적 컴파일 언어로, 이런 클래스(객체의 설계도)가 미리 정해져있으며 코드가 실행되며 클래스가 수정될 수 없다.  
그렇기에 안전한 설계가 가능하다는 장점이 있고, 이런 장점은 규모가 큰 어플리케이션일 수록 빛을 발할 것이다.  

반면 Javascript 는 동적 언어로, 프로그램을 실행할 때, 아무것도 정해져있지 않다.  
모든 객체는 동일한 객체`{}`를 기반으로 클론하여 사용하며, '클래스 상속'을 대신해서 '프로토타입 체이닝을 통한 행동위임(Behavior Delegation)'이라는 개념이 존재한다.  
> 왜 프로토타입 체이닝은 상속보다는 위임이 더 맞는 표현일까?  
객체지향 프로그래밍에서 위임이라는 단어는 객체가 자신에게 의존하는 객체에게 책임의 일부를 수행하도록 하는 것을 말한다.  
즉, 프로토타입 체이닝의 원리를 살펴보면 자식 객체는 부모 객체의 책임을 위임 받아 수행하는 것이기 때문에 상속보다 Behavior Delegation 이라는 표현이 더 맞는 표현이다.

개인적으로 두가지 기법 '클래스 상속'과 '프로토타입 체이닝'은 '객체지향의 추상화의 상속'이라는 동일한 개념을 추구한다는 점에서 일맥상통하지만 구현원리가 조금 다르다고 생각한다.  
무엇이 더 좋다고 딱 잘라서 말하기는 어려울듯 하다. 하지만 그 장단점을 분명이 이해하고 설계하는 것이 개발자로서 가져야할 부분이라 생각한다.  
그렇기에 자바스크립트에서 `class` 키워드를 사용하더라도 반드시 이를 이해하고 사용해야 할 것이다.

### 자바스크립트에서 상속이란
자바스크립트에서 프로토타입 체이닝을 이용해야지만 상속이라고 말할 수 있을까?  
**그렇지 않다**고 생각한다. 그동안 객체지향언어로 자바언어를 오래 사용해온 우리는 '추상화 = 클래스 = 생성자함수 + 멤버변수 + 메소드'가 공식처럼 우리 머릿속에 들어온 것 같다.  
자바언어에서 객체는 클래스를 `new` 키워드를 통해 호출할때만 생성되고 생성된 객체는 클래스에 정의된 멤버변수만 가질수 있다.  
하지만 자바스크립트 언어의 객체`{}`는 동적으로 동작하며 모든 참조형은 이 객체를 기반으로 하는 특징을 가진다.

위의 예제를 만약 이런식으로 구현했다면 이건 상속이 아닐까?
~~~javascript
var car = {
  engine: null,
  fuelEfficiency: null,
  move: function() {
    this.engine -= this.fuelEfficiency;
  }
};

var hyundaiCar = {
  navigation: {
    getRoute: function() {}
  },
  getRoute: function() {
    this.navigation.getRoute();
  }
};

hyundaiCar = _.extend(hyundaiCar, car);
~~~
shallow copy 를 통해 분명히 '현대자동차'는 '자동차'의 특성을 모두 물려받았다.  
이 또한 상속이라고 생각한다. 훨씬 간단한 방식의 상속이지만, 단점은 동일한 객체를 여러개 클론할 경우 프로토타입 체이닝을 활용하는 것보다 메모리를 더 많이 잡아 먹을 것이다.  
또한 누가 누구에게 무엇을 상속받았는지 알기도 어렵다. 이런 특징때문에 자바스크립트에서는 **덕 타이핑**을 많이 활용한다.  
어떤 방식을 사용할지는 상황에 맞게 결정해야한다.

## Polymorphism(다형성)
다형성은 우리가 추상화하는 대상이 단 한가지만으로 추상화 되지 않는다는 것이다. 어떤 대상은 여러개의 책임을 가질 수 있기때문에 여러개의 추상화가 가능하다는 것이다.

### 데이터 추상화 관점에서의 다형성  
이것도 예제를 살펴보자. '나'를 추상화 해보자. 나는 '프로그래머'로 추상화할 수 있을 것이다. 또 나는 '산업공학과 학생'으로 추상화가 가능할 것이다.  
이처럼 나라는 대상은 꼭 '나'로만 추상화되지는 않을 것이다.  
마찬가지로 우리가 위에서 살펴봤던 상속도 다형성의 하나이다. '현대자동차'는 '현대자동차' 뿐만 아니라 '자동차'라는 추상화 또한 가능하다.  
자바스크립트를 이용해서 간단히 구현해보자.
~~~javascript
function Programmer() {
  var instance = this;
  instance.languages = {};
  return instance;
}
Programmer.prototype.develop = function(language) {
  this.languages[language]();
}

function IndustrialEngineeringStudent(grade) {
  var instance = this;
  instance.grade = grade;
  return instance;
}
IndustrialEngineeringStudent.prototype.dataMining = function(technique) {
  if (this.grade >= 4) {
    dataMiningTechniques[technique]();
  }
}

function Me(name, grade) {
  var instance = Programmer.call(this);
  instance = IndustrialEngineeringStudent.call(instance, grade);
  instance.name = name;
  return instance;
}
Me.prototype = Object.create(_.extend({}, Programmer.prototype, IndustrialEngineeringStudent.prototype));
Me.prototype.constructor = Me;
~~~
위의 예제는 다형성에 따라서 기능은 얼핏 잘 구현된 것 처럼 보이지만 한가지 부족한 점이 있다.  
바로 `Me` 로 생성된 객체가 `instanceof` 연산자를 통해 `Programmer` 와 `IndustrialEngineeringStudent` 둘 모두를 자신의 생성자함수라고 인식할 수 없는 것이다.   
자바스크립트 프로토타입 체이닝과 자바 클래스 모두 **다중상속을 지원하지 않기 때문**이다.  
하지만 자바스크립트는 위에서 했던 것처럼 리터럴 객체를 이용해 또 손쉽게 다중상속을 구현할 수 있을 것이다.

### 프로세스 추상화 관점에서의 다형성
그리고 이런 데이터 추상화(생성자함수)의 관점에서만 다형성이 성립하는것이 아니라 프로세스 추상화(메소드)의 관점에서도 다형성은 성립될 수 있다.  
만약 이런식으로 설계한다면 어떨까?
~~~javascript
var animal = {
  say: function() {
    if (this === lion) {
      console.log("i'm lion~~");
    } else if (this === tiger) {
      console.log("i'm tiger~~");
    } else {
      console.log("i'm animal~~");
    }
  }
}

var lion = {}
lion = _.extend(lion, animal);

var tiger = {};
tiger = _.extend(tiger, animal);
~~~
`say` 라는 메서드는 '말한다' 라는 하나의 책임을 추상화하지만 사자한테서 쓰일 때와 호랑이한테서 쓰일 때 각각 다양한 형태를 갖고있다.  
이처럼 동일한 메서드가 여러개로 분기될 때, `Method Overloading`이라고 부르고 이것도 다형성이라고 볼 수 있다.
상속받은 메서드를 재정의하는 `Method Overriding`도 같은 원리로 다형성이다. 예제는 다음과 같다.
~~~javascript
var animal = {
  say: function() {
    console.log("i'm animal");
  }
}
var lion = {}
lion = _.extend(lion, animal);
lion.say = function() {
  console.log("i'm lion");
}

var tiger = {}
tiger = _.extend(tiger, animal);
tiger.say = function() {
  console.log("i'm tiger");
}
~~~ 

### 추상클래스와 인터페이스
자바언어와 같은 클래스 기반 언어에서는 다형성을 좀 더 잘 구현하기 위해 추상클래스와 인터페이스라는 키워드가 또 존재한다.  
이는 TypeScript 에서도 있는 키워드이기 때문에 가볍게 짚고 넘어가보자.

#### 추상클래스
추상클래스란 간단히 말해서 추상화한 대상을 클래스(또는 생성자함수)로 만드는 것이 아니라, 추상화한 그대로 두고 싶은 것이다.  
위의 상속의 예제에 대입해 생각해볼 때, '자동차'와 '현대자동차' 모두 추상화해서 설계하고 싶지만, '자동차'는 구체화가 가능한 클래스로 만들고 싶지 않을 수 있다.  
즉, '자동차'를 `new` 키워드로 직접 인스턴스를 생성할 수는 없지만, 추상화 시킴으로써 좀 더 안전한 설계가 가능하도록 하는 것이다.  

자바스크립트 언어 자체에는 없는 키워드이기 때문에 온전히 구현되지는 않지만, 억지로 구현한다면 이런 느낌이다. 
~~~javascript
var car = {
  engine: null,
  fuelEfficiency: null,
  move: function() {
    throw new Error("it's abstract method! implement this");
  }
}

var hyundaiCar = {
  navigation: {
    getRoute: function() {}
  },
  getRoute: function() {
    this.navigation.getRoute();
  }
}

hyundaiCar = _.extend(hyundaiCar, car);
hyundaiCar.move(); // Error
~~~
'현대자동차'는 '자동차'를 상속받았기 때문에 move 라는 메서드를 가져야한다.  
하지만 나는 '자동차'는 'move'해야한다고 추상화만 해두었지 그 메서드가 정확히 어떤 동작을 하는지는 정하지 않았기 때문에,  
'현대자동차'는 자기에 맞는 'move' 동작을 구현해줘야한다.

#### 인터페이스
인터페이스는 어떤 추상화를 상속이 아닌 구현(HAS-A 관계)을 하고 싶을 때, 사용하는 **단순한 서술**이다. (단지 코드를 봤을 때는, 추상메서드만 모아둔 추상클래스라고도 할 수 있을 것이다)  
예를 들면 프린터도 팔고, 스캐너도 팔고, 복합기도 파는 회사가 있다고 상상하자. 우린 이 회사의 관리 프로그램을 만들어줘야한다.  

우리는 프린터에 필요한 기능을 서술한 '프린터 인터페이스',  
스캐너에 필요한 기능을 서술한 '스캐너 인터페이스'를 미리 설계하고,

프린터 추상화(클래스나 생성자함수)는 프린터 인터페이스를 구현하고,  
스캐너 추상화는 스캐너 인터페이스를 구현하고,  
복합기 추상화는 프린터 인터페이스와 스캐너 인터페이스를 구현한다.  

프린터,스캐너와 복합기는 분명 상속의 관계가 아니지만 추상화간의 어떤 관계가 존재한다. 바로 HAS-A 관계다.  
프린터 has a '프린터 기능'  
스캐너 has a '스캐너 기능'  
복합기 has a '프린터 기능', '스캐너 기능'

이런식으로 설계함으로써 다형성을 구현하는 것 외에 우리가 또 얻을 수 있는 것이 무엇일까?  

1. 책임의 분할  
세번째 글에서 다루겠지만, 객체지향은 기본적으로 단일책임 원칙을 지향한다. 중복성을 낮추고, 확장성있는 설계를 하기 위해서다.  
따라서 이런식의 책임의 분할은 분명한 이점이라 할 수 있다.
2. 협업도/독립성  
개발 시작 이전에 인터페이스를 미리 설계해둠으로써 특정 기능이 아직 구현이 안됬더라도 의존관계와 상관없이 독립적으로 다른 기능을 먼저 구현할 수 있다. 즉, 의존성을 느슨하게 만드는 효과를 가진다.

## 생각하기
지금까지 객체지향 프로그래밍의 4가지 특징을 자바스크립트의 관점에서도 자바의 관점에서도 살펴보았다.  
여러 기능을 살펴보면 분명히 자바는 좀 더 안전하고 분명한 객체지향 설계를 위해 발전한 언어라는 느낌이 든다.
  
하지만 자바스크립트 언어는 자바언어와 다른 방향으로 객체지향을 추구하면서 조금 더 위험성은 존재할 수 있으나, 다양하고 창의적인 디자인패턴이 가능하도록 설계되었다고 생각한다.  
(이 위험성도 타입스크립트나 'use strict'를 통해 감소시킬 수 있다.)  
물론 객체지향의 관점이 아닌 다른 부분에서도 많은 차이가 있는 언어이다.  
객체지향은 프로그램을 개발하는 하나의 방법론일 뿐이지 절대적인 원칙이 아니라는 것을 생각하고 장점을 위주로 활용하는 개발을 해야겠다.  
