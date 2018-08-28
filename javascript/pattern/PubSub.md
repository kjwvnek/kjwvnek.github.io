# Publish Subscribe 패턴
이 패턴에 대한 개념은 'Wiki'에 정리가 잘 되있기 때문에 한번 읽어보는게 좋을 듯 하다.  
[Wiki:Publish-Subscribe-Pattern](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)

## 요약
요약하자면 Publish / Subscribe 패턴은 메세징 패턴의 하나로 아주 단순한 개념이다.  
공급(Publish)이 발생하면, 그 공급을 바라보는 소비자들(Subscriber)들이 공급된 메세지를 전달받는 것이다.  
마케팅 분야의 `Pull Push Strategy`의 `Pull Strategy`와 유사한 특징을 가진다.  
소비자는 공급자에 대해 알 필요가 없다. 단순히 공급(Push)해주면 받을 뿐이다.
  
이 패턴은 위와같이 `Publisher => Subscriber`로 단방향의 데이터 흐름을 가진다는 특징이 있다.  
또한 단순한 공급/구독의 관계(구독자는 공급자를 신경쓰지않고, 공급자도 구독자를 신경쓰지않는다)를 통해 `Loose Coupling`이 발생하기 때문에 많은 경우에 비동기로 설계된다는 특징이 있다.  
이런 `Loose Coupling`은 이 패턴의 장점이기도하지만 단점이 되기도한다.  
구독자(Subscriber) 입장에서 공급자(Publisher)가 보내는 메세지가 어떤 명세를 가지는지 모르기 때문에 어떻게 핸들링하는게 안전한지 모를 수 있다. 
  
PubSub 패턴을 객체지향적인 설계를 하기 위해서 중간에 'Broker' 역할을 하는 'Channel'을 만들고 이를 중심으로 코드를 살펴보자.  
이 'Channel'을 어떻게 관리하느냐에 따라서 **Topic-based Pub/Sub**과 **Content-based Pub/Sub**으로 분류될 수 있다.  
먼저 'Topic-based'은 'Channel'을 **Topic**을 중심으로 관리(필터링)하기 때문에 아래와 같은 흐름을 따를 것이다.

1. `Subscriber(구독자)`는 `Channel(브로커)`의 어떤 `Topic(주제)`를 구독한다.
2. 해당 `Channel`에서 이 `Topic`을 구독하는 `Subscriber`는 여러명일 수도 있을 것이다.
3. `Publisher(공급자)`가 `Channel`의 특정 `Topic`에 `Message(Data)`를 공급한다.
4. 이 `Message`는 해당 `Channel`에서 해당 `Topic`을 구독하고 있는 모든 `Subscriber`에게 `notify(전달)`된다.

그리고 'Content-based'는 같은 느낌으로 **Content**를 중심으로 관리(필터링)을 하기 때문에 아래와 같은 흐름을 따른다.

1. `Subscriber(구독자)`는 `Channel(브로커)`에게 내가 메세지를 받을 `Content(기준:영어 뜻은 내용이지만 기준이라는 표현이 더 어울릴듯 하다)`를 말해준다.  
2. 해당 `Channel`을 구독하고 있는 `Subscriber`는 역시 여러명이 될 수 있을 것이다.
3. `Publisher(공급자)`가 `Channel`에 `Message(Data)`를 공급한다.
4. 이 `Message`의 내용이 `Subscriber`가 받고 싶은 메세지일 때, 그 `Subscriber`들에게 `notify(전달)`된다.

이 'Topic-based'와 'Content-based'를 적절히 섞은 설계도 가능할 것이다.  
이 패턴의 기본적인 개념은 `Publish`와 `Subscribe`라는 것을 유념하자.

## Topic Based Pub/Sub 구현하기
위의 개념을 javascript es6 문법으로 간단하게 구현해보자.  
~~~javascript
class TopicBasedChannel {
  constructor() {
    this.token = 0;
    this.topics = {};
  }
  
  subscribe(topic, subscriber) {
    if (!this.topics.hasOwnProperty(topic)) {
      this.topics[topic] = {};
    }
    
    const token = this.token++;
    this.topics[topic][token] = subscriber;
    return token;
  }
  
