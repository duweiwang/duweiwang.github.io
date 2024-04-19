---
title: flutter-tips-001
tags:
---


一、使用常量：
```dart
// Using SizedBox (Before)
SizedBox(
  width: 120.0,
  height: 40.0,
  child: ElevatedButton(
    onPressed: () {
      // Button logic here
    },
    child: Text('Click Me'),
  ),
);

// Using Compile-Time Constants (After)
const double buttonWidth = 120.0;
const double buttonHeight = 40.0;

SizedBox(
  width: buttonWidth,
  height: buttonHeight,
  child: ElevatedButton(
    onPressed: () {
      // Button logic here
    },
    child: Text('Click Me'),
  ),
);
```