# Observer Pattern
이 패턴을 보기 전에 Publish-Subscribe 패턴을 먼저 보는 것을 추천한다. (PubSub.md)  
그리고 Observer 패턴과 어떤 차이점이 있는지 비교해보자.  

## 요약
옵저버(Observer) 패턴은 해석하기 그대로 무언가를 지속적으로 관찰하기 위한 패턴이다.  
이 옵저버 패턴의 구성요소는 어떤게 있을까? 관찰을 하는 `Observer(관찰자)`와 관찰의 대상이 되는 `Subject(주체)`가 있을 것이다.  
그리고 `Observer`는 자기가 관찰하는 `Subject`의 `State(상태)`변화에 따라서 `Message(메세지)`를 받을 것이다.
  
여기까지 봤을 때 `Topic-based PubSub` 패턴과 유사해보이지만 조금 다른 것을 알 수 있다.  
**PubSub 패턴**의 핵심은 **Loose Coupling**이었다. 메세지를 공급하는 '공급자'와 메세지를 받는 '구독자'가 서로를 모름으로써 의존성을 지웠고 이는 PubSub 패턴의 장점이되기도 단점이 되기도 했다.

하지만 **Observer 패턴**은 단순히 구독한 채널의 메세지를 전달받는 것이 아니라 해당 주체를 관찰하는 패턴이다.(구현하기에 따라 형태는 다양할 것이다.)  
이벤트 시스템을 예를 들며 흐름을 살펴보자.
1. DOM 객체에 click 이벤트 핸들러를 등록한다. >> `Observer`가 `Subject`를 구독했다.  
구현하기에 따라서 이벤트 핸들러 자체가 `Observer`가 될 수도있고, Observer 객체가 따로 관리될 수도 있을 것이다. 개념적으로만 살펴보자.
2. 누군가 해당 DOM 을 클릭했다. >> `Subject`가 `Observer`에게 자신의 상태를 `notify`했다.  
이벤트가 발생할 때, DOM 은 자신의 상태를 Event 객체로 만들어서 등록된 EventHandler 에 넘겨줄 것이다.

위 예를 봤을때 `Observer`와 `Subject`는 직접적으로 메세지를 주고받는 것을 알 수 있다.
`Observer`는 `Subject`의 상태를 '관찰'한다는 특징을 가지기 때문에, 대게 더 강하게 `Coupling` 되도록 구현된다.  

이 특징 때문에 주로 PubSub 패턴은 app 모듈과 app 모듈 사이에서, Observer 패턴은 앱 내의 모듈과 모듈 사이에서 사용된다.
위에서 살펴봤다시피 개념적인 차이는 분명 존재하지만, 구현하기에 따라서 `PubSub`에 가까운 `Observer`, `Observer`에 가까운 `PubSub`은 얼마든지 존재할 수 있다.

`Observer`와 `Observerble`을 이용해서 옵저버 패턴을 사용할 수 있도록 간단히 구현해보자.
그리고 이것을 이용해서 `clickBody`이벤트가 발생하면 `body`에 `<div>posX: 포지션X, posY: 포지션Y</div>`를 붙여주는 기능을 구현해보자. 
~~~javascript
class Observer {
  constructor(handler) {
    this.handler = handler;
    this.observe = false;
  }
  
  notify(message) {
    if (this.observe && this.handler) {
      this.handler(message);
    }
  }
}

class Observable {
  constructor(subject) {
    this.subject = subject;
    this.observers = [];
  }
  
  registerObserver(observer) {
    observer.observe = true;
    this.observers.push(observer);
  }
  
  unregisterObserver(observer) {
    observer.observe = false;
    this.observers.splice(this.observers.indexOf(observer), 1);
  }
  
  notifyObservers() {
    const self = this;
    this.subject({
      notify(message) {
        self.observers.forEach(observer => {
          observer.notify(message);
        });
      }
    });
  }
}

const appendDivElementObserver = new Observer(function({ posX, posY }) {
  const divElement = document.createElement('div');
  divElement.innerHTML = `posX: ${posX}, posY: ${posY}`;
  document.body.appendChild(divElement);
});

const clickBodyObservable = new Observable(function(observer) {
  const message = {
    posX: 'dummy posX',
    posY: 'dummy posY'
  };
  observer.notify(message);
});

clickBodyObservable.registerObserver(appendDivElementObserver);
clickBodyObservable.notifyObservers();
clickBodyObservable.notifyObservers();
~~~

위에서 구현한 허접한 Observer 패턴과 달리 이 패턴을 응용해서 Reactive Programming 을 지향하는 `RxJS`라는 라이브러리가 있다.  
이 라이브러리를 간단하게나마 흉내내보면서 어떤 방식으로 구현하는지 확인해보자.  
그리고 이것을 이용해서 5초 동안만 `body`를 클릭하면 `body`에 `<div>posX: 클릭한 포지션X, posY: 클릭한 포지션Y</div>`를 붙여주는 기능을 구현해보자.
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

const fromEvent = function(element, eventName) {
  return new Observable(observer => {
    element.addEventListener(eventName, function(e) {
      observer.next(e);
    });
    
    setTimeout(() => {
      observer.complete()
    }, 5000);
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
~~~
이 구현에는 허점이 존재한다. `fromEvent`에서 `addEventListener()`를 한 부분이 불필요해지면 (complete, error, 또는 옵저버가 없는 경우) `removeEventListener()` 를 해주는 것이 좋은데 그 부분에 대한 구현이 없다.  
다양한 Observer 패턴을 접하며 조금 더 생각해봐야할 과제인 것 같다.

## 참조
https://hackernoon.com/observer-vs-pub-sub-pattern-50d3b27f838c
https://medium.com/@fknussel/a-simple-observable-implementation-c9c809c89c69