  unsubscribe(token) {
    for (let topic in this.topics) {
      if (this.topics.hasOwnProperty(topic) && this.topics[topic].hasOwnProperty(token)) {
        delete this.topics[topic][token];
        return true;
      }
    }
    return false;
  }
  
  publish(topic, message) {
    if (!this.topics.hasOwnProperty(topic)) {
      return false;
    }
    
    const subscribers = this.topics[topic];
    for (let token in subscribers) {
      if (subscribers.hasOwnProperty(token)) {
        this._notify(topic, message, subscribers[token]);
      }
    }
  }
  
  async _notify(topic, message, subscriber) {
      subscriber(topic, message);
  }
}
~~~
익숙한 Javascript Event 기반 DOM 핸들링처럼 사용해보자.  
두 이벤트핸들러(구독자)가 클릭이벤트(토픽)를 구독하고 있다.  
이때 클릭 이벤트가 발생(공급자가 공급) 했을 때 이벤트핸들러가 실행(메세지 전달)될 것이다.  
그리고 구독을 취소한 다음에 이벤트를 다시 발생시켜보자.  
~~~javascript
const EventChannel = new TopicBasedChannel();
const subscribeToken1 = EventChannel.subscribe('click', function(event, message) {
  console.log(event); // click
  console.log(message); // EventObject
});
const subscribeToken2 = EventChannel.subscribe('click' ,function(event, message) {
  console.log('subscribe click event!');
});

EventChannel.publish('click', { thisIs: 'click event Object' });

EventChannel.unsubscribe(subscribeToken1);
setTimeout(function() {
  EventChannel.publish('click', { thisIs: 'click event Object' });
}, 1000);
~~~
이런 이벤트핸들링 패턴은 Pub/Sub 패턴이 아닌 Observer 패턴으로 구현되어있으며, 이는 Pub/Sub 패턴과 어떻게 다른지는  Observer.md 에서 다루겠다.  
이 예제는 단순히 Topic-based Pub/Sub 패턴을 이해하기 위한 예제이다.

## Content Based Pub/Sub 구현하기
마찬가지로 javascript es6 문법으로 간단하게 구현해보자.
~~~javascript
class ContentBasedChannel {
  constructor() {
    this.token = 0;
    this.subscribers = {};
  }
  
  subscribe(filter, subscriber) {
    const token = this.token++;
    this.subscribers[token] = {
      filter: filter,
      subscriber: subscriber
    };
    return token;
  }
  
  unsubscribe(token) {
    if (this.subscribers.hasOwnProperty(token)) {
      delete this.subscribers[token];
      return true;
    }
    return false;
  }
  
  publish(message) {
    for (let token in this.subscribers) {
      if (this.subscribers.hasOwnProperty(token) && this.subscribers[token].filter(message)) {
        this._notify(message, this.subscribers[token].subscriber);
      }
    }
  }
  
  async _notify(message, subscriber) {
    subscriber(message);
  }
}
~~~
이번에는 아무 메세지나 들어올 수 있는 `randomStore`가 있다고 해보자.  
메세지가 문자열일때는 메세지를 받는 구독자가 있고, 문자열 또는 숫자일때 메세지를 받는 구독자가 있다.  
다음과 같이 코드를 실행해보자.
~~~javascript
const randomStore = new ContentBasedChannel();
const subscribeToken1 = randomStore.subscribe(message => (typeof message === 'string'), function(message) {
  console.log(`type of message is string! and message: ${message}`);
});
const subscribeToken2 = randomStore.subscribe(message => (typeof message === 'number' || typeof message === 'string'), function(message) {
  console.log(`typeof message is number or string! and message: ${message}`);
});

randomStore.publish('my name is message');
randomStore.publish(100);

randomStore.unsubscribe(subscribeToken1);
setTimeout(function() {
  randomStore.publish('second string message~');
}, 1000);
~~~
이처럼 Javascript 언어로 Publish-Subscribe 패턴을 구현해봤다.  
이 구현은 이해를 돕기위한 것일 뿐, 이뿐만이 아니라 다양하게 구현할 수 있다. 패턴의 원리와 추구하는 바를 생각해서 구현하면 되는 것이다.

## 참조
1. https://msdn.microsoft.com/en-us/magazine/hh201955.aspx
2. Ken 님 자료
