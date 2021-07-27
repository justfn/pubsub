<div align="center">
  <h1> 发布订阅 publish & subscribe </h1>
</div>

<p>
通过<b>订阅(subscribe)</b><b>频道(channel)</b>, 
当<b>发布(publish)</b><b>消息(message)</b>到频道(channel), 
来接收消息message以便执行某些<b>动作(action)</b> 
</p>


## 特点
* 轻量, 无依赖 
* 自由灵活, channel key 可为(除函数和NaN外的)任意JS变量 (推荐使用引用类型,避免重复) 
* ES6+ 

## 安装
  npm i -S @justfn/pubsub 


# 举些栗子 
```javascript
// 引入pubsub功能库 
import PubSub from '@justfn/pubsub';

```

### 基本使用 
```javascript
// 定义一个频道, 作为媒介 
const myChannel = 'my-tv';

// 定义需要执行的某些操作 
let myAction = (channel, ...messages)=>{
  // 
  console.log(channel, ...messages);
};

// 订阅频道, 接收到消息后执行响应 
let token = PubSub.sub(myChannel, myAction);

// 向指定频道发送消息 
PubSub.pub(myChannel, 'hello', 'word', '!');
// 异步发送 
PubSub.pubAsync(myChannel, 'hello', 'word', 'async');
```

### 取消订阅 
```javascript
const myChannel01 = 'channel01';
const myChannel02 = 'channel02';
let myAction01 = (channel, ...msgs)=>{
  console.log('action01', channel, ...msgs);
};
let myAction02 = (channel, ...msgs)=>{
  console.log('action02', channel, ...msgs);
};
let token01 = PubSub.sub(myChannel01, myAction01);
let token02 = PubSub.sub(myChannel01, myAction02);
let token03 = PubSub.sub(myChannel02, myAction01);
let token04 = PubSub.sub(myChannel02, myAction02);

// 取消指定的订阅 
PubSub.clearSubs(token01);
// 取消指定频道的所有订阅 
PubSub.clearSubs(myChannel01);
// 取消指定动作的所有订阅 
PubSub.clearSubs(myAction02);

PubSub.pub(myChannel01, '01')
PubSub.pub(myChannel02, '02')
```


### 获取订阅的动作
```javascript
// 获取指定频道的所有action, 返回一个 action 组成的数组  
PubSub.getActions(myChannel);

```


### 统一错误处理 
```javascript 

PubSub.onError(myChannel, (errEvt)=>{
  const {
    channel,
    msgs,
    action,
    err,
  } = errEvt;
  console.log(errEvt);
})

```

### 多频道处理 
```javascript
// 订阅发布的 channel, 可为一数组(由多个channel组成的数组 ), 表示同时订阅/发布多个 channel 

const myChannel01 = 'c01';
const myChannel02 = 'c02';
const myChannel03 = 'c03';
const myChannel04 = 'c04';
const myAction = (channel, ...msgs)=>{
  // 注意: 此时 channel 为 发布的channel值 
  console.log(channel, ...msgs);
}

PubSub.sub([myChannel01, myChannel02, myChannel04], myAction);

PubSub.pub([ myChannel02, myChannel03, myChannel04 ], '同时发布到3个频道');
PubSub.pub([ myChannel01 ], {key: '只发布到一个频道'});

```

### 立即执行的订阅
```javascript
// 当发布早于订阅先执行时, 订阅的动作将执行不了, 此时可使用立即执行一次的订阅来解决 

const myChannel = Symbol('0');
const myAction = (channel, ...msgs)=>{
  console.log(channel, ...msgs);
}

PubSub.pub(myChannel, '早于订阅前已发布的消息 001 ');
PubSub.pub(myChannel, '早于订阅前已发布的消息 002 ');

PubSub.sub(myChannel, myAction, true); // 此处仍能收到前面最后一次的发布消息 

```
 

### 频道使用引用类型, 防止重复定义使用 
```javascript 
// 定义两个channel 
const myChannel01 = {};
const myChannel02 = {}; // 因为channel为引用类型, 两个channel是不同的 

const token01 = PubSub.sub(myChannel01, (channel, ...msgs)=>{
  console.log( channel===myChannel01, channel===myChannel02 );
  console.log(channel, ...msgs);
})
const token02 = PubSub.sub(myChannel02, (channel, ...msgs)=>{
  console.log( channel===myChannel01, channel===myChannel02 );
  console.log(channel, ...msgs);
})

PubSub.pub(myChannel01);
PubSub.pub(myChannel02);
```

### 便捷使用方式  
```javascript 
const myChannel = Symbol('my-channel');
const myAction = (channel, ...msgs)=>{
  console.log(channel, ...msgs);
}
const myChannelHandler = PubSub.use(myChannel); 

// 相当于 PubSub.sub(myChannel, myAction);
myChannelHandler.sub(myAction);
// 相当于 PubSub.pub(myChannel, 'jello', 'word');
myChannelHandler.pub('hello', 'word');
// 相当于 PubSub.pubAsync(myChannel, 'jello', 'word');
myChannelHandler.pubAsync('hello', 'word');
// 相当于 PubSub.getActions(myChannel);
myChannelHandler.getActions();
// 相当于 PubSub.clearSubs(myChannel);
myChannelHandler.clearSubs();
// 相当于 PubSub.onError(myChannel, errEvt=>console.log(errEvt));
myChannelHandler.onError(errEvt=>console.log(errEvt));


```

### 顶级channel, 表示任意channel 
```javascript 
// AnyChannel 一个特殊定义的频道,用来表示任意channel 
import PubSub, { AnyChannel, } from '@justfn/pubsub';

const myChannel01 = Symbol('01');
const myChannel02 = Symbol('02');
const myAction01 = (channel, ...msgs)=>{
  console.log('action01', channel, ...msgs);
}
const myAction02 = (channel, ...msgs)=>{
  console.log('action02', channel, ...msgs);
}

PubSub.sub(AnyChannel, myAction01); // 所有的发布都将被订阅到 
PubSub.sub(myChannel01, myAction01);
PubSub.sub(myChannel01, myAction02);
PubSub.sub([ myChannel02 ], myAction01);

PubSub.pub(myChannel01, 'pub01');
PubSub.pub(myChannel02, 'pub02');
PubSub.pub(AnyChannel, 'pub-all');  // 所有的频道都将接收到订阅 

``` 






## License 

MIT 

## More ...
给我留言吧... 

