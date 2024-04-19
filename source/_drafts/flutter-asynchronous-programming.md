---
title: Flutter异步编程
tags:
-----



#### 一、基础
##### 1.1 async-await

```dart
const String filename = 'with_keys.json';

void main() async {
  // Read some data.
  final fileData = await _readFileAsync();
  final jsonData = jsonDecode(fileData);

  // Use that data.
  print('Number of JSON keys: ${jsonData.length}');
}

Future<String> _readFileAsync() async {
  final file = File(filename);
  final contents = await file.readAsString();
  return contents.trim();
}
```

##### 1.2 Future

```dart

void main() async {
  Future<String> future = _readFileAsync();
  future.then((File file) {
    print('写入成功:${file.path}');
  }).catchError((err) {
    print("catchError:$err");
  }).whenComplete((){
    print("=======Complete=======");
  });
}
```

##### 1.3 Stream
+ 流的创建
+ 流的监听
+ 流的操作符
+ StreamController
+ StreamTransformer

```dart
//创建Stream
Stream<int> stream = Stream<int>.periodic(Duration(seconds: 1), callback);
Stream<String> stream = Stream<String>.fromFuture(fut);
Stream<String> stream = Stream<String>.fromFutures([fut1,fut2]);
Stream<int> stream = Stream<int>.fromIterable([1,2,3]);
Stream<bool> stream = Stream<bool>.value(false);

//监听
stream.listen(
    (x) => print(x),
  onError: (e)=>print(e),
  onDone: ()=>print("onDone")
);
```

##### 1.4 Isolate

```dart
loadData() async {
  // 通过spawn新建一个isolate，并绑定静态方法
  ReceivePort receivePort = ReceivePort();
  await Isolate.spawn(dataLoader, receivePort.sendPort);

  // 获取新的isolate监听port
  SendPort sendPort = await receivePort.first;
  //调用sendReceive自定义方法
  List dataList = await sendReceive(sendPort,
      'http://www.flutterj.com');
  print('dataList $dataList');
}

// isolate绑定方法
static dataLoader(SendPort sendPort) async {
  // 创建监听port，并将sendPort传给外界来调用
  ReceivePort receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);
// 监听外界调用
  await for (var msg in receivePort) {
    String requestURL = msg[0];
    SendPort callbackPort = msg[1];

    Client client = Client();
    Response response = await client.get(requestURL);
    List dataList = json.decode(response.body);
// 回调返回值给调用者
    callbackPort.send(dataList);
  }
}

// 创建自己的监听port，并且向新的isolate发送消息
Future sendReceive(SendPort sendPort, String url) {
  ReceivePort receivePort = ReceivePort();
  sendPort.send([url, receivePort.sendPort]);
// 接收到返回值， 返回给调用者
  return receivePort.first;
}
```

##### 1.5 async vs async*

```dart
Stream<int> countForOneMinute() async* {
  for (int i = 1; i <= 60; i++) {
    await Future.delayed(const Duration(seconds: 1));
    yield i;
  }
}

main() async {
  await for (int i in countForOneMinute()) {
    print(i); // prints 1 to 60, one integer per second
  }
}
```
