# Intermediate

## 1

```cpp
for (int i = 0; i < 300; ++i)
    std::cout << i << ' ' << i * 12345678 << std::endl;
```

с -O2 будет infinite loop. Компилятор оптимизирует исходя из условия, что выражение корректно. Если корректен, то предполагается, что никогда не будет переполнения (i < 174 - когда нет переполнения).  i < 300 всегда истинно и компилятор заменяет это на true

**-Werror** - трактовать ворнинги как ошибки компиляции

## 2

***Operator precedence*** specifies the order in which an expression should be *parsed*. It is similar to the use of operators in mathematics and tells us which operand that belongs ("glues") to which operator. In an expression such as `a + b * c`, operator precedence specifies that the expression must be treated as equivalent to `a + (b * c)`.

***Order of evaluation*** specifies the order in which an expression should be *executed*. That is the order in which the functions `a()`, `b()` and `c()` are executed. It is obvious that `b()` and `c()` must be called before `b() * c()` can be calculated, but it is less obvious that all three functions `a()`, `b()` and `c()` might be called in advance and that a() might be called first. The order in which they are called/executed is the order of evaluation.

**Unspecified behaviour**

Термин означает, что поведение валидного C++ кода не определено cтандартом и зависит от реализации. Пример: порядок вычисления значений  аргументов функции определяется компилятором, но нигде нет описания, как именно. Стандарт говорит нам: это особенности поведения, которые не  зафиксированы нигде, следовательно, на них нельзя полагаться. Поэтому и  поведение вашего кода не должно зависеть от этих особенностей.

**Undefined behaviour**

```cpp
int bar() {
    int* p = nullptr;
    return *p;        // Unconditional UB
}
```



## 3

**static_cast<...>(...)** - преобразование типов данных, присутствует проверка его выполнения компилятором во время компиляции (однако, только на основе compile-time данных)

```cpp
CE - два класса с user-defined конверсиями, которые не приводятся друг к другу
UB - long к int
```



**const_cast<...>(...)**  - на случай, если хотим нарушить константность (reinterpret_cast ее все же уважает)

```cpp
const int x = 1;
int& x = const_cast<int&>(x); // без каста это было бы CE
x = 2; // OK, but UB
```

Применение:

```cpp
f (int& x);
f (const int& x);

хотим запустить f() для константного инта от обычного
```





**reinterpret_cast<...>(...)**  - позволяет интерпретировать значение в смысле набора байтов как другой тип данных

```cpp
int x = 3;
double* d = reinterpret_cast<double*>(&x);
// теперь смотрим на x как на double
```

```cpp
Пример CE, RC уважает константность
const int x = 3;
int* d = reinterpret_cast<int*>(&x);
```

```cpp
Пример UB
int* x = new int(5);
std::cout << reinterpret_cast<int&>(x); // каст указателя к ссылке
```

Применение: представить объект как последовательность байтов (uint8_t)

**c-style** - пытается сделать все, чтобы получилось скастовать: const, static, static+const, reinterpret, reinterpret+const.

## 4

**Статические переменные** - имеют статическое время жизни

Сохраняет свое значение даже после выхода из блока, в котором она  определена. То есть она создается (и инициализируется) только один раз, а затем сохраняется на протяжении выполнения всей программы. Кроме того, выходя из области видимости, она не уничтожается.

```cpp
int incrementor() {
    static int x = 0;
    ++x;
    std::cout << x << '\n';
}

incrementor(); incrementor(); incrementor();
>> 1 2 3
```

**Статические поля и методы класса**

Хранятся в статической области памяти, определяются и инициализируются один раз и общие для всех объектов класса

Применение: подсчет количества объектов класса (поле), вспомогательные функции (методы)

```cpp
class Singleton {
    static Singleton* obj;
    Singleton() {}
public:
    static Singleton& get() {
        if (obj) return *obj;
        obj = new Singleton();
        return *obj;
    }
    void destroy() { delete obj; }
};

Singleton* Singleton::obj = nullptr;
// если static поле не константна и не имеет целочисленного типа, то
// можно инициализировать только вне класса
```

****

**Виды памяти**

**Статический** — выделение памяти до начала исполнения  программы. Такая память доступна на протяжении всего времени выполнения  программы. Там хранятся глобальные переменные, статические переменные

**Автоматическая**, «размещение на  стеке», — самый основной, выделяется при вводе новой области действия и освобождает память при выходе из неё. 

## 5

**private** - доступ к методам и полям открыт внутри класса и для друзей класса. Производные этого класса доступа не имеют

**public** - доступ открыт абсолютно всем, кто видит определение данного класса

**Инкапсуляция** — механизм упаковки данных и функций в единый компонент (а также инкапсуляция позволяет ограничить доступ одних компонентов программы к другим). Один из трёх основных принципов ООП. Позволяет пользователю взаимодействовать с объектом через абстракции, не задумываясь об их реализации.

**Перегрузка методов в случае выбора приватных и публичных версий**

```cpp
class C {    
    private:    
    void add(int a) {}    
    public:    
    void add(double a) {}
};
C c;
c.add(1); // CE
```

Перегрузка выполняется до того, как компилятор проверяет доступность. Тем самым гарантируем, что именно от такой сигнатуры функцию вызвать нельзя. Сначала перегрузим, потом поймем, что функция приватная, дальше СЕ.



**Дружественные функции** (или классы) — это функции (или классы), которые не являются членами класса, однако имеют доступ ко всему, к чему имеет доступ текущий класс, кроме полей/методов класса, чьим другом является наш класс — дружба не транзитивна. Для определения дружественных функций (или классов) используется ключевое слово friend. Часто используется для перегрузки операторов, когда левый операнд не является членом класса (например, ввод и вывод в поток).

```cpp
class String {
  char* buffer;
  size_t size;
public:
  String(const char[]);
  //  позволяет определить оператор вывода в поток
  friend std::ostream& operator<<(std::ostream& out, const String& s);
  friend std::istream& operator>>(std::istream& in, const String& s);
};

std::istream& operator>>(std::istream& in, const String& s) {
    char t;
    while (std::cin >> t) {
        ...
    }
}
 
std::ostream& operator<<(std::ostream& out, const String& s) {
 for (int i = 0; i < s.size; ++i) {
   out << s.buffer[i];
 }
 return out;
}

```

