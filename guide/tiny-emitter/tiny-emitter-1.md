#### 一、初识tiny-emitter

##### 1、tiny-emitter是什么？

`tiny-emitter`是一个极小的事件发布订阅库。

##### 2、克隆代码/安装依赖

```
git clone https://github.com/scottcorgan/tiny-emitter.git
```

```sh
$ npm install
```

##### 3、如何引入和使用`tiny-emitter`

```sh
$ npm install tiny-emitter --save
```

##### （1）tiny-emitter

```javascript
var Emittter = require('tiny-emitter');
var emitter = new Emitter(); 
emitter.on('some-event', function(arg1, arg2, arg3) {
    // 
})
emitter.emit('some-event', 'arg1 value', 'arg2 value', 'arg3 value');
```

##### （2）tiny-emitter/instance

```javascript
var emitter = require('tiny-emitter/instance');
emitter.on('some-event', function(arg1, arg2, arg3) {
    // 
})
emitter.emit('some-event', 'arg1 value', 'arg2 value', 'arg3 value');
```

##### 4、`tiny-emitter`实例方法

###### （1）on(event, callback[, context])

`on`方法用于订阅事件。

###### （2）once(event, callback[, context])

`once`方法用于订阅事件一次。

###### （3）off(event[, callback])

`off`方法用于取消订阅一个事件或全部事件，如果没有传入`callback`，则取消订阅全部事件。

###### （4）emit(event[, arguments...])

`emit`方法用于触发一个事件。

#### 二、tiny-mitter/test/index.js

##### 1、准备

```javascript
var Emitter = require('../index');
var emitter = require('../instance');
var tape = require('tape');
```

##### 2、订阅一个事件

```javascript
test('subscribe to an event', function(t) {
    var emitter = new Emitter();
    emitter.on('test', function() {});
    t.equal(emitter.e.test.length, 1, 'subscribe to event');
    t.end();
})
```

![tiny-emitter-1](../../images/tiny-emitter/tiny-emitter-1.jpg)

##### 3、订阅一个带有上下文的事件

```javascript
test('subscribes to an event with context', function (t) {
  var emitter = new Emitter();
  var context = {
    contextValue: true
  };

  emitter.on('test', function () {
    t.ok(this.contextValue, 'is in context');
    t.end();
  }, context);

  emitter.emit('test');
});
```

![tiny-emitter-2](../../images/tiny-emitter/tiny-emitter-2.jpg)

##### 4、订阅只执行一次的事件

```javascript
test('subscibes only once to an event', function (t) {
  var emitter = new Emitter();

  emitter.once('test', function () {
    t.notOk(emitter.e.test, 'removed event from list');
    t.end();
  });

  emitter.emit('test');
});
```

##### 5、当订阅一次的事件时可以保持上下文

```javascript
test('keeps context when subscribed only once', function (t) {
  var emitter = new Emitter();
  var context = {
    contextValue: true
  };

  emitter.once('test', function () {
    t.ok(this.contextValue, 'is in context');
    t.notOk(emitter.e.test, 'not subscribed anymore');
    t.end();
  }, context);

  emitter.emit('test');
});
```

##### 6、触发一次事件

```javascript
test('emits an event', function (t) {
  var emitter = new Emitter();

  emitter.on('test', function () {
    t.ok(true, 'triggered event');
    t.end();
  });

  emitter.emit('test');
});
```

##### 7、将所有参数传递给事件监听器

```javascript
test('passes all arguments to event listener', function (t) {
  var emitter = new Emitter();

  emitter.on('test', function (arg1, arg2) {
    t.equal(arg1, 'arg1', 'passed the first argument');
    t.equal(arg2, 'arg2', 'passed the second argument');
    t.end();
  });

  emitter.emit('test', 'arg1', 'arg2');
});
```

##### 8、取消订阅带有名称的所有事件

