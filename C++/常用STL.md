# map

```c++
//定义
map<int,int> m;
//插入
m[1] = 4;
//判断是否有某个元素
m.count(key); //如果没有返回0，有返回1

//遍历
for (map<int,int>::iterator it=m.begin(); it!=m.end(); ++it)
    cout << it->first << " => " << it->second << '\n';
```