## 6

**Перегрузка приведения типов** - механизм, который позволяет user-defined классам быть конвертируемыми в другие типы

```cpp
operator int() { // не имеет тип возвращаемого значения
    return smth;
}
```

type => MyType - с помощью конструкторов, обратно - перегрузка

**Explicit** ставится перед конструктором/оператором приведения типа и запрещает компилятору неявные конверсии с его использованием. Помечать конструкторы классов explicit во избежание непредвиденных преобразований (например, в аргументах функций) — распространённая практика.

```cpp
struct U {
    int id = 0;
    public:
    explicit U(int id): id(id) {}
    explicit operator int() {
        return id;
    }
}

U id = 5; // CE due to explicit constructor
5 + id; // CE due to explicit operator
```



Пример: карты, много функций, в аргументах некоторых сначала долгота, потом широта, и наоборот. Чтобы не запутаться, введем типы широты и долготы и запретим неявные конверсии одного к другому.

**Литерал** — это некоторое выражение, создающее объект:

```cpp
'x';      // character
"some";   // c-style string
7.2f;     // float
74u;      // unsigned int
74l;      // long
```

Можно создавать свои, при этом определять и реализовывать нужно вне класса.

```cpp
// сигнатура литерала для целочисленных типов
OutputType operator "" _suffix(unsigned long long);

// сигнатура литерала для вещественных типов
OutputType operator "" _suffix(long double);

42_suffix;      // OutputType operator "" _suffix(unsigned long long);
42.24_suffix;   // OutputType operator "" _suffix(long double);
```

```cpp
#include <iostream>
using namespace std;

class Lit {
	int id;
public:
	Lit(int val): id(id) {}
};

Lit operator "" _suffix(unsigned long long val) {
	return Lit(val);
}

int main() {
	343_suffix;
	return 0;
}
```

## 7

**Protected** - модификатор доступа, доступ к полям и методам открыт классам, производным от данного (наследникам), и друзьям.

**Типы наследования**

```cpp
// Открытое наследование
class Pub: public Parent { };
 
// Закрытое наследование
class Pri: private Parent { };

// по умолчанию - private наследование
```

- public наследование
  - наследникам доступны public, protected поля, private запрещены
- private наследование
  - private родителя из наследника недоступны, protected и public родителя в теле наследника становятся private

У структур наследование по умолчанию public, у классов - private

**Разница между видимостью и доступностью**

Обращение к приватному полю извне не то же самое, что обращение к несуществующему полю.

```cpp
struct Mom {
    void f() {}
}
struct Son: Mom {
    void f() {}
}

Son s;
s.f(); // по умолчанию частное предпочтительнее общего
s.Mom::f(); // явное обращение к методам родителя
```

Наглядно разница между доступностью и видимостью: модификаторы доступа не влияют на выбор версий

```cpp
struct Mom {
    void f() {}
    void g(double) {}
}

struct Son: Mom {
private:
    void f() {}
public:
    void g(int) {}
}

Son s;
s.f();
// f из Mom is not visible, f из Son ее затмевает
// доступность (модификаторы доступа) проверяется после
// выбора версий и перегрузки (выбрали Son::f() на пред. шаге)
// Получается, будет CE, версии сына недоступны

// При этом s.Mom::f() доступна, но не видна, s.f() видна, но недоступна

s.g(0.0); // версия Mom НЕ видна, сработает приведение типов к версии Son
```

**using**

```cpp
struct Son: Mom {
    using Mom::f;
}

// теперь версии родителя видны компилятору
```

**extra**

Конструкторы по умолчанию не наследуются, но в наследнике можно написать using Base::Base; и тогда можно будет создавать наследников через конструкторы родителей

## 8

**Порядок вызова конструкторов/деструкторов**

*Конструкторы вызываются в том порядке, в котором классы выводились один из другого. Порядок вызова деструкторов обратный.*

Перед тем, как вызовется конструктор, поля должны быть созданы и проинициализированы.

```cpp
struct A {
    A() {std::cout << 'A';}
    ~A() {std::cout << '~A';}
};

struct B {
    B() {std::cout << 'B';}
    ~B() {std::cout << '~B';}
};

struct Base {
    A a;
    Base() {std::cout << 'Base';}
    ~Base() {std::cout << '~Base';}
};

struct Derived : Base {
    B b;
    Derived() {std::cout << 'Derived';}
    ~Derived() {std::cout << '~Derived';}
}

Derived d;
// order:
// entering Derived
// init fields of parent - Base
// entering Base
// init field A
// entering A and init fields (empty)
// creation of A - print(A)
// return to Base: creation of Base - print(Base)
// return to Derived: Base fully inited, creation of B - print(B)
// fields inited, creation of Derived - print(Derived)

// destruct Derived - print(~Derived), print(~Base), print(~A)
```

**Обращение к конструкторам родителя из конструкторов наследника**

```cpp
class Derived : public Base {
    int y = 0;
    Derived(int x, int y): Base(x), y(y) {}
}

// сначала инициализировать родителя, потом уже свои поля (иначе warning)
// напрямую поля родителя инициализировать нельзя
```

**Множественное наследование**

```cpp
struct Mom {
    int a = 0;
    int m = 1;
}

struct Dad {
    int a = 1;
    int f = 2;
}

struct Son : Mom, Dad {
    int s = 3;
}

// [ Mom::m ][ Dad::f ][ Son::s ] - хранение в памяти Son (12 байт)

Son s;
print(&s);

Father* pf = &s; // смотрим на сына как на отца
// получим смещение указателя (сдвиг на 4 байта вправо)
print(pf); 

static_cast<Son*>(pf); // компилятор действует из предположения, что каст корректен
// сдвиг указателя влево на 4 байта

s.a; // ambigious call
```

**Проблема ромбовидного наследования**



