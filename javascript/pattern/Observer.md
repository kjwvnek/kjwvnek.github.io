# Observer Pattern
이 패턴을 보기 전에 Publish-Subscribe 패턴을 먼저 보는 것을 추천한다. (PubSub.md)  
그리고 Observer 패턴과 어떤 차이점이 있는지 비교해보자.  

## 요약
옵저버(Observer) 패턴은 뜻 그대로 무언가를 지속적으로 관찰하기 위한 패턴이다.  
이 옵저버 패턴의 구성요소는 어떤게 있을까? 우선 관찰자를 관리하며 메세지를 전달하는 `Subject(주체)`와 관찰을 하는 `Observer(관찰자)`가 있을 것이다.  
그리고 `Observer`는 자기가 구독한 `Subject`의 `State(상태)`변화에 따라서 `Message(메세지)`를 받을 것이다.
  
여기까지 봤을 때 `Topic-based PubSub` 패턴과 유사해보이지만 조금 다른 것을 알 수 있다.  
**PubSub 패턴**의 핵심은 `Loose Coupling`이었다. 공급자와 구독자가 서로를 모름으로써 의존성을 지웠고 이는 장점이되기도 단점이 되기도 했다.

하지만 **Observer 패턴**은 단순히 구독한 채널의 메세지를 전달받는 것이 아니라 해당 주체(설계하기에 따라 여러가지 형태가 가능할 것이다)를 관찰하는 패턴이다.  
이벤트 시스템을 예로 살펴보자.
1. DOM 객체에 click 이벤트 핸들러를 등록한다.  => `Observer`가 `Subject`를 구독했다.  
설계하기에 따라서 이벤트 핸들러 자체가 `Observer`가 될 수도있고, Observer 객체가 따로 존재할 수도 있을 것이다. 개념적으로만 접근하자.
2. 누군가 해당 DOM 을 클릭했다. => `Subject`가 `Observer`에게 자신의 상태를 `notify`했다.  
DOM 은 자신의 상태를 Event 객체로 만들어서 등록된 EventHandler 에 넘겨줄 것이다.

위 예를 봤을때 `Observer`와 `Subject`는 직접적으로 메세지를 주고받는 것을 알 수 있다.  
이 특징 때문에 설계하기에 따라서 `Observer`와 `Subject`는 서로의 상태를 추적/기록하고 더 강하게 `Coupling`될 수 있다.
그리고 이런 차이때문에 PubSub 패턴은 대게 app 모듈과 app 모듈 사이의 통신에 사용되고,  
Observer 패턴은 app 모듈 내의 모듈들 사이에서 사용된다.

이런 개념적인 차이는 분명 존재하지만, 구현하기에 따라서 `PubSub`에 가까운 `Observer`, `Observer`에 가까운 `PubSub`은 얼마든지 존재할 수 있다.

먼저 기존에 class 기반 언어에서 사용하던 방법과 같이, `Observer`와 `Observerble`을 상속받아 사용할 수 있도록 만들어보자. 
~~~javascript
// Observer 클래스이다. 받은 상태에 따라 update 하는 메소드를 가진다.
class Observer {
  constructor(observable) {
    if (observable && observable.register) {
      observable.register(this);
    }
  }
  update(state) {
    throw new Error('implement update method of observer')
  }
  subscribe(observable) {
    observable.register(this);
  }
  unsubscribe(observable) {
    observable.unregister(this);
  }
}
// Subject의 역할을 할 수 있게 만들어주는 Observable 클래스이다.
// 이 클래스를 상속받는 클래스는 관찰 가능한(Observable) Subject 클래스가 된다.
class Observable {
  constructor() {
    this.observers = [];
  }
  notify(state) {
    this.observers.forEach(observer => {
      observer.update(state);
    });
  }
  register(observer) {
    this.observers.push(observer);
    return observer;
  }
  unregister(observer) {
    this.observers.splice(this.observers.indexOf(observer), 1);
  }
}
~~~
위와 같이 Observer 와 Observable 를 설계하고, 다음과 같은 예를 만들어보자.
`body`를 클릭하면 `body`에 `<div>posX: 클릭한 포지션X, posY: 클릭한 포지션Y</div>`를 붙여주는 예를 이 패턴을 활용해서 만들어보자.
~~~javascript
class AppendDivElementObserver extends Observer {
  constructor(observable) {
    super(observable);
  }
  update({ posX, posY }) {
    const divElement = document.createElement('div');
    divElement.innerHTML = `posX: ${posX}, posY: ${posY}`;
    document.body.appendChild(divElement);
  }
}

