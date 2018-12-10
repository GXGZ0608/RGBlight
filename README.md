# Aliyun IoT Device SDK for Javascript

> 使用 Javascript 将设备接入到阿里云 IoT 套件和 LinkDevelop。

## 安装

> 安装 Node.js 运行环境，版本 `>=4.0.0` 。

通过 rap 包管理工具安装：

```bash
rap install aliyun-iot-device-sdk --save
```

## 设备接入云端

```javascript
var aliyunIot = require('aliyun-iot-device-sdk');

var device = aliyunIot.device({
  productKey: '<productKey>',
  deviceName: '<deviceName>',
  deviceSecret: '<deviceSecret>'
});

device.on('connect', () => {
  console.log('connect successfully!');
});
```

## IoT 套件基础版 API

## 设备上报数据

```javascript
device.publish('/<productKey>/<deviceName>/update', 'hello world!');
```

## 云端下行消息监听

```javascript
device.subscribe('/<productKey>/<deviceName>/get');

device.on('message', (topic, payload) => {
  console.log(topic, payload.toString());
});
```

## Link Develop 和 IoT 套件高级版 API

IoT 套件高级版封装了物模型定义与 Alink 异步协议，SDK 封装使得设备与云端通信时不需要关心 MQTT topic，只需要调用属性上报（<a href="#postProps"><code>aliyunIot.device#<b>postProps()</b></code></a>）、服务监听（<a href="#serve"><code>aliyunIot.device#<b>serve()</b></code></a>）、事件上报（<a href="#postEvent"><code>aliyunIot.device#<b>postEvent()</b></code></a>）等相关 API。

### 设备属性上报

```javascript
device.postProps({
  CurrentTemperature: 25
});
```

调用 `device.postProps()`  等同于执行以下代码：

```javascript
// 消息 id 用来确保服务端异步响应的时序
var msgId = "123";

// 发布属性上报 topic
device.publish('/sys/<productKey>/<deviceName>/thing/event/property/post', JSON.stringify({
  {
    id: msgId,
    version: '1.0',
    params: {
      CurrentTemperature: 25,
    },
    method: 'thing.event.property.post'
  }
}));

// 监听属性上报响应 topic
device.subscribe('/sys/<productKey>/<deviceName>/thing/event/property/post_reply');

device.on('message', function(topic, payload){
  var res = payload.toString();
  if (res.id === msgId) {
    // 在这里处理服务端响应
    console.log(res.data);
  }
});
```

###  监听云端下发的服务调用消息

```javascript
// 监听云端设置属性服务消息
device.serve('property/set', function(params) {
  // 处理服务参数
});
```

## API

* <a href="#device"><code>aliyunIot.<b>device()</b></code></a>
* <a href="#publish"><code>aliyunIot.device#<b>publish()</b></code></a>
* <a href="#subscribe"><code>aliyunIot.device#<b>subscribe()</b></code></a>
* <a href="#postProps"><code>aliyunIot.device#<b>postProps()</b></code></a>
* <a href="#postEvent"><code>aliyunIot.device#<b>postEvent()</b></code></a>
* <a href="#serve"><code>aliyunIot.device#<b>serve()</b></code></a>
* <a href="#gateway"><code>aliyunIot.<b>gateway()</b></code></a>
* <a href="#addTopo"><code>aliyunIot.gateway#<b>addTopo()</b></code></a>
* <a href="#getTopo"><code>aliyunIot.gateway#<b>getTopo()</b></code></a>
* <a href="#removeTopo"><code>aliyunIot.gateway#<b>removeTopo()</b></code></a>
* <a href="#login"><code>aliyunIot.gateway#<b>login()</b></code></a>
* <a href="#logout"><code>aliyunIot.gateway#<b>logout()</b></code></a>
* <a href="#postSubDeviceProps"><code>aliyunIot.gateway#<b>postSubDeviceProps()</b></code></a>
* <a href="#postSubDeviceEvent"><code>aliyunIot.gateway#<b>postSubDeviceEvent()</b></code></a>
* <a href="#serveSubDeviceService"><code>aliyunIot.gateway#<b>serveSubDeviceService()</b></code></a>
* <a href="#signUtil"><code>aliyunIot.<b>signUtil()</b></code></a>

<a name="device"></a>

### aliyunIot.device(options)

和云端建立连接，返回一个 `Device` 连接实例，入参：

* `options`
  * `productKey`   (`String`)
  * `deviceName`   (`String`)
  * `deviceSecret` (`String`)
  * `regionId`     (`String`) 阿里云 regionId
  * `tls`          (`Bool`)   是否开启 TLS 加密，Node.js 中如果开启将使用 TLS 协议进行连接，浏览器如果开启将上使用 WSS 协议

### Event `'connect'`

`function(connack) {}`

当连接到云端成功时触发。

### Event `'message'`

`function(topic, message) {}`

当接受到云端消息时触发，回调函数参数：

* `topic`  消息主题
* `message` 消息 payload

### Event `'error'`

`function(error) {}`

当设备不能连接到云端的时候触发。

