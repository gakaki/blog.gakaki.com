---
title: Swift和NodeJS的SimdJSON测试
date: 2019-12-14 10:00
categories:
- iOS
tags:
- iOS
- Node
---

#### Swift

测试Demo为一个200行的JSON，Swift的Codable解码，循环测试1000次。
结果比Swift官方要快10到20%。原理使用了CPU SIMD特性，相当于并行解码了。真机iPhone测试成功。
simd 440ms对apple原生 660ms 相差200ms
[simdZippyJSONBenchmark GitHub](https://github.com/gakaki/myIOSDemo/tree/master/simdZippyJSONBenchmark)
![2019_blog_swift_simdjson](https://midu.studio515.cn/blog/2019/2019_blog_swift_simdjson.png)

#### NodeJS
```Typescript
   npm install simdjson
   var parsedJSON = simdjson.parse(jsonString);
```
Node的测试结果大跌眼镜，还没有原生JSON来的快，看来小文件上优势不大，怕是和C++交换上损失了吧。
1000次循环 原生34ms对simd 120ms 
[NodeJS SimdJSON测试 GitHub](https://github.com/gakaki/myJSDemo/blob/master/node_simdjson/index.js)
![2019_blog_swift_simdjson](https://midu.studio515.cn/blog/2019/2019_blog_node_simdjson.png)



