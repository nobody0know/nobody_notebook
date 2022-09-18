# c++下的for打印数组和vector时的简化用法

## 一维数组和vector

```cpp
unsigned scores[11]={0};
vector<int> v{1,2,3,4,5};

for(auto i: scores)   //使用范围for语句，将scores数组的内容全部打出
    cout<<i<<" ";
cout<<endl;

for(auto i: v)
    cout<<i<<" "
cout<<endl;
```

## 二维数组

```cpp
int ia[2][3]={
    {1,2,3},
    {4,5,6}
};
for(auto &row : ia)
    for(auto &col :row){
    cout<<col<<endl;
}
```
