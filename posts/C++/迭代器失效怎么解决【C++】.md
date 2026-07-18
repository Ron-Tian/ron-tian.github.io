---
id: cpp-iterator-invalidation
title: "C++ 迭代器失效怎么解决"
date: 2026-07-18
tags: C++,STL
excerpt: "序列式容器和关联式容器迭代器失效的不同处理方式，erase 返回下一个有效迭代器。"
cover: linear-gradient(135deg, #fbc2eb 0%, #a6c1ee 100%)
readingTime: 1
type: post
---

（1）序列式容器（数组式容器如array、vector、queue）： 使用erase方法可以返回下一个有效的iterator
 // 序列式容器

```cpp
 vector<int> vecTemp = {1,2,3};
 for (auto iter = vecTemp.begin(); iter != vecTemp.end(); ++iter)
 {
 	if (*iter == 2)
 	{
 		iter = vecTemp.erase(iter);
 	}
 }
```

（2）关联式容器：使用erase方法可以返回下一个有效的iterator或者删除之后再次++
 // 关联式容器

```cpp
 map<int, int> mapTemp = { {1,1},{2,2 },{3,3} };
 for (auto iter = mapTemp.begin(); iter != mapTemp.end(); ++iter)
 {
 	if (iter->first == 2)
 	{
 	iter = mapTemp.erase(iter);
 	// OR mapTemp.erase(iter++);
 	}
 }
```
