# C++在一个类中调用另一个类的方法

## 调用公开成员

通过在调用的类的方法里声明被调用类的形参，即可使用被调用类的方法

```c
#include<iostream>
#include<iomanip>
#include<cmath>
using namespace std;
//----类定义----
class Point
{
    public:
        int getX()//简单函数内联写法即可
        {
            return x;
        }
        int getY()
        {
            return y;
        }
        void setPoint()
        {
            std::cin>>x>>y;
        }
    private:
        int x;
        int y;
};

class Circle
{
    //Point point;//声明被调用类的类型变量
    public:
        void Contain(Point &point);
    private:
        int x;
        int y;
        int r; 
};

//----类实现----

void Circle::Contain(Point &point)//声明被调用类的形参
{
    if(sqrt((point.getX()-x)*(point.getX()-x)+(point.getY()-y)*(point.getY()-y))<r)
        cout<<"yes"<<endl;
    else
        cout<<"no"<<endl;
}

//主函数
int main()
{
    Point point;
    Circle circle;
    circle.Contain(point);

}
```

## 调用私有成员