```cpp
struct Granny {
    int g = 0;
}

struct Mom : Granny {
    int m = 1;
}

struct Dad: Granny {
    int f = 2;
}

struct Son: Mom, Dad {
    int s = 3;
}

// [ g ][ m ][ g ][ f ][ s ]

Son s;/
s.g; // CE
Granny& g = s; // CE
s.Mom::Granny::g(); // CE
```

Тут у сына две разные бабушки. 

## 14

Каст вниз - CE (подстановка родителя туда, где требуется наследник)

**Особенности использования данных операторов при приватном наследовании, множественном наследовании**

При множественном static_cast будет пытаться двигать указатели:

![image-20210623135643074](/home/h4zzkr/snap/typora/39/.config/Typora/typora-user-images/image-20210623135643074.png)

reinterpret_cast ничего двигать не будет, а просто посчитает, что начало Father - это начало Son, выведется 2 в последней строчке

**Приведения**

```cpp
Derived d;
Base b = d; // отрезать все, что относится к derived и создать на основе оставшегося Base b

Base& bb = d; // трактуем derived как Base

Base* pb = &d; // OK
Derived* pd = pb; // CE
```

Приватное наследование запрещает касты к родителю (извне класс как-будто бы не наследник)

Явные касты:

**static_cast при наследовании**

```cpp
static_cast<B&>(d); // теперь d как-будто бы Base
static_cast<B>(d); // создание копии - Base из Derived

Если наследование приватное, то static_cast даст CE.
    
Derived d;
Base& b = d;
static_cast<Derived&>(b); // можно даже вниз, но будет UB, если был не Derived, a Base
```

**reinterpret_cast при наследовании**

Позволяет тупо все (не смотрит на приватности и т.д.), кроме кастов от const в не const. Просто интерпретирует одни классы как другие. Можно только к ссылке и указателю, новый объект создать не может

**dynamic_cast**

Осуществляется в runtime. Его успешность зависит только от динамического состояния, применим только к полиморфным типам. Если нельзя привести, то возвращает nullptr. Можно применять к ссылке и указателю. Обычно используется для каста **вниз**.

```cpp
struct Granny {
    virtual void f();
}
struct Mother: Granny {}
struct Son: Mother {}

Mother* pm = new Mother();
std::cout << dynamic_cast<Son*>(pm); // 0 bcz cast is bad

Mother* pm = new Son();
std::cout << dynamic_cast<Son*>(pm); // ok, указатель на son
```

Касты в другие ветки и  пример ситуации, когда все три этих оператора ведут себя по-разному:

```cpp
struct Granny {
    virtual void f();
}
struct Mother: Granny {}
struct Father: Granny {}
struct Son: Mother, Father {}

// каст вбок
Mother* pm = new Son();
dynamic_cast<Father*>(pm); // ok
static_cast<Father*>(pm); // CE, типы несовместимы
reinterpret_cast<Father*>(pm); // ваще па**ю, но UB
```

**dynamic cast не работает:**

- при приватном и protected наследовании
- невиртуальные классы

При невозможности каста для ссылок кидает std::bad_cast, для указателей - nullptr

## 12

Виртуальные функции призваны решить следующую проблему:

```cpp
struct Base {
    void f() {}
    virtual void g() {} // переопределение
}
struct Derived : Base {
    void f() {}
    void g() override {}
}

Derived d;
Base& b = d;
b.f(); // вызов метода Base

b.g(); // вызов метода Derived
```

**Виртуальная функция в С++** — это особый тип  функции, которая, при её вызове, выполняет «наиболее» дочерний (частный) метод,  который существует между родительским и дочерними классами

Когда сигнатура у переопределяемой функции отличается от исходной виртуальной, виртуальность тут же ломается (например, const в наследнике не будет переопределять non const virtual в родителе)

Выбор переопределения совершается в runtime, потому что в compile time нельзя проверить, какой объект лежит под ссылкой.

Если написали override, но сигнатура не совпала с virtual, то CE

**Правило: Никогда не вызывайте виртуальные функции в теле конструкторов или деструкторов (будет вызвана родительская версия).**

```cpp
#include <iostream>
using namespace std;

struct A {
	private:
	virtual void f() {
		std::cout << "1";
	}
};

struct B : A {
	void f() {
		std::cout << "2";
	}
};

int main() {
	B b;
	A& a = b;
	a.f(); // CE
    // выбор кандидатов происходит на этапе компиляции в классе А (ссылка на B сейчас неотличима от А). Компилятор видит, что пытаемся вызвать приватный метод и выкидывает CE
	return 0;
}
```

Типы, у которых ест8 хот8 одна виртуал8на- функци- – полиморфный тип

![image-20210623184411028](/home/h4zzkr/snap/typora/39/.config/Typora/typora-user-images/image-20210623184411028.png)

private не вызовет СЕ

## 13

Пусть у нас есть геометрические фигуры, и мы хотим сделать класс-родитель “Фигура”, чтобы создать, допустим, массив “указателей на Фигуру” и хранить там разные фигуры. И, допустим, мы хотим уметь искать площадь фигур. Но нет общего алгоритма подсчёта площади любой фигуры, который мы могли бы написать в “Фигуре”. На помощь приходят чисто виртуальные функции. 

Мы пишем **virtual double** area() = 0; в родителе, и такая функция *может не иметь* тела, но её обязаны определить наследники. Тогда класс-родитель становится **абстрактным** классом (то есть, нельзя создать объект такого типа), и всякий наследник, который не определит эту функцию, будет тоже абстрактным классом.

**virtual destructors.** Представим следующую ситуацию:

```cpp
// B - наследник A, деструктор обычный
B* b = new B();
A* a = b;
delete a;
```

Произойдет утечка памяти, потому что вызовется деструктор A, и B-часть останется неудалённой. Поэтому при наследовании всегда стоит объявлять // B - наследник A, деструктор обычный

B* b = **new** B();

A* a = b;

**delete** a; **виртуальным**, если вы хотите обращаться к наследнику через указатель на родителя

**RTTI**

Runtime type information - для полиморфных типов компилятор должен уметь выбирать правильную версию. Но в compile time это сделать нельзя - нельзя предугадать, что будет лежать под ссылкой или указателем.  

