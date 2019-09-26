# C++静态成员函数访问非静态成员的几种方法

## 回调函数在C中的使用

回调函数是指 使用者自己定义一个函数，实现这个函数的程序内容，然后把这个函数（入口地址）作为参数传入别人（或系统）的函数中，由别人（或系统）的函数在运行时来调用的函数。函数是你实现的，但由别人（或系统）的函数在运行时通过参数传递的方式调用，这就是所谓的回调函数。简单来说，就是由别人的函数运行期间来回调你实现的函数。

在C语言中的调用非常简单，比如在libevent中的使用如下：
```C
static void timeout_cb(evutil_socket_t fd, short event, void * arg) {
    //! do what you want to do
}

int main() {
    //! init event base

    //! create a event
    struct event* ev = event_new(base, -1, EV_PERSIST, timeout_cb /*this is a callback*/ , "");

    //! add the event above

    //! clear event base
}
```

在C语言中，只需要将函数的指针传递过去即可。

## 回调函数在C++中的使用

C++中的回调函数和C中的回调函数有点儿不太一样，为什么呢？因为，在C++中有`this`指针的概念，默认情况下，使用C++下的回调函数的时候，是会传递`this`指针的，但是在C中没有`this`指针，那么C语言调用C++的在某种情况下就会出错，甚至是不可能编译过去的。如果将成员函数定义成静态的，那么静态的成员方法将不能访问非静态的成员变量。

### 方法一

针对`libevent`的函数特性定义。传递实例的内存地址并且做强制性转换。

```C++
class A {
public:
    void setTimeout_cb(){
        //! this指针是为了在静态函数中可以访问成员变量
       struct event *ev = event_new(base, -1, EV_PERSIST, timeout_cb, this);
    }

private:
    static void timeout_cb(evutil_socket_t fd, short event, void * arg){
        A *a = (A *)arg;
        //! 访问非静态变量
        a->m_count++;

        //! do what you want to do
    }

private:
    int m_count = 0;
}

```

### 方法二

使用全局变量地址

```C++
A a;

class A {
public:
    void setTimeout_cb(){
       struct event *ev = event_new(base, -1, EV_PERSIST, timeout_cb, "");
    }

private:
    static void timeout_cb(evutil_socket_t fd, short event, void * arg){
        //! 访问非静态变量
        a.m_count++;

        //! do what you want to do
    }

private:
    int m_count = 0;
}

```
该方法并不友好，不推荐使用。

### 方法三

跟方法一基本类似，只是在形参中传递的不是`void*`类型的，而是实例对象地址。


```C++
class A {
public:
    void setTimeout_cb(){
        //! this指针是为了在静态函数中可以访问成员变量
       struct event *ev = event_new(base, -1, EV_PERSIST, timeout_cb, this);
    }

private:
    static void timeout_cb(evutil_socket_t fd, short event, A *a){
        //! 访问非静态变量
        a->m_count++;

        //! do what you want to do
    }

private:
    int m_count = 0;
}

```
在`libevent`使用不太方便，所以只是作为说明而已。