```javascript
test('unsubscribes from all events with name', function (t) {
  var emitter = new Emitter();
  emitter.on('test', function () {
    t.fail('should not get called');
  });
  emitter.off('test');
  emitter.emit('test')

  process.nextTick(function () {
    t.end();
  });
});
```

##### 9、取消订阅带有名称和回调函数的单个事件

```javascript
test('unsubscribes single event with name and callback', function (t) {
  var emitter = new Emitter();
  var fn = function () {
    t.fail('should not get called');
  }

  emitter.on('test', fn);
  emitter.off('test', fn);
  emitter.emit('test')

  process.nextTick(function () {
    t.end();
  });
});
```

##### 10、订阅两次时取消订阅带有名称和回调的单个事件

```javascript
test('unsubscribes single event with name and callback when subscribed twice', function (t) {
  var emitter = new Emitter();
  var fn = function () {
    t.fail('should not get called');
  };

  emitter.on('test', fn);
  emitter.on('test', fn);

  emitter.off('test', fn);
  emitter.emit('test');

  process.nextTick(function () {
    t.notOk(emitter.e['test'], 'removes all events');
    t.end();
  });
});
```

##### 11、当订阅两次乱序时，取消订阅带有名称和回调的单个事件

```javascript
test('unsubscribes single event with name and callback when subscribed twice out of order', function (t) {
  var emitter = new Emitter();
  var calls = 0;
  var fn = function () {
    t.fail('should not get called');
  };
  var fn2 = function () {
    calls++;
  };

  emitter.on('test', fn);
  emitter.on('test', fn2);
  emitter.on('test', fn);
  emitter.off('test', fn);
  emitter.emit('test');

  process.nextTick(function () {
    t.equal(calls, 1, 'callback was called');
    t.end();
  });
});
```

##### 12、移除另一个事件中的一个事件

```javascript
test('removes an event inside another event', function (t) {
  var emitter = new Emitter();

  emitter.on('test', function () {
    t.equal(emitter.e.test.length, 1, 'event is still in list');

    emitter.off('test');

    t.notOk(emitter.e.test, 0, 'event is gone from list');
    t.end();
  });

  emitter.emit('test');
});
```

##### 13、即使在事件回调中取消订阅，也会触发事件

```javascript
test('event is emitted even if unsubscribed in the event callback', function (t) {
  var emitter = new Emitter();
  var calls = 0;
  var fn = function () {
    calls += 1;
    emitter.off('test', fn);
  };

  emitter.on('test', fn);

  emitter.on('test', function () {
    calls += 1;
  });

  emitter.on('test', function () {
    calls += 1;
  });

  process.nextTick(function () {
    t.equal(calls, 3, 'all callbacks were called');
    t.end();
  });

  emitter.emit('test');
});
```

##### 14、在添加任何事件之前取消没有任何影响

```javascript
test('calling off before any events added does nothing', function (t) {
  var emitter = new Emitter();
  emitter.off('test', function () {});
  t.end();
});
```

##### 15、触发尚未订阅的事件

```javascript
test('emitting event that has not been subscribed to yet', function (t) {
  var emitter = new Emitter();

  emitter.emit('some-event', 'some message');
  t.end();
});
```

##### 16、取消订阅单个事件，该事件带有名称和回调，且该事件已被订阅一次

```javascript
test('unsubscribes single event with name and callback which was subscribed once', function (t) {
  var emitter = new Emitter();
  var fn = function () {
    t.fail('event not unsubscribed');
  }

  emitter.once('test', fn);
  emitter.off('test', fn);
  emitter.emit('test');

  t.end();
});
```

##### 17、暴露一个实例

```javascript
test('exports an instance', function (t) {
  t.ok(emitter, 'exports an instance')
  t.ok(emitter instanceof Emitter, 'an instance of the Emitter class');
  t.end();
});
```

#### 三、tiny-emittter/index.js

#### 四、tiny-emitter/index.d.ts

#### 五、收获