Это механизм, который позволяет определить тип переменной или объекта на этапе выполнения программы.

**typeid()** - спросить в runtime тип объекта, возвращает std::typeinfo, их можно сравнивать на равенство

```cpp
Son s;
Mother& m = s;
typeid(m) == typeid(s) // true
```

typeid(s).name() - вернуть имя класса объекта, typeid замедляет runtime

## 9

 **Шаблоны** — это средство языка, предназначенное для написания кода без привязки к конкретному типу.

```cpp
template <typename T>
struct remove_reference {
    typedef type T;
}

template <typename T>
void max(const T& a, const T& b) {}

max<int>(1, 2.0);
```

**Перегрузка функций с шаблонными**

Пример перегрузки шаблонных функций с участием шаблонной и не шаблонной версии

*частное лучше общего*

```cpp
template <typename T>
void f(T x) {}
void f(int x) {}
f(3); //выберется вторая версия, частное лучше общего
```

*точное соответствие лучше приведения типов*

```cpp
#include <iostream>
using namespace std;

template <typename T, typename U>
void f(T x, U y) { std::cout << 1; }

template <typename T>
void f(T x, T y) { std::cout << 2; }

void f(int x, double y) { std::cout << 3; }

int main() {
	f(0, 0); // 2
    f(0.5, 0) // 1
	return 0;
}
```

Шаблонная перегрузка по ссылке и по значению - CE, но

```cpp
template <typename T>
void f(T& x) {}

template <typename T>
void f(const T& x) {}

int x = 0;
int& y = x;
f(y); // выберется первая версия (не нужен каст)
```

## 10

**Специализации шаблонов** нужны для случаев, когда мы хотим, чтобы с данным набором типов данных функция вела себя по-другому (т.е. как-то специально).

```cpp
template<typename T>
struct remove_ref {
   typedef T type;
};
 
// частичная
template<typename T>
struct remove_ref<T&> {
   typedef T type;
};
 
// частичная (а также для констант, указателей, массивов)
template<typename T>
struct remove_ref<T&&> {
   typedef T type;
};

// полная
template<>
struct remove_ref<int> {
    typedef int type; // useless
}
```

**Частичная специализация шаблона позволяет выполнить  специализацию шаблона класса (но не функции!), где некоторые (но не все) параметры шаблона явно определены**.

```cpp
// Шаблон функции print() с частично специализированным шаблоном класса StaticArray<char, size> в качестве параметра
template <int size> // size по-прежнему является non-type параметром
void print(StaticArray<char, size> &array) // мы здесь явно указываем тип char
{
	for (int count = 0; count < size; ++count)
		std::cout << array[count];
}
```



Без исходного шаблона это работать не будет (нужно соблюдать сигнатуру шаблонов). Классы перегружать нельзя, для них только специализации.

У функций частичная специализация запрещена.

**Разница**

Специализация зависит от родительского шаблона и выбирается после того, как выбрана перегрузка.

```cpp
template <typename T, typename U>
void f(T, U) { std::cout << 1; }

template <> // специализация от T, U
void f(int, int) { std::cout << 3; }

template <typename T>
void f(T, T) { std::cout << 2; }

f(0, 0); // выберется 2, т.к. сначала разрешится перегрузка (частное лучше общего)
```

**std:is_same**

```cpp
template<class T, class U>
struct is_same : std::false_type {};
 
template<class T>
struct is_same<T, T> : std::true_type {};
```

## 15

В runtime может пойти что-то не так. Как это происходит? Выполняется недопустимая операция, программа вылетает с ошибкой (exceptions). Мы же хотим эту операцию отловить (сделать специальные действия на тот случай, если программа даст ошибку). Программа при этом не завершится.

Terminate - аварийное завершение программы, например когда исключение нигде не поймали.

**Недостатки исключений:**

- try ловит только то, что было брошено с помощью throw. Т.е. нельзя отследить UB, битые ссылки и т.п. Нельзя ловить RE - не каждая ошибка - исключение.
- при вызове throw мы прекращаем выполнение текущего кода, поэтому потенциально могут быть не освобождены выделенные ресурсы
- исключения нельзя использовать в деструкторах
- могут замедлить (ненамного) код
- усложняют написание библиотек:
- нужно помнить про noexcept

**Преимущества:**

- повышают надежность кода
- механизм передачи исключений вверх по вложенным функциям
- позволяют не прекращать работу в случае каких-то ошибок

**Копирование исключений**

```cpp
void f() {
    Noisy x;
    throw x; // уничтожается, поэтому при выходе создается копия
}

int main() {
    try {
        f();
    } catch (const Noisy& x) { // поймали по ссылке копию
        // без const & будет опять копия
        std::cout << "caught";
    }
}
```

**throw и throw ex**

```cpp
void g() {
    try {
        throw std::out_of_range("a");
    } catch (const std::exception& ex) {
        // поймали по ссылке на родителя
        throw; // передать std::out_of_range вверх
        // throw ex; // бросить локальный еще раз, т.е. скопируем как новое std::exception, старое уничтожится
    }
}

void exit() {
    throw; // сразу получим terminate, нельзя никак ничего поймать
}

int main() {
    try {
        g();
    } catch (std::out_of_range& oor) {
        std::cout << "caught2"; // поймаем нужное исключение
    }
}
```

**правила приведения типов**

не действует приведение типов (даже если они приводятся), за исключением ситуации приведения между родителем и наследником и const/non cost

**Правила выбора подходящей секции catch**

При нескольких catch подряд выбирается первый подходящий

## 16

Если конструктор объекта выкинул исключение, то объект считается не до конца созданным, следовательно от него нельзя вызвать деструктор (иначе возникало бы UB), а значит может произойти утечка ресурсов, которые были выделены конструктором до того, как было выкинуто исключение.

Для решения таких проблем нужны умные указатели. Тогда при аварийном выходе вызовется деструктор умного указателя и ресурс будет освобожден.

