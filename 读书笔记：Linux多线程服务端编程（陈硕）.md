# 00 背景
看\<\<Linux多线程服务端编程\>\>过程中的读书笔记，同时将实践代码放入https://github.com/jeasonchan/CppExercise 相关target中。


# 01 以线程安全的方式管理对象生命周期

## 01.01 析构函数遇到多线程

### 01.01.01 线程安全的定义
线程安全的类：

1. 多线程访问时，表现出正确的行为
2. 无论操作系统如何调度线程，被调度的线程按照什么样的顺序执行
3. 调用端无需额外的同步或者其他协调动作

因此，C++标准库的里的class大部分都不是线程安全的，比如strung、map等

### 01.01.02 mutex和lock_guard
17之前常用的互斥锁和自动管理锁的声明周期的封装lock_guard，比如，为了使类的方法互斥，刚进入方法体时，使用lock_guard锁住类成员mutex，离开方法体时，对象自动析构时再解锁，实现Java里synchronized method的功能。



### 01.01.03 线程安全的计数器类

```cpp
#include <mutex>
#include <memory>
#include <thread>
#include <iostream>
#include <chrono>

// 匿名方法区，方法区内的符号天然对本编译单元内可见，且由于匿名性，可以等效认为对其他编译单元不可见
namespace
{
    class unsafe_counter
    {
    private:
        int times;

    public:
        unsafe_counter(/* args */) : times(0)
        {
        }

        void increase()
        {
            ++(this->times);
        }

        int getTimes()
        {
            return this->times;
        }
    };

    class safe_counter
    {

    private:
        long times; //2147483647
        std::mutex lock;

    public:
        safe_counter() : times(0) {}

        void increase()
        {
            std::lock_guard<std::mutex> guard(this->lock);
            ++(this->times);
        }

        int getTimes()
        {
            return this->times;
        }
    };

}

int main()
{
    using namespace std;

    unsafe_counter counter01;
    thread counter01_thread01([&counter01]() {
        for (int i = 0; i < 10000000; ++i)
        {
            counter01.increase();
        }
    });

    thread counter01_thread02([&counter01]() {
        for (int i = 0; i < 10000000; ++i)
        {
            counter01.increase();
        }
    });

    thread counter01_thread03([&counter01]() {
        for (int i = 0; i < 10000000; ++i)
        {
            counter01.increase();
        }
    });

    // 必须用detach，否则主线程会尝试析构线程，导致异常，这是CPP强制程序员有效进行资源管理
    counter01_thread01.detach();
    counter01_thread02.detach();
    counter01_thread03.detach();

    safe_counter counter02;
    thread counter02_thread01([&counter02]() {
        for (int i = 0; i < 10000000; ++i)
        {
            counter02.increase();
        }
    });

    thread counter02_thread02([&counter02]() {
        for (int i = 0; i < 10000000; ++i)
        {
            counter02.increase();
        }
    });

    thread counter02_thread03([&counter02]() {
        for (int i = 0; i < 10000000; ++i)
        {
            counter02.increase();
        }
    });
    counter02_thread01.detach();
    counter02_thread02.detach();
    counter02_thread03.detach();

    std::this_thread::sleep_for(std::chrono::seconds(5));

    cout << counter01.getTimes() << endl;
    cout << counter02.getTimes() << endl;

    void (safe_counter::*fun)() = &safe_counter::increase;
    thread thread222(&safe_counter::increase, &counter02);
    (counter02.*fun)();

    return 0;
}

```

但是，多线程间共享一个指向堆上该类的指针，有的线程想访问，有的线程想delete，还是县不安全，这就是堆上对象的多线程析构问题。


## 01.02 多线程的构造很简单
前面讲堆中对象，在多线程场景中很难判断判断析构时机（其实，share_ptr和weak_ptr就可以解决，但是作者还是要讲一下如何一步步演化到引用计数这个解决方案中的）。但是，多线程的对象创建就很不算麻烦。

多线程构造的隐患：

```cpp
class Obj{
    Obj(Observer & looker){
        looker.doSth(this);//对外暴露了指针
    }

}

```
这个构造函数不是线程安全的，在多线程中，对象还没构造完，就对外暴露了this指针，即使放在最后一行也不行，因为别人可能把你当作父类。

