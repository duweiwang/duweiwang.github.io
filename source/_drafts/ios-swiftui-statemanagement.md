---
title: SwiftUI状态管理
tags:
---

+ @State
+ @Binging
+ @ObservedObject
+ @StateObject
...



二、@Binding

1、参数传递问题

```
    //父View
    struct Parent : View{
        @State private var isHappy = false
    
        
        
    }
    
    //子View
    struct Child : View{
        @Binding var isHappy: Bool
        
        private var mIsHappy = false
        
        init(isHappy: Binding<Bool>){
            self._isHappy = isHappy
            
            //使用参数里的值,value将取到Binding里面的值
            self.mIsHappy = isHappy.value
            
            
        }
    }
```