**Идиома RAII - Resourse Acquisition Is Inizialization** - захват ресурса есть инициализация некоторого объекта. Всякий раз, когда нужно захватить какой-то ресурс, это делаем не лоб, а с помощью объекта, который явно это делает. Тогда каждый раз когда будет выбрасываться исключение, деструктор локальных объектов гарантированно вызовутся и следовательно ресурсы будут освобождены вовремя.

**Исключения в деструкторах**

*Язык не поддерживает обработку  нескольких исключений одновременно*, поэтому могут возникать неприятные ситуации:

```cpp
void g() {
    Dangerous a;
}

void f() {
    Dangerous b;
    g();
    // exception on g
    // exception on b
    // => TERMINATE
}
```

Поэтому все деструкторы считаются noexcept.

**Спецификации исключений**

- есть функции, не выбрасывающие исключений
- есть функции, которые потенциально выбрасывают исключения

C C++11 есть возможность указать в сигнатуре, кидает ли потенциально функция исключение с помощью ключевого слова noexcept. Это обещание, которое не проверяется компилятором. Если функция, отмеченная noexcept выкинет исключение, то программа сразу завершится вызовом terminate (но компилятор выдаст Warning)

``` 
noexcept(true) - не выбрасывает исключения
```

```cpp
void function() noexcept(true) {} // спецификатор noexcept, мб условным (содержать условие)
// noexcept(expression) - оператор, который возвращает true, если выражение не содержит операторов, приводящих к исключению:
// throw, new, dynamic_cast, no noexcept function
```

noexcept(1 / 0) - true, не вычисляется в compile time, а просто оценивается

## 17

Оператор new - довольно низкоуровневая абстракция, поэтому чтобы работать на более высоком уровне, на уровне языка программы, а не на уровне операционной системы, придумали выделять и владеть памятью в виде класса.

```cpp
template <typename T>
struct allocator {
    T* allocate(size_t n) {
        return ::operator new(n * sizeof(T));
    }
    
    void deallocate(T* ptr, size_t n) {
        ::operator delete(ptr);
    }
    
    template <typename... Args>
    void construct(T* ptr, const Args&... args) {
        new (ptr) T(args...);
    }
    
    void destroy(T* ptr) {
        ptr->~T();
    }
};
```

**placement new** - направляем конструктор на уже выделенную по указателю память. Собственно, потом придется вручную разрушать объект и чистить память с помощью ::operator delete(ptr)

Отличие от new в том, что new выделяет память и сразу же конструирует на ней объект.

**allocator_traits** - Создан для того, чтобы некоторые вещи доопределить за вас, так как некоторые методы в практически всех аллокаторах делают одно и то же. https://docs.microsoft.com/en-gb/cpp/standard-library/allocator-traits-class?view=msvc-160

**rebind** - нужен для подмены типа, выделяемого на аллокаторе. Например, в list нужно аллоцировать Node, а не тип Т:

```cpp
template <typename T, typename Alloc = std::allocator<T>>
class list {
    struct Node {...};
    ...;
    std::allocator_traits<Alloc>::rebind_alloc<Node> alloc;
    
public:
    list(const Alloc& alloc = Alloc()) : alloc(alloc) {}
    	
};

template<typename U> struct rebind { typedef FastAllocator<U> other; };
```

Изнутри контейнера получаем аллокатор для другого типа.



## 18

**Итераторы** - сущности которые позволяют перечислять элементы некоторой последователь-
ности. То что ведет как итератор и есть итератор. Вести себя как итератор - позволять себя
разыменовывать, инкрементировать и сравнивать на равенство.

**Const iterator** - не позволяют менять объект под собой. Разыменовывая такой итератор,
получаем ссылку на константный объект. Константному итератору можно присвоить обыч-
ный, но не наоборот

*std::conditional_t*<bool_value, type_one, type_two> - rovides member typedef `type`, which is defined as `type_one` if is true at compile time, or as `type_two` if `B` is false.

**итератор над List без дублирования кода**

```cpp
/// START OF ITERATOR ///
#include <iterator>

template<bool IsConst>
struct common_iterator {
public:
    // for iterator_traits
    using difference_type       = std::ptrdiff_t;
    using value_type            = T;
    using pointer               = typename std::conditional_t<IsConst, const T *, T*>;
    using reference             = typename std::conditional_t<IsConst, const T &, T&>;
    using iterator_category     = std::bidirectional_iterator_tag;

private:
    friend List;
    Node* ptr;
    common_iterator(Node* ptr) : ptr(ptr) {}

public:
    common_iterator() = default;
    common_iterator(const common_iterator &it) : ptr(it.ptr) {}
    common_iterator &operator=(const common_iterator &other) {
        ptr = other.ptr;
        return *this;
    }
    pointer operator->() { return &ptr->value; }
    reference operator*() { return ptr->value; }
    common_iterator &operator++() {
        ptr = ptr->next;
        return *this;
    }
    common_iterator &operator--() {
        ptr = ptr->prev;
        return *this;
    }
    common_iterator operator++(int) {
        common_iterator iter(*this);
        ++(*this);
        return iter;
    }
    common_iterator operator--(int) {
        common_iterator iter(*this);
        --(*this);
        return iter;
    }
    common_iterator &operator+=(size_t shift) {
        for (size_t i = 0; i < shift; ++i)
            ptr = ptr->next;
        return *this;
    }
    common_iterator &operator-=(size_t shift) {
        for (size_t i = 0; i < shift; ++i)
            ptr = ptr->prev;
        return *this;
    }
    common_iterator operator+(size_t shift) {
        common_iterator n(*this);
        n += shift;
        return n;
    }

    common_iterator operator-(size_t shift) {
        common_iterator n(*this);
        n -= shift;
        return n;
    }
    bool operator==(const common_iterator &it) const { return (ptr == it.ptr); }
    bool operator<(const common_iterator &rhs) const { return ptr < rhs.ptr; }
    bool operator<=(const common_iterator &rhs) const { return !(*this > rhs); }
};

using iterator                  = common_iterator<false>;
using const_iterator            = common_iterator<true>;

iterator begin() { return iterator(afterfront->next); }}
iterator end() { return iterator(afterback); }
const_iterator cbegin() const { return const_iterator(afterfront->next); }
const_iterator cend() const { return const_iterator(afterback); }
/// END OF ITERATOR ///
```

