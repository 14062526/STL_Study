# 迭代器和萃取
***通过迭代器将算法与数据结构（容器）分开***

## 一个简单的迭代器
```c++
template<class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator end , const T& value)
{
    while(first != last && *first!=value)
        ++first;
        return first;
}
```

```c++
template<typename T>
class List
{
    void insert_front(T value);
    void insert_end(T value);
    void display(std::ostream &os = std::cout) const;
private:
    ListItem<T>* _end;
    ListItem<T>* _front;
    long _size;
}

template<typename T>
class ListItem
{
public:
    T value() const {return _value;}
    ListItem* next() const {return _next;}
private:
    T _value;
    ListItem* _next;
}

template<class Item>
struct ListIter
{
    Item* ptr;

    ListIter(Iter* p = 0):ptr(p){}
    Item& operator*() const {return *ptr;}
    Item* operator->() const {return ptr;}

    ListIter operator++() {ptr = ptr->next(); return *this;}
    ListIter operator++(int) 
    {
        ListIter temp = *this;
        ++*this;
        return temp;
    }

    bool operator==(const ListIter& i) const {return ptr == i.ptr;}
    bool operator!=(const ListIter& i) const {return ptr != i.ptr;}
}
```

***使用迭代器***
```c++
template<typename T>
bool operator!=(const ListItem<T>& item, T n)
{
    return item.value() != n;
}

List<int> mylist;
ListIter<ListItem<int>> begin(mylist.front());
ListIter<ListItem<int>> end;
find(begin, end, 10);
```

## 萃取

***获取迭代器存取的数据类型，例如排序算法需要交换两个迭代器指向的值。***

```c++
template<class I, class T>
void swap_impl(I iter1, I iter2, T t)
{
    T tmp;
    //...
}
template<class I>
void swap(I iter1, I iter2)
{
    swap_impl(iter1, iter2, *iter1);
}
```
**缺点：当类型需要用于函数返回值时上述方式将失效**
```c++
template<class T>
struct MyIter
{
    typedef T value_type;
    T* ptr;
    MyIter(T* p=0) : ptr(p){}
    T& operator*() const {return *ptr;}
}
template <class I>
typename I::value_type func(I iter){return *iter;}
```
**缺陷：原生指针非迭代器，但应该可作为迭代器使用**

**引入萃取**
```c++
template<class I>
struct iterator_traits
{
    typedef typename I::value_type value_type;
}

template<class I>
typename iterator_traits<I>::value_type func(I iter){return *iter;}

template<class T>
struct iterator_traits<T*>
{
    typedef T value_type;
}
template<class T>
struct iterator_traits<const T*>
{
    typedef T value_type;
}
```

#### 常用迭代器的型别
```c++
template<class I>
struct iterator_traits
{
    typedef typename I::iterator_category iterator_category;
    typedef typename I::value_type value_type;
    typedef typename I::difference_type difference_type; // 迭代器之间的距离
    typedef typename I::pointer pointer;
    typedef typename I::referance referance;
}
```
### 关于iterator_category
*表示迭代器类型, 针对不同迭代器，算法对迭代器的操作不同，例如将某个迭代器前移--advanced()*

**常见迭代器类型**
- 输入迭代器(Input Iterator)
- 输出迭代器(Output Iterator)
- 前向迭代器(Farward Iterator)
- 双向迭代器(Bidirectional Iterator)
- 随机读取迭代器(Random Access Iterator)

```c++
template<class Iter, class step>
void advance_II(Iter& i, step n)
{
    while(n--) ++i;
}

template<class Iter, class step>
void advance_BI(Iter& i, step n)
{
    if(n >= 0)
        while(n--) ++i;
    else
        while(n++) --i;
}

template<class Iter, class step>
void advance_RAI(Iter& i, step n)
{
    i += n.;
}

template<class Iter, class step>
void advance(Iter& i, step n)
{
    if(is_II) advance_II(i, n);
    else if(is_BI) advance_BI(i, n);
    else advance_RAI(i, n);
}
```
定义五个类用于表示迭代器类型，通过重载机制实现不同操作
```c++
struct input_iterator_tag{};
struct output_iterator_tag{};
struct forward_iterator_tag : public input_iterator_tag{};
struct bidirectional_iterator_tag : public forward_iterator_tag{};
struct random_access_iterator_tag : public Bidirectional_iterator_tag{};
```
```c++
template<class Iter, class step>
void __advance(Iter& i, step n, input_iterator_tag)
{
    ...
}
template<class Iter, class step>
void __advance(Iter& i, step n, bidirectional_iterator_tag)
{
    ...
}
template<class Iter, class step>
void __advance(Iter& i, step n, random_access_iterator_tag)
{
    ...
}

template<class Iter, class step>
void advance(Iter& i, step n)
{
    advance(i, n, iterator_traits<iter>::iterator_category());
}

```
