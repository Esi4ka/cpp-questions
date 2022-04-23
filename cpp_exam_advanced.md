# Advanced

## 1

**Protected-наследование**: public- и protected-члены из родителя в наследнике становятся protected, а private-члены родителя в наследнике недоступны

**Двухуровневое наследование, примеры**

```cpp
#include <iostream>
#include <map>
#include <algorithm>

struct Granny {
    void f() { std::cout << 1; };
};

struct Mom : private Granny {
    void f(int y) { std::cout << 2; }
};

struct Son : Mom {
private:
    void g() {}
  void f(double y) {}  
};

int main() {
    Mom m; Son s;
    m.f(); // версия бабушки видна, но недоступна из-за приватного наследования
    m.Granny::f();
    s.Mom::f(4); // видно и доступно
    s.f(); // видно, но недоступно, т.к. недоступно из мамы
    s.g();
}
```

https://ravesli.com/urok-160-sokrytie-metodov-roditelskogo-klassa/

**friend**

```cpp
struct S {

    public:
    void g() {}
    int oa = 3;
    // friend int main(); эффекта не будет, сам наследник запрещает доступ к поляс
};

struct T: private S {
    void f() {}
    friend int main(); // эффект будет
};

int main() {
    T t;
    t.g();
}
```

```cpp
struct S {
    protected:
    void g() {}
    int oa = 3;
    // friend int main();
};

struct T: private S {
    void f() {}
    friend int main();
};

int main() {
    T t;
    t.g();
}
```

Для protected родительских полей с помощью friend можно получить доступ извне

## 2

**Полиморфный объект** - такой, у которого есть хотя бы один виртуальный метод. Для каждого вызова виртуальной функции компилятор должен уметь правильно выбирать версию. Для этого в runtime язык поддерживает информацию о типе (RTTI)

Каждый такой объект имеет указатель vptr на область статической памяти, которая содержит информацию о типе и создается для каждого типа - vtable.

![image-20210624153442486](/home/h4zzkr/snap/typora/39/.config/Typora/typora-user-images/image-20210624153442486.png)

## 3

Проблема ромбовидного наследования была решена введением нового типа наследования, виртуального. При таком типе создаётся одна копия родителя. Виртуальное наследование даёт увеличение размера объекта.

```cpp
struct Granny {
	int g = 0;
};
struct Mom : public virtual Granny {
	int m = 1;
};
struct Dad : public virtual Granny {
	int d = 2;
};
struct Son : public Mom, public Dad {
	int s = 3;
};

sizeof(son) = 20 без virtual;

// [mom_ptr][Mom::m][dad_ptr][Dad::d][Son::s][Granny::g ]
// указатель на базу
```

При виртуальном наследовании предку нужно знать, где его начало. Размер увеличивается



# 4

Шаблонными параметрами не обязательно должны быть типы. Шаблонным параметром может быть любой объект целочисленного типа, а также char и bool.

- Разные числа будут считаться разными типами. Типы, которые можно отдавать в шаблонные аргументы должна быть константами.

```cpp
template <int P>class S {};S<5> s; S<6> ss;
```

Шаблонным параметром м.б. другой шаблон:

```cpp
template <typename T, template <typename> typename Container> // объявление шаблона как параметра шаблона
class Stack {
    Container<T> c;
public:
    push(); pop();
}

Stack<int, std::vector> s; // std::vector - не тип, а шаблон, пока не зафиксировали парамтеры
```

**Variadic templates**

Функции, принимающие переменное количество шаблонных аргументов.

```cpp
void print() {} // для завершения рекурсии (от пустого пакета)

template <typename Head, typename... Tail> // пакет параметров шаблона (разные типы шаблонов)
void print(const Head& head, const Tail&... tail) { // пакет аргументов функции
    std::cout << head << ' ';
    print(tail...); // т.к tail - это пакет, распаковка пакета - превращение в список аргументов
}

print(1, 2, 'c', "abc")
```

```cpp
template <typename First, typename Second, typename... Types>
struct is_homogeneous {
    static const bool value = std::is_same<First, Second> && is_homogeneous<Second, Types...>::value;
};

template <typename First, typename Second>
struct is_homogeneous<First, Second> {
    static const bool value = std::is_same<First, Second>;
};
```