## 19

Функция **std::advance** перемещает итератор на определенное число позиций вперед, ничего
не возвращает, **std::distance** возвращает расстояние между двумя итераторами. Если второй
итератор недостижим от первого, то поведение std::distance не определено.

```cpp
std::list<int> l1 = {1, 2, 3, 4};auto it = l1.begin();std::advance(it, 2);std::distance(l1.begin(), l1.end());
```

**Реализация std::advance**

```cpp
// 1 version

template <typename iterator>
void my_advance(iterator& iter, size_t n) {
    if constexpr (std::is_same_v<typename std::iterator_traits<iterator>::iterator_category, std::random_access_iterator_tag>) {
        iter += n;
        std::cout << 1;
    } else {
        for (int i = 0; i < n; ++i, ++iter);
    }
} // без constexpr компилятор бы ругался на iter+=n для не random_access

// 2 version
template <typename iterator, typename itercat>
void advance_helper(iterator& iter, int n, itercat) {
    for (int i = 0; i < n; ++i, ++iter); // all but not RA
}

template <typename iterator>
void advance_helper(iterator& iter, int n, std::random_access_iterator_tag) {
    iter += n;
}

template <typename iterator>
void my_advance(iterator& iter, size_t n) {
    advance_helper(iter, n, typename std::iterator_traits<iterator>::iterator_category());
}
```

## 20 - 21хор

**Формальное определение lvalue, rvalue**

Важно: 

- rvalue и lvalue - виды выражений, не объектов и не типов

- Rvalue ссылка далеко не всегда rvalue, как и lvalue ссылка не обязательно lvalue, то есть 𝑙𝑣𝑎𝑙𝑢𝑒&𝑟𝑣𝑎𝑙𝑢𝑒 это виды выражений (синтаксическая конструкция, составленная из переменных, литералов и операторов), а не объектов или типов.

Интуитивно rvalue - выражение, которе представляет из себя создание нового объекта (то, что не может стоять слева от знака равенства). А lvalue - выражение, представляющее из себя обращение к уже существующему объекту (то, что может стоять слева от знака равенства). К данным определениям есть контрпримеры, это не всеобъемлющее определение.

**Примеры, когда rvalue-выражение допускает присваивание и когда lvalue не допускает**

Const objects - это lvalue, но им присваивать нельзя, или просто
объекты, у которых не определён оператор присваивания. 

Противоположный пример, когда мы делаем обращение к 𝑣𝑒𝑐𝑡𝑜𝑟 < 𝑏𝑜𝑜𝑙 > с помощью [], мы получаем не ссылку, а временный объект BitReference, к которому мы и должны присваивать, чтобы изменить значение.

*Type&& - rvalue reference, type& - lvalue reference*

**Expressions:**

- lvalue:
  - идентификаторы (имя переменной)
  - операторы =, (+-*/)= над примитивными типами
  - prefix ++ -- on standart types
  - унарная *
  - result of ?: if both operands are lvalue
  - result of ",", if right operand is lvalue
  - результат вызова функции (или метода или custom operator call), если возвращаемый тип - lvalue-reference
  - результат каста, если result-type is lvalue-reference
- rvalue:
  - *prvalue - pure rvalue*
    - литералы (5, 3.0f)
    - бинарные операторы +, -, *, /, % etc над примитивными
    - postfix ++ --
    - унарный &, +, -
    - result of ?: if both are rvalue
    - result of "," if right operand is rvalue
    - результат вызова функции (или метода или custom operator call), если возвращаемый тип - не ссылка
    - результат каста, если result-type is not lvalue-reference
  - *xvalue - expired value*
    - результат вызова функции (или метода или custom operator call), если возвращаемый тип - это rvalue-reference
    - результат каста, если result-type is rvalue-reference

Тогда rvalue-reference ведет себя как обычная ссылка, но ее возвращение считается как rvalue expression

**Инициализация rvalue ссылок**

```cpp
int x = 0;
int& rx = x; // ok
const int& crx = 2;

// rvalue ссылки можно инициализировать только от rvalue
int&& rrx = x; // wrong

int&& rrx = 1; // ok, продление жизни объекта
rrx = x; // ok то, что под ссылкой, теперь 0

x = 3; // но rrx все равно 0, ссылки нельзя переприсваивать
int& rx = rrx; // rrx - идентификатор => lvalue
```

move позволяет инициализировать rvalue-ссылку посредством lvalue

*Note:* когда выводится параметр шаблона Т шаблонной функции, все амперсанды у переменной отбрасываются, но не в случае передачи по универсальной ссылке

**reference-collapsing** - работает только в функциях с универсальными ссылками

```cpp
template <typename T>
void f(T&& x) {
    ...
}

int y = 5;
f(y); // y - lvalue, T = int&, decltype(x) = int&
```



& + & = &

& + && = &

&& + & = &

&& + && = &&

## reference qualifiers

```cpp
struct S {
    void f() & { // can be called only on lvalues
        ...
    }
    void f() && { // rvalues
        ...
    }
}

S s;
s.f();
std::move(s).f(); // перегрузка по типу ссылки левого операнда
```

## 22

**Универсальные ссылки**

std::move должна принимать и lvalue и rvalue, причем не константные.

```cpp
f(const T& x); - lvalue and rvalue, but const;
f(T& x); f(T&& x) - only l(r)value;
```

```cpp
template <typename T>
void f(T&& x); - принимает как lvalue, так и rvalue
```

Приоритетнее, чем передача по константной

**Реализация std::move**

```cpp
template <typename T>
std::remove_reference_t<T>&& move(T&& x) noexcept { // превратить все в rvalue
    return static_cast<std::remove_reference_t<T>&&>(x); // каст к rvalue - rvalue
}
```

Если возвращаем T&&, то работает ref.collapsing, и может вернуться lvalue. Нужна всегда rvalue. Поэтому нужно убирать этот амперсанд. 

## 23 (отл 11)

**perfect forwarding, std::forward**