二段式构造，即 构造函数+initialize() 有时会是好 办法，这虽然不符合 C++ 教条，但是多线程下别无选择。调用方靠 initialize() 的返回值来判断对象是否构造成功， 这能简化错误处理。同时，initialize必然也是要保证线程安全。


## 01.03 堆中对象销毁太难
也就是mutex成员变量确实能保证方法之间的互斥，但是，由于析构函数的操作，析构函数甚至能释放mutex成员对象，将互斥对象破坏掉，比如这种情况：

1. thread A 刚检查完不为null，准备开始尝试进行成员函数调用x->doSth()

2. thread B立刻delete x并将x=nullptr

这时候，x->doSth()可能发生空指针 或者 core dump 访问不存在的对象实例


### 01.03.02 线程安全的swap和operator =
之前说了利用成员函数并不能保证析构的安全性，但是，在多线程中，swap和operaor = 这种同类型之间数据加交互，同样值得我们用线程安全的写法：


## 01.05 原始指针有何不妥
在多线程的程序中，原始指针无法知道堆中对象的存活状态，各线程很难确认何时进行delete操作。正如前面提到的，一个线程delete的时机不对，就造成另一个线程里的指针变为悬空指针。

要想安全的删除对象，就是要做到在对象完全没人用的情况下，再将其删除，Java里分析对象没人用的方法，从引用技术，逐渐发展到可达性分析（比如，经典的三色标记算法）。

## 01.06 share_ptr/weak_ptr
scoped_ptr/shared_ptr/weak_ptr都是值语意，要么是栈上对象，或是其他对象的直接数据成员，或是标准库容器里的元素。几乎不会有下面这种用法：

1. new一个智能指针对象
2. 智能指针对象的引用

因为，从上面两个方法得到的智能指针对象再去做一些操作，极有可能无法使引用计数增加，从而失去了智能指针原本的意义：使用引用计数来管理对象的生命周期

注意，如果这几种智能指针是对象x的数据成员，而它的模板参数T是个incomplete类型，那么x的析构函数不能是默认的或内联的，必须在.cpp文件里边显式定义，否则会有编译错或运行错。**？？？？？？？？不明白，说是下文有解释**


## 01.08 智能指针用于观察者模式
代码实践：

```cpp

#include <iostream>
#include <memory>
#include <mutex>
#include <vector>

class Observer {
public:
  virtual void notify() = 0;
};

class Observable {
private:
  mutable std::mutex _mutex;

  // 如果list使用share_ptr，别观察者使用持有观察者的智能指针
  //被观察者就会直接影响观察者的生命周期，让观察者始终无法释放。
  // 所以，不想延长对象的生命周期时，就用share_ptr接受指针
  std::vector<std::weak_ptr<Observer>> observerList;

  using Iterator = std::vector<std::weak_ptr<Observer>>::iterator;

public:
  void register_(std::weak_ptr<Observer> x) { 
    std::lock_guard<std::mutex> methodLock(this->_mutex);
    this->observerList.push_back(x);
  }

  void notifyAllObservers() {
    //模板类型不能自动推导？？？？？lock_guard模板参数还要写全？？？？
    std::lock_guard<std::mutex> methodLock(this->_mutex);
    Iterator it = this->observerList.begin();

    while (it != this->observerList.end()) {
      std::shared_ptr<Observer> obj(it->lock());
      if (obj) {
        obj->notify();
        ++it;
      } else {
        //对象已经销毁，直接从队列中删除对象
        this->observerList.erase(it);
      }
    }
  }
};

int main(int argc, char *argv[]) {
  // do nothing
  return 0;
}
```


潜在的问题：

1. 侵入性，要求观察者必须用弱引用传入

2. 内部方法的锁竞争，因为，notifyAllObservers会逐个调用观察者的方法，万一某个观察者的方法十分耗时，那其他观察者进行register_操作时就会一直等待，性能大大降低

3. 死锁，某个观察者内部又生成了一个新的观察者，让这个新观察者注册观察，由于本例使用了普通的std::mutex，是不可重入的，这种情况会发生死锁；如果单纯的换曾可重入锁，那么vector就会在遍历的过程中发生改变，迭代器会失效，所以，换成可重入锁的同时还要将vector换成std::list。

## 01.09 再论share_ptr的线程安全




# 02 线程同步精要




# 03 多线程服务器的适用场景与常用编程模型