sizeof...(param_pack) - returns the number of elements in a [parameter pack](https://en.cppreference.com/w/cpp/language/parameter_pack).

```cpp
template<class... Types>
struct count {
    static const std::size_t value = sizeof...(Types);
};
```

## 5 - Dependent names

Возникают некоторые проблемы неоднозначности при компиляции

```cpp
template <typename T>
struct S {
    using X = T;
}

template <>
struct S<int> {
    static int X;
}

int a = 0;
template <typename T>
void f() {
    S<T>::X * a; // имя переменной или название типа?
}
```

Тело функции f можно трактовать как expression: S<int>::X * a (умножение на ноль); а также как declaration: S<T>::X* a === T* a;

По умолчанию это имя переменной (умножение). Фикс:

```cpp
typename S<T>::X* a; // T* a;
```

**Зависимые имена шаблонов**

```cpp
template <typename T>
struct Outer {
    template<typename A, typename B>
    class Inner {};
};

template<>
class Outer<int> {
    static const int Inner = 0;
};

int a = 0;

template<typename T>
void f(T x) {
    typename Outer<T>::template Inner<int,char> a;
}

int main() {
    f(3.0f);
}
```

Здесь просто typename недостаточно: без template компилятор не будет воспринимать это все как имя шаблона

https://www.it-swarm.com.ru/ru/c%2B%2B/gde-i-pochemu-ya-dolzhen-postavit-klyuchevye-slova-template-i-typename/957772073/

## 6

**std::insert_iterator** - итератора типа OutputIterator, позволяет алгоритмам, которые обычно перезаписывают элементы контейнеров, осуществлять вставку новых элементов по позиции.

```cpp
template <typename Container>
class _insert_iterator : public std::iterator<std::output_iterator_tag, void, void, void, void> {
    Container* container;
    typename Container::iterator iter;

public:
    explicit _insert_iterator(Container& x, typename Container::iterator i):
            container(&x), iter(i) {}
    
 _insert_iterator<Container>& operator=(const typename Container::value_type& val) {
        iter = container->insert(iter, val);
        ++iter;
        return * this;
    } // move аналогично

 _insert_iterator<Container>& operator*() {
        return *this;
    }

 _insert_iterator<Container>& operator++() {
        return *this;
    }

 _insert_iterator<Container> operator++(int) {
        return *this;
    }
};

template <typename Container> _insert_iterator<Container> _inserter(Container& c, typename Container::iterator i) {
    return _insert_iterator<Container>(c, i);
}

int main() {
    std::vector<int> v = {1,5,2};
    std::list<int> l = {-2, -1};
    std::copy(v.begin(), v.end(), _inserter(l, std::next(l.begin())));
    for (auto i : l)
        std::cout << i;  
}
```

## 7 istream/ostream

Запись в поток вывода (ostream), чтение из потока ввода (istream)

```cpp
template <typename T>
class istream_iterator {
    std::istream& in; // поток ввода
    T value; // текущий введенный объект
public:
    istream_iterator(std::istream& in): in(in) {
        in >> value;
    }
    istream_iterator<T>& operator++() {
        in >> value;
    }
    T& operator*() {
        return value;
    }
}
```

```cpp
// usage
istream_iterator<int> it(std::cin); // input first here
for (int i = 0; i != 5; ++i, ++it) {
    std::cout << (*it);
}

std::copy(v.begin(), v.end(), std::ostream_iterastor(std::cout));
```

Можно отдавать в стандартные алгоритмы (которые требуют итераторы).

## 8

Оператор new помимо выделения памяти под какой-то тип, еще и направляет конструкторы для него в каждую ячейку выделенной памяти.

Действия оператора new можно перегрузить, но не целиком - только ту часть, которая отвечает выделению памяти. Конструкторы будут неизбежно вызваны после выделения памяти. По стандарту оператор new принимает число - сколько нужно выделить байт.

Оператор delete сначала вызовет деструкторы, потом очистит память, перегрузке аналогично подлежит только очищение памяти

```cpp
void* operator new(size_t n);
void* operator new[](size_t n);
void operator delete(void* p);
void operator delete[](void* p);
```

**Оператор new и функция оператор new - разные вещи. Функция - выделяет память, оператор - выделяет память и вызвает конструкторы. Именно первый и первую часть второго мы перегружаем.**

Пример: у структуры S конструктор сделан приватным, тогда new S (или new S()) не скомпилируется, а operator ::operator new(sizeof(S)) (но нужно сделать reinterpret_cast от void* к S*, но вообще-то так делать не стоит) скомпилируется, и выделит память с помощью глобального new, без вызова конструктора.

**Разновидности оператора new**

- placement new:

  ```cpp
  S* ptr = reinterpret_cast<S*>(::operator new(sizeof(S)));
  new(ptr) S(value);
  ```

  Перегрузка:

  ```cpp
  void* operator new(size_t, S* ptr) {
      return ptr;
  }
  
  static void* operator new(size_t n) {} // писать внутри класса, перегрузка только для его типа
  ```

Очередная перегрузка, но с кастомными параметрами:

```cpp
void* operator new(size_t n, MyStruct) {
    ...
    return malloc(n);
}

void operator delete(void* ptr, MyStruct) {
    ...
    free(ptr);
}

S* p = new(mys) S(); // это оператор, не функция, просто с параметром
delete p; // вызов обычного
p->~S();
operator delete(p, mys); // вызов кастомного, но деструктор не вызовется

Но если получили исключение в конструкторе на этапе вызова new, то delete-функция будет вызвана с кастомным delete
```

## 9 

```cpp
#include <iostream>
#include <type_traits>

template <typename T, size_t max = 1024>
struct stack_allocator {

    using aligned = typename std::aligned_storage_t<max>;
    aligned stack{};
    aligned* ptr = &stack;

    T* allocate(size_t n) {
      aligned* rptr = ptr;
      ptr += n * sizeof(T);
      return reinterpret_cast<T*>(rptr);
    }
    
    void deallocate(T* ptr, size_t n) {
      return;
    }
    
    template <typename... Args>
    void construct(T* ptr, const Args&... args) {
        new (ptr) T(args...);
    }
    
    void destroy(T* ptr) {
        ptr->~T();
    }
};

int main() {
  auto alloc = stack_allocator<int>();
  auto p = alloc.allocate(1);
  alloc.construct(p, 5);
  std::cout << *p;
  alloc.destroy(p);
}
```



## 10

**AllocatorAware контейнер** - контейнер, в котором есть аллокатор в качестве поля, который используется во всех allocate/deallocate memory командах функций, а также с помощью которого (и allocator_traits) конструируются и уничтожаюются объекты в памяти аллокатора.

При создании нового контейнера из копии другого:

- Copy constructors of *AllocatorAwareContainer*s obtain their instances of the allocator by calling [std::allocator_traits](http://en.cppreference.com/w/cpp/memory/allocator_traits)<allocator_type>::select_on_container_copy_construction(const Alloc& a) on the allocator of the container being copied.

The only way to replace an allocator is copy-assignment, move-assignment, and swap:

- Оператор копирования заменит аллокатор контейнера только если **propagate_on_container_copy_assignment::value** is true
- Оператор перемещения переместит аллокатор контейнера только если **propagate_on_container_move_assignment::value** is true
- Swap заменит аллокатор контейнера только если **propagate_on_container_swap::value** is true

**vector allocator-awareness**

```cpp
Vector(const Vector& other) {
    ...
    alloc = std::allocator_traits<Alloc>::select_on_container_copy_construction(other.alloc);
    ...
}

Vector(Vector&& other) {
    ...
    просто мувнуть (Move constructors obtain their instances of allocators by move-constructing from the allocator belonging to the old container.)
    ...
}
```

## 11

```cpp
template <typename T>
T&& forward(typename remove_reference_t<T>& x) noexcept {
    return static_cast<T&&>(x);
}
```

![image-20210628103439451](/home/h4zzkr/snap/typora/39/.config/Typora/typora-user-images/image-20210628103439451.png)

~~remove_reference нужен, чтобы заставить пользователя явно писать тип шаблона в вызове~~

~~*В emplace приняли rvalue => & отбрасываются, Arg = int, decltype(arg) = int&&. Теперь в forward передается lvalue => срабатывает правило и без явного указания параметра шаблона в forward T = int&. Без remove_reference decltype(x) = int&, но T = int& => возвращаемый тип int& => потрачено*~~

![image-20210629065318355](/home/h4zzkr/snap/typora/39/.config/Typora/typora-user-images/image-20210629065318355.png)

С T&& при передаче rvalue в f: T = F, auto t = forward<S>(value), где value - lvalue. В самом forward происходит инстанцирование:

```cpp
S&& forward(S&& value) {
    return static_cast<S&&>(value);
}
```

и мы никак не сможем принять lvalue, хотя он изначально был rvalue

## 12

**copy elision**

```cpp
BigInt operator+ (const& BigInt lhs, const& BigInt rhs) {
    BigInt copy = lhs;
    // copy += rhs;
    // return copy; // with rvo
    return copy += rhs; // copy elision
}

BigInt newint = a + b;
```

Если создаем временный объект и сразу же инициализируем им что-то, то срабатывает оптимизация copy elision. Copy конструктор вызван в 2, в 3 - при возвращении результата, в 6 при присваивании конструктор копирования не вызывается. Справа от = после выполнения создался временный объект, которым инициализируется левый операнд. Тогда компилятор не создает еще один временный объект для присваивания внутри оператора, а сразу считает получившийся нужным

**Return Value Optimization(RVO)** - если компилятор понимает, что в методе создается локальный
объект и он же возвращается, то компилятор выделит память в том месте, где ожидается воз-
вращание результата функции, таким образом убирая лишнее копирование (по сути, инициализируемый объект - возвращенный объект)

```cpp
T foo(int n){
   T t(n); // (1)
   /* some tweaking of t */
   return t;
}
 
int main() {
   T v = foo(100);
}
конструктор (1) сразу конструирует T на месте v из main. Обращение к t в foo — по сути, обращение к v.
```

**Виды оптимизаций**

```cpp
T f() {
    return T(); // copy elision, rvo
}
T g() {
    T t;
    return t; // мб copy elision, but rvo always
}
void foo(C c); // copy elision when temp object passed by value
```



## 15

Функции могут давать или не давать гарантию безопасности относительно исключений. Кон-
тейнеры могут перестать работать корректно вследствие вызова исключений и вызвать UB.

**Базовая** (basic) гарантия: объект останется в валидном состоянии после вызова исключе-
ний
**Сильная** (strong) гарантия: объект останется в исходном состоянии после выхова исклю-
чений.

Почти все STL-библиотечные функции/контейнеры дают сильную гарантию безопасности.

**map/unordered**

Проблема, если исключение вылетает из компаратора. Map ничего не сможет с этим сделать

## 16

**Проблема циклических shared_ptr**

Допустим мы реализуем двоичное дерево и в какой-то момент хотим удалить поддерево этого дерева. Логично предположить, что все указатели должны удалиться, однако из-за того что внутри поддерева сын указывает на родителя, а родитель на сына, будут оставаться объекты, указывающие на вершину - вершина не сможет удалится, и во всем поддереве, произойдет memory leak.

**weak_ptr**

Хранит указатель на объект, но не владеет им, просто наблюдает (не может как бы то ни было изменять объект - разыменовать нельзя, при его удалении ничего не происходит). Поддерживаются два метода - можно спросить, сколько shared_ptr указывают на объект (жив ли еще объект), а также можно создавать shared_ptr на объект указателя weak_ptr.

**Решение проблемы** - делаем связь сын-родитель через weak_ptr в дереве. Тогда при уничтожении связи с подкорнем, на эту вершину будет указывать два weak_ptr и не более - вершина удалится. И так умрет все поддерево.

Т.е. в цикл. зависимости х.б один указатель должен быть weak.

```cpp
struct ControlBlock {
    size_t shared_count = 0 ;
    size_t weak_count = 0;
    T* object;

    template <typename... Args>
    ControlBlock(size_t count, Args&&... args): shared_count(count) {
        object = new T(std::forward<Args>(args)...);
    }
};

template <typename T>
class weak_ptr;

template <typename T>
class shared_ptr {
    T* ptr = nullptr;
    ControlBlock<T>* cptr = nullptr;

    shared_ptr(ControlBlock<T>* cptr): cptr(cptr), ptr(cptr->object) {}
    
    template <typename Y, typename... Args>
    friend shared_ptr<Y> make_shared(Args&&... args);

    friend weak_ptr<T>;

public:
    shared_ptr() {}
    shared_ptr(T* ptr): ptr(ptr) {
        cptr = new ControlBlock{1, 0, ptr};
        // shared_from_this
        // if constexpr (std::is_base_of_v<enable_shared_form_this<T>, T>)
        //     ptr->wptr = *this;
    }

    shared_ptr(const shared_ptr<T>& other): ptr(other.ptr), cptr(other.cptr) {
        ++cptr->shared_count;
    }

    shared_ptr(shared_ptr<T>&& other): ptr(other.ptr), cptr(other.cptr) {}

    T& operator*() {
        return *ptr;
    }

    ~shared_ptr() {
        if (!cptr) return;
        --cptr->shared_count;
        if (cptr->shared_count == 0) {
            delete ptr;
            if (cptr->weak_count == 0)
                delete cptr;
        }
    }
};

template <typename T>
class weak_ptr {
    ControlBlock<T>* cptr = nullptr;
public:
    weak_ptr(const shared_ptr<T>& ptr): cptr(ptr.cptr) {
        ++cptr->weak_count;
    }

    bool expired() const {
        return cptr->shared_count == 0;
    }

    shared_ptr<T> lock() const {
        return shared_ptr<T>(cptr);
    }

    ~weak_ptr() {
        if (!cptr) return;
        --cptr->weak_count;
        if (cptr->weak_count == 0 && cptr->shared_count == 0)
            delete cptr;
    }
};
```

```cpp
template <typename T, typename... Args>
shared_ptr<T> make_shared(Args&&... args) {
    auto block = new ControlBlock<T>(1, std::forward<Args>(args)...);
    return shared_ptr<T>(block);
}
```



## 17

**enable_shared_from_this**

Кейс: вернуть в методе указатель this. Но мы не хотим работать с C-style pointers. Классы, для которых хотим поддерживать возвращение this как shared_ptr, должны публично наследоваться от enable_shared_from_this<YourClassName>. Теперь когда хотим this, нужно писать return shared_from_this().

```cpp
template <typename T>
struct ControlBlock {
    size_t shared_count = 0 ;
    size_t weak_count = 0;
    T* object;

    template <typename... Args>
    ControlBlock(size_t count, Args&&... args): shared_count(count) {
        object = new T(std::forward<Args>(args)...);
    }
};

template <typename T>
class enable_shared_from_this;

template <typename T>
class weak_ptr;

template <typename T>
class shared_ptr {
    T* ptr = nullptr;
    ControlBlock<T>* cptr = nullptr;

    shared_ptr(ControlBlock<T>* cptr): cptr(cptr), ptr(cptr->object) {
        if constexpr (std::is_base_of_v<enable_shared_from_this<T>, T>)
            ptr->wptr = *this;
    }
    
    template <typename Y, typename... Args>
    friend shared_ptr<Y> make_shared(Args&&... args);

    friend weak_ptr<T>;

public:
    shared_ptr() {}
    shared_ptr(T* ptr): ptr(ptr) {
        cptr = new ControlBlock{1, 0, ptr};
        // shared_from_this
        // if constexpr (std::is_base_of_v<enable_shared_form_this<T>, T>)
        //     ptr->wptr = *this;
    }

    shared_ptr(const shared_ptr<T>& other): ptr(other.ptr), cptr(other.cptr) {
        ++cptr->shared_count;
    }

    shared_ptr(shared_ptr<T>&& other): ptr(other.ptr), cptr(other.cptr) {}

    T& operator*() {
        return *ptr;
    }

    ~shared_ptr() {
        if (!cptr) return;
        --cptr->shared_count;
        if (cptr->shared_count == 0) {
            delete ptr;
            if (cptr->weak_count == 0)
                delete cptr;
        }
    }
};

template <typename T>
class weak_ptr {
    ControlBlock<T>* cptr = nullptr;
public:
    weak_ptr() {}
    weak_ptr(const shared_ptr<T>& ptr): cptr(ptr.cptr) {
        ++cptr->weak_count;
    }

    bool expired() const {
        return cptr->shared_count == 0;
    }

    shared_ptr<T> lock() const {
        return shared_ptr<T>(cptr);
    }

    ~weak_ptr() {
        if (!cptr) return;
        --cptr->weak_count;
        if (cptr->weak_count == 0 && cptr->shared_count == 0)
            delete cptr;
    }
};

template <typename T, typename... Args>
shared_ptr<T> make_shared(Args&&... args) {
    auto block = new ControlBlock<T>(1, std::forward<Args>(args)...);
    return shared_ptr<T>(block);
}

template <typename T>
class enable_shared_from_this {
public:
    weak_ptr<T> wptr;

protected:
    shared_ptr<T> shared_from_this() const {
        return wptr.lock();
    }
};

class S: public enable_shared_from_this<S> {
    public:
    int a = 32;
    shared_ptr<S> foo() {
        return shared_from_this();
    }
};

int main() {
    auto ptr = make_shared<S>();
    auto thisptr = (*ptr).foo();
    std::cout << (*thisptr).a;
}
```

Конструктора нет, инициализируем в shared_ptr в конструкторе от ptr-а.

## 18

```cpp
auto f = [](int x, int y) { return x < y; }
sizeof(f); // 1 байт, пустой функциональный объект (класс с оператором круглых скобок)
int a = 1;
auto f = [a](int x, int y) { return x < y; }
sizeof(f); // 4 байт, в поле "класса" добавили поле
```

Это объясняет, почему при const операторе () мы можем менять значения по ссылке, по факту они - поля класса, а их менять изнутри можно (const справа, не слева)

```cpp
auto ff = f; // copy constructor - копирует все поля
auto ff = std::move(f);; // move constructor - мувает все поля
```

оператор присваивания не генерируется

**захват полей класса, захват this**

```cpp
struct S {
    int a = 1;
    void foo() {
        // вообще говоря, захватываются только локальные переменные
        auto f = [](int x, int y) { return a; } // CE
        auto f = [a](int x, int y) { return a; } // CE
        
        auto f = [this](int x, int y) { return a; } // теперь все поля доступны
    }
}
```

**capture with initialization, default capture**

```cpp
auto f = [b = a](int x) { return b; } // завести поле b, инициализировать через a
auto f = [s = std::move(s)](int x) { ... } // можно мувать объект в лямбда функци
```

**захват всех локальных переменных**

```cpp
auto f = [=](int x) { ... } // захват по значению, bad code-style
auto f = [&](int x) { ... } // захват по ссылке, еще хуже
```

плохо, потому что можем получить UB, когда, например, захваченный this умрет

```cpp
auto f = [=](auto x) { ... } // шаблонная лямбда функция, будут подставляться нужные вещи
```

## 19

**std::function**

Можно хранить любой объект с оператором () от нужных нам типов.

```cpp
std::function<bool(int,int)> f;
```

Можно инициализировать callable объектами (указатель на функцию, ваш тип, лямбда функция)

## 20

```cpp

template <size_t D, size_t N>
struct isMore {
    static const bool value = D > N;
};

template <size_t D, size_t N, bool isMoreThan>
struct isPrimeHelper {
    static const bool value = (N % D == 0)? false : isPrimeHelper<D+1, N, isMore<D*D, N>::value >::value;
};

template <size_t D, size_t N>
struct isPrimeHelper<D, N, true> {
    static const bool value = true;
};

template <size_t N>
struct isPrime {
    static const bool value = isPrimeHelper<2, N, false>::value;
};

template <>
struct isPrime<2> {
    static const bool value = 1;
};

```



## 21

**has method - check of method presence in a class**

Пусть T - тип у которого проверяем наличие метода, Args - аргументы от которых должен быть проверяемый метод

```cpp
namespace has_method {
    template <typename T>
    T&& declval() noexcept;

    template <typename T, typename... Args>
    struct has_method_name {
    private:

        template <typename TT, typename... AArgs>
        static auto f(int) -> decltype(declval<TT>().name(declval<AArgs>()...), int());
        
        template <typename...>
        static char f(...);

    public:
        static const bool value = std::is_same_v<decltype(f<T, Args...>(0)), int>;
    };

    template <typename T, typename... Args>
    const bool has_method_name_v = has_method_name<T, Args...>::value;
}
```

- происходит инстанцирование шаблонных параметров при инстанцировании класса, поэтому вызывать функцию f от тех же параметров что и сама структура нельзя - SFINAE не сработает, так как подстановка уже произошла. Выход - у функции должны быть свои шаблонные параметры

- проблема: если нет конструктора по умолчанию, то false

Решение последней проблемы - функция declval() - используется когда нужен объект
типа T, но мы не знаем есть ли конструктор по умолчанию. Ее не надо реализовывать -
она не предназчначена для вызова непосредственно в констекстве выполнения, достаточно
для обращения к себе под unevaluated context (под decltype, size, noexcept и т.д, в остальных
случаях нельзя). && дает преимущство в виде возможности работать с типами у которых нет тела (incomplete types - ссылку на такие создать можно) и с шаблонными типами, которые не придется ин-
станцировать до этого.

## 22

```cpp
namespace is_constructible {
    template <typename T>
    T&& declval() noexcept;

    template <typename T, typename... Args>
    struct has_method_name {
    private:

        template <typename TT, typename... AArgs>
        static auto f(int) -> decltype(declval<TT>(declval<AArgs>()...), int());
        // , int() - чтобы возвращаемый тип был как надо, потому что компилятор все ранво вычислит выражение слева от запятой

        template <typename...>
        static char f(...);

    public:
        static const bool value = std::is_same_v<decltype(f<T, Args...>(0)), int>;
    };
}

template <typename T, typename... Args>
const bool is_constructible_v = is_constructible<T, Args...>::value;

template <typename T>
const bool is_copy_constructible_v = is_constructible<T, const T&>::value;

template <typename T>
const bool is_move_constructible_v = is_constructible<T, T&&>::value;
```

## 23

Почему нельзя наивно выразить is_nothrow_move_constructable через std::if_move_constructible_v &&
noexcept(T(std::declval<T>())) ? 

Потому что правило игнорирования второго аргумента в конъюнции при ложном первом аргументе - это рантайм-действие, в Compile-Time все равно все проверится, и упадет с CE (если у типа нет move конструктора, то noexcept(T(...)) упадет)

```cpp
namespace is_nothrow_constructible {
    template <typename T>
    T&& declval() noexcept;

    template <typename T, T v>
    struct integral_constant {
        static const T value = v;
    };

    struct true_type : integral_constant<bool, true> {};
    struct false_type : integral_constant<bool, false> {};

    template <typename T>
    struct is_nothrow_move_constructible {
    private:
        template <typename TT>
        static auto f(int) -> integral_constant<bool, noexcept(TT( declval<TT>() ))>;

        template <typename TT>
        static auto f(...) -> false_type;
    public:
        static const bool value = decltype(f<T>(0))::value;
    };

    template <typename T>
    const bool is_nothrow_move_constructible_v = is_nothrow_move_constructible<T>::value;
}

struct S {
  S(S&& other) noexcept {
    std::cout << 2;
  }
};

int main() {
  std::cout << is_nothrow_move_constructible_v<std::unique_ptr<int>>;
  std::cout << is_nothrow_move_constructible_v<S>;
  return 0;
}
```

## 24

**is_base_of**

```cpp
template <typename B, typename D>
struct is_base_of {
private:
    template <typename T>
    static auto f(T*) -> std::true_type;

    template <typename...>
    static auto f(...) -> std::false_type;

    template <typename BB, typename DD>
    static auto check_private(int) -> decltype(f<BB>(std::declval<DD*>()));
    
    template <typename...>
    static auto check_private(...) -> std::true_type;

public:
    static const bool value = std::integral_constant< bool, std::is_class_v<B> && std::is_class_v<D> && decltype(check_private<B,D>(0))::value >::value;
};
```

## 25

```cpp
template <typename T>
struct is_polymorphic {
private:
    template <typename U>
    static std::true_type f( decltype(dynamic_cast<const void*>(std::declval<U*>()), int()) );

    template <typename...>
    static std::false_type f(...);

public:
    static const bool value = decltype(f<T>(0))::value;
};
```



## 26

```cpp
template <size_t N>
struct index_sequence {};

template <typename T, size_t N>
struct push_back{};

template <typename N, typename... Ints>
struct push_back<index_sequence<Ints...>, N> {
  using type = index_sequence<Ints..., N>;
};

template <size_t N>
struct make_index_sequence_s {
  using type = typename push_back<typename make_index_sequence_s<N-1>, N-1>::type;
};

template <>
struct make_index_sequence_s<0> {
  using type = index_sequence<>;
};

template <size_t N>
using make_index_sequence = typename make_index_sequence_s<N>::type;
// 0, 1, 2, ..., N-1 
```

## 27

**Detecting fields count**

```cpp
struct ubiq {
    template <typename T>
    operator T();
}

template <int N>
using ubiq_constructor = ubiq;

template <typename T, size_t I0, size_t... I>
auto detect_fields_count(std::index_sequence<I0, I...>)
	-> decltype( T{ ubiq_constructor<I0>{}, ubiq_constructor<I>{}... }, size_t() ) {
    return sizeof...(I) + 1;
}

template <typename T, size_t... I>
size_t detect_fields_count(std::index_sequence<I...>) {
    return detect_fields_count<T>(std::make_index_sequence<sizeof...(I) - 1>{});
}

std::cout << detect_fields_count<S>(make_index_sequence<100>{});
```