```cpp
template <typename... Args>
void emplace_back(const Args&... args) {
    if (sz == cap) reserve(2 * cap);
    AllocTraits::construct(alloc, arr + sz, args...);
    ++sz;
}
```

Преимущество перед push_back в том, что создаем объект сразу по нужному адресу в памяти, при push_back пришлось бы использовать временно созданный объект, потом его копировать.

Но в emplace_back тоже принимает по конст ссылке. А если параметры конструктора объекта тяжело копировать по времени?

Часть аргументов м.б. lvalue, часть rvalue, надо передать как rvalue только те, которые изначально были rvalue.

```cpp
template <typename... Args>
void emplace_back(Args&&... args) {
    if (sz == cap) reserve(2 * cap);
    AllocTraits::construct(alloc, arr + sz, std::forward<Args>(args)...); // args - lvalue, но тип либо &&, либо &
    ++sz;
}
```

```cpp
template <typename T>
T&& forward(std::remove_reference_t<T>& x) noexcept {
    return static_cast<T&&>(X);   
}

forward<T>(type val);
```

val is lvalue => T is type&, decltype(x) = type& => T&& = type&

val is rvalue => T is type, decltype(x) = type& - принимаем всегда все равно по lvalue-ref => T&& = type&&

![image-20210625231932866](/home/h4zzkr/snap/typora/39/.config/Typora/typora-user-images/image-20210625231932866.png)

## 30

**мотивировка auto**

Проблема: 

слишком длинные названия типов, например в случае std::unordered_map с нестандартным аллокатором, или например, в том же типе можно случайно забывать квалификатор const у ключа, и как следствие можно получить лишние копирования например в range-based loop. Решение этих проблем- ключевое слово auto, которое сообщает компилятору, что тип переменной должен быть установлен исходя из типа инциализируемого значения.

В auto, как и в T, ссылки отбрасываются.

auto&& y = f(x); // флешбек, универсальная ссылка

**нельзя использовать**:

- auto не может использоваться в аргументах функции; 
- auto не может использоваться в качестве возвращаемого значения функции, если в зависимости от работы функции возвращаются вещи разных типов

**decltype ** - в CT возвращает тип выражения, сохраняет ссылки, можно навешивать амперсанды.

```cpp
int x = 1; f<decltype(x)>();
// decltype не отбрасывает амперсанды
```

```cpp
if epxression is not and identifier:
	if expression is prvalue of type T: decltype(expression) is T
    if expression is lvalue of type T: decltype(expression) is T&
    if expression is xvalue of type T: decltype(expression) is T&&
```

auto можно писать в return type функций (но если возвращают одинаковые типы)

Пример необходимого auto:

```cpp
template <typename T>
auto h(const T& x) -> decltype(f(x)) {
    return f(x); // может возвращать как lvalue, так и rvalue
    // как определить возвращ. тип в зависимости от принятого аргумента
}

```

**decltype(auto)**

```cpp
template <typename Container>
decltype(auto) get(const Container& container, size_t index) {
    // сверху decltype(auto) аналогичен предыдущим записям с ->
    return container[index];
} // вывод типа без отбрасывания ссылок
```

decltype(auto) p = x; // как auto, но с сохранением амперсандов



## 28

**мотивировка**

Позволяют автоматически освобождать некоторый выделенный динамический ресурс (RAII)

**unique_ptr** - простейший умный указатель, ведет себя как c-style pointer, но его нельзя скопировать (единолично владеет ресурсом) - его копирующий конструктор и оператор удалены.

У unique_ptr существует специализация для массивов: std::unique_ptr<int[]>

```cpp
template<typename T>
class unique_ptr {
private:
   T *ptr;
public:
   unique_ptr(T *ptr) : ptr(ptr) {}
   unique_ptr(const unique_ptr&) = delete;
   unique_ptr &operator=(const unique_ptr &) = delete;

   unique_ptr(unique_ptr&& another) noexcept {
       ptr = another.ptr;
       another.ptr = nullptr;
   };
   unique_ptr& operator=(unique_ptr&& another) noexcept {
       if (&another == this)
           return *this;
       delete ptr;
       ptr = another.ptr;
       another.ptr = nullptr;
       return *this;
   };
   T& operator*() const {
       return *ptr;
   }
   T* operator->() const noexcept {
       return ptr;
   }
   ~unique_ptr() {
       delete ptr;
   }
};

```

```cpp
template <typename T, typename... Args>
unique_ptr<T> make_unique(Args&&... args) {
    return unique_ptr<T>(new T(std::forward<Args>(args)...));
}
```



## 29

**shared_ptr -** позволяет создавать несколько указателей на одно и то же, при этом удаление будет происходить один раз - когда все копии умрут.

*Implementation draft*

```cpp
#include <iostream>

template <typename T>
class shared_ptr {
    T* ptr = nullptr;
    size_t* count = nullptr;
public:
    shared_ptr() {}
    explicit shared_ptr(T* ptr): ptr(ptr), count(new size_t(1)) {}
    
    shared_ptr(const shared_ptr& other): ptr(other.ptr), count(other.count) {
        ++*count;
    }
    
    shared_ptr(shared_ptr&& other): ptr(other.ptr), count(other.count) {
        other.ptr = nullptr;
        other.count = nullptr;
    }
    
    shared_ptr& operator=(const shared_ptr& other) {
        ~shared_ptr();
        ptr = other.ptr;
        count = other.count;
        ++*count;
    }
    
    shared_ptr& operator=(shared_ptr&& other) {
        if (&other == this)
           return *this;
        ~shared_ptr();
        ptr = other.ptr;
        count = other.count;
        other.ptr = nullptr;
        other.count = nullptr;
    }

    T& operator*() {
      return *ptr;
    }

    T* operator->() {
      return ptr;
    }
    
    ~shared_ptr() {
        if (!count) return;
        --*count;
        if (!*count) {
            delete ptr;
            delete count;
        }
    }
};

int main() {
  shared_ptr<int> a(new int); // bad style
}
```

**Принцип: не следует использовать c-style pointers с shared_pointers**