<a name="publish"></a>

### aliyunIot.device#publish(topic, message, [options], [callback])

等同于 [mqtt.Client#publish()](https://github.com/mqttjs/MQTT.js/blob/master/README.md#publish) 方法。

<a name="unsubscribe"></a>

### aliyunIot.device#unsubscribe(topic, [callback])

等同于 [mqtt.Client#unsubscribe()](https://github.com/mqttjs/MQTT.js/blob/master/README.md#unsubscribe) 方法。

<a name="postProps"></a>

### aliyunIot.device#postProps(params, [callback])

上报物模型属性：

* `params` 属性参数，`Object` 类型
* `callback`
  * `err` 错误，比如超时或者 `res.code !== 200`
  * `res` 服务端 reply 消息内容

<a name="postEvent"></a>

### aliyunIot.device#postEvent(eventIdentifier, params, [callback])

上报物模型事件：

* `eventIdentifier` 事件 id `String` 类型
* `params` 事件参数，`Object` 类型
* `callback`
  * `err` 错误，比如超时
  * `res` 服务端 reply 消息内容

<a name="serve"></a>

### aliyunIot.device#serve(seviceIdentifier, [callback])

监听物模型服务：

* `seviceIdentifier` 服务 id `String` 类型
* `callback`
  * `params` 服务参数

值得注意的是，`serve` 方法返回的是一个 `deServe` 函数，可以通过调用 `deServe` 函数取消服务监听：

```javascript
var deServe = device.serve('turnOn', function(params) {
  // 收到 turnOn 服务消息后取消对该服务的监听
  deServe();
});
```

<a name="gateway"></a>

### aliyunIot.gateway(options)

和云端建立连接，返回一个网关 `Gateway` 类连接实例，继承自 `Device`  类。

* `options` 设备激活凭证
  * `productKey`
  * `deviceName`
  * `deviceSecret`

<a name="addTopo"></a>

### aliyunIot.gateway#addTopo(deviceSign, [callback])

添加子设备到拓扑

* `deviceSign` 子设备身份加签信息，通过调用 <a href="#signUtil"><code>aliyunIot.<b>signUtil()</b></code></a> 方法进行加签
* `callback`
  * `err` 错误，比如超时或者 `res.code !== 200`
  * `res` 服务端 reply 消息内容

<a name="getTopo"></a>

### aliyunIot.gateway#getTopo(thingId, [callback])

添加子设备到拓扑关系

* `thingId` 子设备身份
  * `productKey`
  * `deviceName`
* `callback`
  * `err` 错误，比如超时或者 `res.code !== 200`
  * `res` 服务端 reply 消息内容

<a name="removeTopo"></a>

### aliyunIot.gateway#removeTopo(thingId, [callback])

从拓扑关系里移除子设备

* `thingId` 子设备身份
  * `productKey`
  * `deviceName`
* `callback`
  * `err` 错误，比如超时或者 `res.code !== 200`
  * `res` 服务端 reply 消息内容

<a name="login"></a>

### aliyunIot.gateway#login(thingId, [callback])

子设备上线

* `deviceSign` 子设备身份加签信息，通过调用 <a href="#signUtil"><code>aliyunIot.<b>signUtil()</b></code></a> 方法进行加签
* `callback`
  * `err` 错误，比如超时或者 `res.code !== 200`
  * `res` 服务端 reply 消息内容

<a name="logout"></a>

### aliyunIot.gateway#logout(thingId, [callback])

子设备下线

* `thingId` 子设备身份
  * `productKey`
  * `deviceName`
* `callback`
  * `err` 错误，比如超时或者 `res.code !== 200`
  * `res` 服务端 reply 消息内容

<a name="postSubDeviceProps"></a>

### aliyunIot.gateway#postSubDeviceProps(thingId, params, [callback])

* `thingId` 子设备身份
  * `productKey`
  * `deviceName`
* `params` 属性参数，类型 `Object`
* `callback`
  * `err` 错误，比如超时或者 `res.code !== 200`
  * `res` 服务端 reply 消息内容

<a name="postSubDeviceEvent"></a>

### aliyunIot.gateway#postSubDeviceEvent(thingId, eventIdentifier, params, [callback])

* `thingId` 子设备身份
  * `productKey`
  * `deviceName`
* `eventIdentifier` 事件 id，`String` 类型
* `params` 事件参数，`Object`类型
* `callback`
  * `err` 错误，比如超时或者 `res.code !== 200`
  * `res` 服务端 reply 消息内容

<a name="serveSubDeviceService"></a>

### aliyunIot.gateway#serveSubDeviceService(thingId, serviceIdentifier, [callback])

* `thingId` 子设备身份
  * `productKey`
  * `deviceName`
* `serviceIdentifier` 服务 id，`String` 类型
* `callback`
  * `params` 服务参数

<a name="signUtil"></a>

### aliyunIot.signUtil()

设备身份连接加签工具函数

* `deviceId` 设备激活凭证
  * `productKey`
  * `deviceName`
  * `deviceSecret`
