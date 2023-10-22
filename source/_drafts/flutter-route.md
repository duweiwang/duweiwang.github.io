---
title: flutter-route
tags:
---


一、路由和Widget建立关联

1.1 MaterialApp内部会建立WidgetsApp

```dart

class MyApp extends StatelessWidget {
  MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    //程序入口
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
      routes: routerTable,
    );
  }
}
```