```cpp
void g(const shared_ptr<int>& p, int x) {}
int f(int x) {
    throw std::runtime_error();
}
g(shared_ptr<int>(new int(5)), f(0));
// компилятор может вычислять аргументы в произвольном порядке, потенциальный memory leak
```

Чтобы избавиться от ручного использования new, напишем функцию make_shared (make_unique).

```cpp
template <typename T, typename... Args>
shared_ptr<T> make_shared(Args&&... args) {
    // оптимизация, один вызов new вместо двух
    auto p = ControlBlock();
    return shared_ptr<T>(p);
}
```

## 31

```cpp
[](int x, int y) { return x < y; }
[](int x) { std::cout << x << '\n' }(4);
auto f = [](int y) { return y * 2; }
```

```cpp
[](int x, int y) -> bool { ... }; // задать явно тип возвращаемого значения
```

**capture lists**

```cpp
int a = 1;
[a](int x) { return a + x; } // захват переменной в функции
```

```cpp
[a](int x) {++a;
           std::cout << a;} // CE, захваченные переменные константные с т.ч. функции
```

```cpp
[a, b](int x) mutable {...} // теперь все переменные изменяемые, но на исходные переменные изменения не повлияют
```

```cpp
[&a, b](int x) {
    ++a; return 1;
} // теперь a меняется, захват по ссылке
```

## 32

**sfinae, идея**

Неудачная шаблонная подстановка - не ошибка компиляции

```cpp
template <typename T>
auto f(const T&) -> decltype(T().size()) { std::cout << 1; }

int f(...) { std::cout << 2; return 0; }

f(std::vector<int>(4)); // ok, first ver
f(4); // неудачная подстановка в первую версию, выбор 2й - SFINAE
```

Работает только для сигнатур функций/классов. В теле функции будет CE. Это правило, применяется в момент выбора версий

**std::enable_if**

```cpp
template <typename T, typename = std::enable_if_t<std::is_class_v<std::remove_reference_t<T>>>>
void g(const T&) {
	std::cout << 1;
}

template <typename T, typename = std::enable_f_t<!std::is_class_v<std::remove_reference_t<T>>>>
void g(T&&) {
	std::cout << 2;
}
```

```cpp
template <bool B, typename T = void>
struct ebable_if {};

template<typename T>
struct enable_if<true, T> {
    using type = T;
}

template <bool B, typename T>
using enable_if_t = typename enable_if<B,T>::type;
```

Если попадаем в <false,T>, то фиктивный параметр отсутствует => срабатывает sfinae

## 11

**fibonacci**

```cpp
#include <iostream>

// Fibonacci
template <size_t N>
struct Fibonacci {
  static const size_t value = Fibonacci<N-1>::value + Fibonacci<N-2>::value;
};

template <>
struct Fibonacci<0> { static const size_t value = 0; };

template <>
struct Fibonacci<1> { static const size_t value = 1; };
```

**isPrime**

```cpp
#include <iostream>
template <size_t N, size_t D>
struct isPrimeHelper {
  static const bool value = (N % D == 0) ? false : isPrimeHelper<N, D-1>::value;
};

template <size_t N>
struct isPrimeHelper<N, 1> {
  static const bool value = true;
};

template <size_t N>
struct isPrime {
  static const bool value = isPrimeHelper<N, N-1>::value;
  // для O(N) запуститься от <N, N - 1>
};

template <>
struct isPrime<1> {
  static const bool value = false;
};

int main() {
  std::cout << isPrime<73>::value;
}
```

## 33

```cpp
int x;
std::cin >> x;
const int y = x; // константа реального времени

constexpr int y = x; // CE
```

**constexpr** - константа времени компиляции, обязана быть вычислена на этапе компиляции

```cpp
std::array<int, n> a; // n is constexpr
static const int val = ... // constexpr
```

```cpp
constexpr int factorial(int n) {
    ...
}

constexpr int y = factorial(5);
```

Чтобы вызов функции считался константным выражением, функция должна быть constexpr, т.е. такая, которая мб вызвана в compile time ради вычисления constexpr константы.

**since cpp17 - if constexpr** - проверяется на этапе компиляции, ложная ветка не будет компилироваться

Разрешается работать только constexpr выражениями.

Нельзя работать с динамической памятью в constexpr функциях. Создавать объекты с не constexpr когструкторами нельзя. Бросать исключения тоже нельзя в compile time (если компилятор зайдет в ветку, где бросается исключение)

То, что функция constexpr, не обязывает компилятор всегда ее вычислять на этапе компиляции

![image-20210627155436722](/home/h4zzkr/snap/typora/39/.config/Typora/typora-user-images/image-20210627155436722.png) 



## 24 - Vector

**reserve** должен просто выделять память, а не конструировать на ней что-либо. 

```cpp
void reserve(size_t n) {
    if (n <= capacity) return;
    T* newarr = AllocTraits::allocate(alloc, n);
    for (size_t i = 0; i < size; ++i)
        AllocTraits::construct(alloc, newarr + i, std::move(arr[i]));
    for (size_t i = 0; i < size; ++i)
        AllocTraits::destroy(alloc, arr + i);
    AllocTraits::deallocate(arr, capacity);
    capacity = n;
    arr = newarr;
}

void push_back(const T& value) {
    if (size == capacity)
        reserve(2 * capacity);
    AllocTraits::construct(alloc, arr + size, value);
    ++size;
}

void resize(size_t n, const T& value = T()) {
    if (n > capacity) reserve(n);
    if (n > size) { ... } // add some values with value
    else if (n < size) { ... } // remove some values
}
```



## 25 vector<bool>

```cpp
// пакуем по 8 бит в одной ячейке
template <>
class Vector<bool> {
    int8_t* arr;
    size_t size;
    size_t capacity;
    
    struct BitReference {
        int8_t* cell;
        uint8_t pos;
        
        BitReference& operator=(bool b) {
            if (b)
                *cell |= (1u << pos);
            else
                *cell &= ~(1u << pos);
        }
        
        operator bool() const {
            return *cell & (1u << pos);
        }
    }
    
public:
    BitReference operator[](size_t i) {
        return BitReference{arr + i / 8, i % 8};
    }
}
```