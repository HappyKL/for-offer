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



# stack

```c++
stack<int> S;

S.push(int val);
S.pop();
S.top();


S.size();
S.empty();
```

# queue

```c++
queue<int> Q;

Q.push(int val);
int x = Q.front();
Q.pop();

Q.empty();
Q.size();
```



# vector

```c++
vector<int> vec;

//添加元素
vec.push_back(a);//向尾部加元素
vec.insert(vec.begin()+i,a);//向第i个元素前加元素

//删除元素
vec.erase(vec.begin()+i,vec.end()-j);//删除第i个元素至倒数第j个元素
vec.erase(vec.begin()+i);

vec.clean();//清空元素

vec.size();//元素数量

vec.empty();//判空

vec[i]
```



# priority_queue

```c++
priority_queue<int> h;
h.push();
h.top();
h.pop();


priority_queue<int,vector<int>,greater<int>> min;
priority_queue<int,vector<int>,less<int>> max;
```