class ClickObservable extends Observable {
  constructor(element) {
    super();
    const self = this;
    element.addEventListener('click', function(e) {
      self.notify({ 
        posX: e.clientX,
        posY: e.clientY 
      });
    }); 
  }
}

const clickBodySubject = new ClickObservable(document.body);
const appendDivElementObserver = new AppendDivElementObserver(clickBodySubject);

setTimeout(function() {
  appendDivElementObserver.unsubscribe(clickBodySubject);
}, 5000);
~~~
이건 Observer 패턴을 사용하는 하나의 예일 뿐 다양한 방식으로 구현할 수 있을 것이다.
  
Observer 패턴을 응용해서 Reactive Programming 을 지향하는 `RxJS`라는 라이브러리가 있다.  
이 라이브러리의 Observer 패턴을 흉내내보면서 어떻게 업그레이드 시켰는지 확인해보자.
~~~javascript
class Observer {
  constructor(handlers) {
    this.handlers = handlers;
    this.doNotObserve = false;
  }
  next(state) { // Subject 의 state 변화에 따라서 호출된다.
    if (this.handlers.next && !this.doNotObserve) {
      this.handlers.next(state);
    }
  }
  complete(message) { // Subject 가 완료되면 관찰을 그만두고 완료핸들링을 한다.
    if (this.doNotObserve) {
      return;
    }
    if (this.handlers.complete) {
      this.handlers.complete(message);
    }
    this.unsubscribe();
  }
  error(error) { // Subject 에서 에러가 발생하면 관찰을 그만두고 에러핸들링을한다.
    if (this.doNotObserve) {
      return;
    }
    if (this.handlers.error) {
      this.handlers.error(error);
    }
    this.unsubscribe();
  }
  unsubscribe() { // 관찰을 그만둔다.
    this.doNotObserve = true;
  }
}

class Observable {
  constructor(subjectAction) {
    this._subjectAction = subjectAction; 
  }
  subscribe(observerHandler) {
    const observer = new Observer(observerHandler);
    this._subjectAction(observer);
    
    return {
      unsubscribe() {
        observer.unsubscribe();
      }
    }
  }
}
~~~  
새로만든 이번 패턴을 이용해서 처음에 구현해봤던 Observer 와 같은 기능을 구현해보자.  
`body`를 클릭하면 `body`에 `<div>posX: 클릭한 포지션X, posY: 클릭한 포지션Y</div>`를 붙여주는 기능이다.  
~~~javascript
const fromEvent = function(element, eventName) {
  return new Observable(observer => {
    element.addEventListener(eventName, function(e) {
      observer.next(e);
    })
  })
};

const clickBodySubject = fromEvent(document.body, 'click');
const observer = clickBodySubject.subscribe({
  next(e) {
    const divElement = document.createElement('div');
    divElement.innerHTML = `posX: ${e.clientX}, posY: ${e.clientY}`;
    document.body.appendChild(divElement);
  },
  complete() {
    console.log('complete handling');
  },
  error() {
    console.log('error handling');
  }
});

setTimeout(function() {
  observer.unsubscribe();
}, 5000);
~~~
위의 예제들은 완전한 예제가 아니다. unsubscribe 을 한다면, removeEventListener() 를 해줄 필요가 있는데 그런 부분들에 대한 설정도, 확장성도 부족하다.  
다만 이해를 돕기 위한 예제이고 지속적으로 생각하고 사용하면서 우선 익숙해져야할 과제인 것 같다.
## 참조
https://hackernoon.com/observer-vs-pub-sub-pattern-50d3b27f838c
https://medium.com/@fknussel/a-simple-observable-implementation-c9c809c89c69