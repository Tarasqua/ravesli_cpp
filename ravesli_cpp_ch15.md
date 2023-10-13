# Глава №15. Умные указатели и Семантика перемещения в C++
## Содержание
1. [Урок №197. Умные указатели и Семантика перемещения](#урок-197-умные-указатели-и-семантика-перемещения)
2. [Урок №198. Ссылки r-value](#урок-198-ссылки-r-value)
3. [Урок №199. Конструктор перемещения и Оператор присваивания перемещением](#урок-199-конструктор-перемещения-и-оператор-присваивания-перемещением)
4. [Урок №200. Функция `std::move()`](#урок-200-функция-stdmove)
5. [Урок №201. Умный указатель `std::unique_ptr`](#урок-201-умный-указатель-stduniqueptr)
6. [Урок №202. Умный указатель `std::shared_ptr`](#урок-202-умный-указатель-stdsharedptr)
7. [Урок №203. Умный указатель `std::weak_ptr`](#урок-203-умный-указатель-stdweakptr)
8. [Глава №15. Итоговый тест](#глава-15-итоговый-тест)

## [Урок №197. Умные указатели и Семантика перемещения](#урок-197-умные-указатели-и-семантика-перемещения)
```c++
#include <iostream>

template<class T>
class Auto_ptr1 {
    T *m_ptr;
public:
    // Получаем указатель для "владения" через конструктор
    explicit Auto_ptr1(T *ptr = nullptr) : m_ptr(ptr) {}

    // Деструктор позаботится об удалении указателя
    ~Auto_ptr1() { delete m_ptr; }

    // Выполняем перегрузку оператора разыменования и оператора ->, 
    // чтобы иметь возможность использовать Auto_ptr1 как m_ptr
    T &operator*() const { return *m_ptr; }
    T *operator->() const { return m_ptr; }
};

// Класс для проверки работоспособности вышеприведенного кода
class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    Auto_ptr1<Item> item(new Item); // динамическое выделение памяти
    // ... но никакого явного delete здесь не нужно
    // Также обратите внимание на то, что Item-у в угловых скобках не требуется символ *, 
    // поскольку это предоставляется шаблоном класса
    return 0;
} // item выходит из области видимости здесь и уничтожает выделенный Item вместо нас

// Item acquired
// Item destroyed
```

Рассмотрим детально, как работают эти программа и класс. Сначала мы\
динамически выделяем объект класса `Item` и передаем его в качестве параметра\
нашему шаблону класса `Auto_ptr1`. С этого момента объект `item` класса `Auto_ptr1`\
владеет выделенным объектом класса `Item` (`Auto_ptr1` имеет композиционную\
связь с `m_ptr`). Поскольку `item` объявлен в качестве локальной переменной и\
имеет область видимости блока, он выйдет из области видимости после\
завершения выполнения блока, в котором находится, и будет уничтожен. А\
поскольку это объект класса, то при его уничтожении будет вызван деструктор\
`Auto_ptr1`. Этот деструктор и обеспечит удаление указателя `Item`, который он\
хранит!

До тех пор, пока объект класса `Auto_ptr1` определен как локальная переменная (с\
автоматической продолжительностью жизни, отсюда и часть «Auto» в имени\
класса), `Item` гарантированно будет уничтожен в конце блока, в котором он\
объявлен, независимо от того, как этот блок (функция `main()`) завершит свое\
выполнение (досрочно или нет).

**Умный указатель** — это класс, предназначенный для управления динамически\
выделенной памятью и обеспечения освобождения (удаления) выделенной памяти\
при выходе объекта этого класса из области видимости.

```c++
#include <iostream>

template<class T>
class Auto_ptr1 {
    T* m_ptr;
public:
    // Получаем указатель для "владения" через конструктор
    explicit Auto_ptr1(T* ptr=nullptr) :m_ptr{ptr} {}

    // Деструктор позаботится об удалении указателя
    ~Auto_ptr1() { delete m_ptr; }

    // Выполняем перегрузку оператора разыменования и оператора ->,
    // чтобы иметь возможность использовать Auto_ptr1 как m_ptr
    T& operator*() const { return *m_ptr; }
    T* operator->() const { return m_ptr; }
};

// Класс для проверки работоспособности вышеприведенного кода
class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
    static void sayHi() { std::cout << "Hi!\n"; }
};

void myFunction() {
    Auto_ptr1<Item> ptr(new Item); // ptr теперь "владеет" Item-ом

    int a;
    std::cout << "Enter an integer: ";
    std::cin >> a;

    if (a == 0)
        return; // досрочный возврат функции

    // Использование ptr
    ptr->sayHi();
}

int main() {
    myFunction();

    return 0;
}

// если ненулевое значение
// Item acquired
// Enter an integer: 7
// Hi!
// Item destroyed

// ноль
// Item acquired
// Enter an integer: 0
// Item destroyed
```

Поскольку переменная `ptr` является локальной переменной, то она уничтожается\
при завершении выполнения функции (независимо от того, как это будет сделано:\
досрочно или нет). И поскольку деструктор `Auto_ptr1` выполняет очистку `Item`, то мы\
можем быть уверены, что `Item` будет корректно удален.

### Критический недостаток
```c++
int main() {
    Auto_ptr1<Item> item1(new Item);
    Auto_ptr1<Item> item2(item1); // поверхностное копирование, так как нет своего конструктора

    return 0;
}

// Item acquired
// Item destroyed
// Item destroyed
```

Когда мы инициализируем `item2` значением `item1`, оба объекта класса `Auto_ptr1` указывают\
на один и тот же `Item`. Когда `item2` выходит из области видимости, он удаляет `Item`,\
оставляя `item1` с висячим указателем. Когда же `item1` отправляется на удаление\
своего (уже удаленного) `Item`, происходит «Бум!».

```c++
void passByValue(Auto_ptr1<Item> item) {}

int main() {
    Auto_ptr1<Item> item1(new Item);
    passByValue(item1);

    return 0;
}
// та же ситуация, что и выше
```

В этой программе `item1` передается по значению в параметр `item` функции\
`passByValue()`, что приведет к дублированию указателя `Item`.

### Семантика перемещения
**Семантика перемещения** означает, что класс, вместо копирования, передает право\
собственности на объект.

```c++
#include <iostream>

template<class T>
class Auto_ptr2 {
    T *m_ptr;
public:
    explicit Auto_ptr2(T *ptr = nullptr) : m_ptr{ptr} {}

    ~Auto_ptr2() { delete m_ptr; }

    // Конструктор копирования, который реализовывает семантику перемещения
    Auto_ptr2(Auto_ptr2 &a) { // примечание: Ссылка не является константной
        m_ptr = a.m_ptr; // перемещаем наш глупый указатель от источника к нашему локальному объекту
        a.m_ptr = nullptr; // подтверждаем, что источник больше не владеет указателем
    }

    // Оператор присваивания, который реализовывает семантику перемещения
    Auto_ptr2 &operator=(Auto_ptr2 &a) { // примечание: Ссылка не является константной
        if (&a == this)
            return *this;

        delete m_ptr; // подтверждаем, что удалили любой указатель, который наш локальный объект имел до этого
        m_ptr = a.m_ptr; // затем перемещаем наш глупый указатель из источника к нашему локальному объекту
        a.m_ptr = nullptr; // подтверждаем, что источник больше не владеет указателем
        return *this;
    }

    T &operator*() const { return *m_ptr; }
    T *operator->() const { return m_ptr; }
    bool isNull() const { return m_ptr == nullptr; }
};

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    Auto_ptr2<Item> item1(new Item); // Item acquired
    Auto_ptr2<Item> item2; // начнем с nullptr

    std::cout << "item1 is " << (item1.isNull() ? "null\n" : "not null\n"); // item1 is not null
    std::cout << "item2 is " << (item2.isNull() ? "null\n" : "not null\n"); // item2 is null

    item2 = item1; // item2 теперь является "владельцем" значения item1, объекту item1 присваивается null

    std::cout << "Ownership transferred\n";
    std::cout << "item1 is " << (item1.isNull() ? "null\n" : "not null\n"); // item1 is null
    std::cout << "item2 is " << (item2.isNull() ? "null\n" : "not null\n"); // item2 is not null
    
    return 0;
    // Item destroyed
}
```

Обратите внимание, перегруженный `operator=` передает право собственности на\
`m_ptr` от `item1` к `item2`! Следовательно, у нас не выполняется дублирования\
указателей, и всё аккуратно очищается (удаляется).

### `std::auto_ptr` и почему его лучше не использовать
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: `std::auto_ptr` устарел и не должен использоваться. Используйте вместо\
> него `std::unique_ptr` или `std::shared_ptr`.</b>

## [Урок №198. Ссылки r-value](#урок-198-ссылки-r-value)
### l-values и r-values
Каждое выражение в языке C++ имеет два свойства: **тип** и **категорию значения**.

О **l-value** проще всего думать, как о функции, объекте или переменной (или\
выражении, результатом которого является функция, объект или переменная),\
которая имеет свой адрес памяти.\
**l-values** разделены на две подкатегории:
* Модифицируемые l-values, которые можно изменить (например, переменной\
  `x` можно присвоить другое значение).
* Немодифицируемые l-values, которые являются `const` (например, константа `PI`).

О **r-value** проще всего думать, как «обо всем остальном, что не является l-value». Это\
литералы (например, `5`), временные значения (например, `x + 1`) и анонимные\
объекты (например, `Fraction(7, 3)`). r-values имеют область видимости\
выражения (уничтожаются в конце выражения, в котором находятся) и им нельзя\
что-либо присвоить.

Для поддержки семантики перемещения в C++11 ввели **3 новые категории\
значений**:
1. `pr-values`;
2. `x-values`;
3. `gl-values`.

### Ссылки l-value
**Ссылки l-value могут быть инициализированы только изменяемыми l-values.**

**Ссылки l-value на константные объекты могут быть инициализированы с помощью\
как l-values, так и r-values.**

### Ссылки r-value
**Ссылки r-value** - это ссылки, которые инициализируются только значениями r-values.

```c++
int x = 7;
int &lref = x; // инициализация ссылки l-value переменной x (значение l-value)
int &&rref = 7; // инициализация ссылки r-value литералом 7 (значение r-value)
```

**Ссылки r-value не могут быть инициализированы значениями l-values.**

Ссылки r-value имеют **два полезных свойства**:
1. Они увеличивают продолжительность жизни объекта, которым\
   инициализируются, до продолжительности жизни ссылки r-value (ссылки l-\
   value на константные объекты также могут это делать).
2. Неконстантные ссылки r-value позволяют нам изменять значения r-values, на\
   которые указывают ссылки r-value!

```c++
#include <iostream>

class Fraction {
private:
    int m_numerator;
    int m_denominator;

public:
    explicit Fraction(int numerator = 0, int denominator = 1) :
            m_numerator(numerator), m_denominator(denominator) {}

    friend std::ostream &operator<<(std::ostream &out, const Fraction &f1) {
        out << f1.m_numerator << "/" << f1.m_denominator;
        return out;
    }
};

int main() {
    Fraction &&rref = Fraction(4, 7); // ссылка r-value на анонимный объект класса Fraction
    std::cout << rref << '\n'; // 4/7

    return 0;
} // rref (и анонимный объект класса Fraction) выходит из области видимости здесь
```

Создаваемый анонимный объект `Fraction(4, 7)` обычно вышел бы из области\
видимости в конце выражения, в котором он определен. Однако, так как мы\
инициализируем ссылку r-value этим анонимным объектом, то его\
продолжительность жизни увеличивается до продолжительности жизни самой\
ссылки r-value, т.е. до конца функции `main()`. Затем мы используем ссылку r-value\
для вывода значения анонимного объекта класса `Fraction`.

```c++
#include <iostream>

int main() {
    int &&rref = 7; // поскольку мы инициализируем ссылку r-value литералом 7,
    // то создается временный объект со значением 7, на который указывает ссылка r-value
    rref = 12;
    std::cout << rref; // 12

    return 0;
}
```

Хотя это может показаться странным, но при инициализации ссылки r-value\
литералом, создается временный объект, на который ссылается ссылка r-value (она\
не ссылается на сам литерал).

### Ссылки r-value в качестве параметров функции
Ссылки r-value чаще всего используются в качестве параметров функции. Это\
наиболее полезно при перегрузке функций, когда вы хотите, чтобы выполнение\
функции отличалось в зависимости от аргументов (l-values или r-values):
```c++
#include <iostream>

void fun(const int &lref) { // перегрузка функции для работы с аргументами l-values
    std::cout << "l-value reference to const\n";
}

void fun(int &&rref) { // перегрузка функции для работы с аргументами r-values
    std::cout << "r-value reference\n";
}

int main() {
    int x = 7;
    fun(x); // аргумент l-value вызывает функцию с ссылкой l-value - l-value reference to const
    fun(7); // аргумент r-value вызывает функцию с ссылкой r-value - r-value reference

    return 0;
}
```

### Возврат ссылки r-value
Вы почти никогда **не должны возвращать ссылку r-value из функции** по той же\
причине, по которой вы почти никогда не должны возвращать ссылку l-value из\
функции. В большинстве случаев вы возвратите висячую ссылку (указывающую на\
удаленную память), а объект, на который будет ссылаться ссылка, выйдет из\
области видимости в конце функции.

### Тест
```c++
int main() {
    int x;

    // Ссылки l-value
    int &ref1 = x; // A
    int &ref2 = 7; // B - не скомпилируется

    const int &ref3 = x; // C
    const int &ref4 = 7; // D

    // Ссылки r-value
    int &&ref5 = x; // E - не скомпилируется
    int &&ref6 = 7; // F

    const int &&ref7 = x; // G - не скомпилируется
    const int &&ref8 = 7; // H

    return 0;
}
```

## [Урок №199. Конструктор перемещения и Оператор присваивания перемещением](#урок-199-конструктор-перемещения-и-оператор-присваивания-перемещением)
### Конструктор копирования и оператор присваивания копированием
**Конструктор копирования** используется для инициализации класса путем создания\
копии необходимого объекта. **Оператор присваивания копированием** (или\
**«копирующее присваивание»**) используется для копирования одного класса в\
другой (существующий) класс.

```c++
#include <iostream>

template<class T>
class Auto_ptr3 {
    T *m_ptr;
public:
    explicit Auto_ptr3(T *ptr = nullptr) : m_ptr(ptr) {}

    ~Auto_ptr3() { delete m_ptr; }

    // Конструктор копирования, который выполняет глубокое копирование x.m_ptr в m_ptr
    Auto_ptr3(const Auto_ptr3 &x) {
        m_ptr = new T;
        *m_ptr = *x.m_ptr;
    }

    // Оператор присваивания копированием, который выполняет глубокое копирование x.m_ptr в m_ptr
    Auto_ptr3 &operator=(const Auto_ptr3 &x) {
        // Проверка на самоприсваивание
        if (&x == this)
            return *this;

        // Удаляем всё, что к этому моменту может хранить указатель
        delete m_ptr;

        // Копируем передаваемый объект
        m_ptr = new T;
        *m_ptr = *x.m_ptr;

        return *this;
    }

    T &operator*() const { return *m_ptr; }
    T *operator->() const { return m_ptr; }
    bool isNull() const { return m_ptr == nullptr; }
};

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

Auto_ptr3<Item> generateItem() {
    Auto_ptr3<Item> item(new Item);
    return item; // это возвращаемое значение приведет к вызову конструктора копирования
}

int main() {
    Auto_ptr3<Item> mainItem;
    mainItem = generateItem(); // эта операция присваивания приведет к вызову оператора присваивания копированием

    return 0;
}

// Item acquired
// Item acquired
// Item destroyed
// Item destroyed
```

Используя конструктор копирования и оператор присваивания копированием\
с выполнением глубокого копирования мы, в итоге, выделяем и уничтожаем\
3 отдельных объекта, так как в функции создается и удаляется временный\
`item`.

### Конструктор перемещения и оператор присваивания перемещением
Определение **конструктора перемещения и оператора присваивания\
перемещением** выполняется аналогично определению конструктора копирования\
и оператора присваивания копированием.

Вот вышеприведенный класс `Auto_ptr3`, но уже с добавленными конструктором\
перемещения и оператором присваивания перемещением:
```c++
#include <iostream>

template<class T>
class Auto_ptr4 {
    T *m_ptr;
public:
    explicit Auto_ptr4(T *ptr = nullptr) : m_ptr(ptr) {}

    ~Auto_ptr4() { delete m_ptr; }

    // Конструктор копирования, который выполняет глубокое копирование x.m_ptr в m_ptr
    Auto_ptr4(const Auto_ptr4 &x) {
        m_ptr = new T;
        *m_ptr = *x.m_ptr;
    }

    // Конструктор перемещения, который передает право собственности на x.m_ptr в m_ptr
    Auto_ptr4(Auto_ptr4 &&x)  noexcept : m_ptr(x.m_ptr) {
        x.m_ptr = nullptr; // мы поговорим об этом чуть позже
    }

    // Оператор присваивания копированием, который выполняет глубокое копирование x.m_ptr в m_ptr
    Auto_ptr4 &operator=(const Auto_ptr4 &x) {
        // Проверка на самоприсваивание
        if (&x == this)
            return *this;

        // Удаляем всё, что к этому моменту может хранить указатель
        delete m_ptr;

        // Копируем передаваемый объект
        m_ptr = new T;
        *m_ptr = *x.m_ptr;

        return *this;
    }

    // Оператор присваивания перемещением, который передает право собственности на x.m_ptr в m_ptr
    Auto_ptr4 &operator=(Auto_ptr4 &&x)  noexcept {
        // Проверка на самоприсваивание
        if (&x == this)
            return *this;

        // Удаляем всё, что к этому моменту может хранить указатель
        delete m_ptr;

        // Передаем право собственности на x.m_ptr в m_ptr
        m_ptr = x.m_ptr;
        x.m_ptr = nullptr; // мы поговорим об этом чуть позже

        return *this;
    }

    T &operator*() const { return *m_ptr; }
    T *operator->() const { return m_ptr; }
    bool isNull() const { return m_ptr == nullptr; }
};

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

Auto_ptr4<Item> generateItem() {
    Auto_ptr4<Item> item(new Item);
    return item; // это возвращаемое значение приведет к вызову конструктора перемещения
}

int main() {
    Auto_ptr4<Item> mainItem;
    mainItem = generateItem(); // эта операция присваивания приведет к вызову оператора присваивания перемещением

    return 0;
}

// Item acquired
// Item destroyed
```

Вместо выполнения глубокого копирования исходного объекта в неявный объект,\
мы просто перемещаем (воруем) ресурсы исходного объекта. Под этим\
подразумевается поверхностное копирование указателя на исходный объект в\
неявный (временный) объект, а затем присваивание исходному указателю значения\
`null` (точнее `nullptr`) и в конце удаление неявного объекта.

### Когда вызываются конструктор перемещения и оператор присваивания перемещением?
Конструктор перемещения и оператор присваивания перемещением вызываются,\
когда **аргументом для создания или присваивания является r-value**. Чаще всего этим\
r-value будет литерал или временное значение (временный объект).

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: Если вам нужен конструктор перемещения и оператор присваивания\
> перемещением, которые выполняют перемещение (а не копирование), то вам их\
> нужно предоставить (написать) самостоятельно.</b>

### Ключевое понимание семантики перемещения
Если мы создаем объект или выполняем присваивание, где аргументом является l-\
value, то единственное разумное, что мы можем сделать — это скопировать l-value.

Однако, если мы создаем объект или выполняем присваивание, где аргументом\
является r-value, то мы знаем, что r-value — это просто некоторый временный\
объект. Вместо того, чтобы копировать его (что может быть затратно), мы можем\
просто переместить его ресурсы (что не так затратно) в другой объект, который мы\
создаем или которому присваиваем текущий. Это безопасно, поскольку временный\
объект будет уничтожен в конце выражения в любом случае, поэтому мы можем\
быть уверены, что он никогда не будет повторно использован!

В языке C++ есть правило, согласно которому а**втоматические объекты, возвращаемые\
функцией по значению, можно перемещать (а не копировать), даже если они являются\
l-values**.

### Отключение копирования
Желательно удалять конструктор копирования и оператор присваивания копированием,\
чтобы убедиться, что копирования объектов никогда не будет выполнено.
```c++
template<class T>
class Auto_ptr5 {
    T *m_ptr;
public:
    explicit Auto_ptr5(T *ptr = nullptr) : m_ptr(ptr) {}

    ~Auto_ptr5() { delete m_ptr; }

    // Конструктор копирования - запрещаем любое копирование!
    Auto_ptr5(const Auto_ptr5 &x) = delete;

    // Конструктор перемещения, который передает право собственности на x.m_ptr в m_ptr
    Auto_ptr5(Auto_ptr5 &&x)  noexcept : m_ptr(x.m_ptr) {
        x.m_ptr = nullptr;
    }

    // Оператор присваивания копированием - запрещаем любое копирование!
    Auto_ptr5 &operator=(const Auto_ptr5 &x) = delete;

    // Оператор присваивания перемещением, который передает право собственности на x.m_ptr в m_ptr
    Auto_ptr5 &operator=(Auto_ptr5 &&x)  noexcept {
        // Проверка на самоприсваивание
        if (&x == this)
            return *this;

        // Удаляем всё, что может хранить указатель до этого момента
        delete m_ptr;

        // Передаем право собственности на x.m_ptr в m_ptr
        m_ptr = x.m_ptr;
        x.m_ptr = nullptr;

        return *this;
    }

    T &operator*() const { return *m_ptr; }
    T *operator->() const { return m_ptr; }
    bool isNull() const { return m_ptr == nullptr; }
};
```

### Семантика копирования vs. Семантика перемещения
**Копирование:**
```c++
#include <iostream>
#include <chrono> // для функций из std::chrono

template<class T>
class DynamicArray {
private:
    T *m_array;
    int m_length;

public:
    DynamicArray(int length) : m_array(new T[length]), m_length(length) {}

    ~DynamicArray() { delete[] m_array; }

    // Конструктор копирования
    DynamicArray(const DynamicArray &arr) : m_length(arr.m_length) {
        m_array = new T[m_length];
        for (int i = 0; i < m_length; ++i)
            m_array[i] = arr.m_array[i];
    }

    // Оператор присваивания копированием
    DynamicArray &operator=(const DynamicArray &arr) {
        if (&arr == this)
            return *this;

        delete[] m_array;

        m_length = arr.m_length;
        m_array = new T[m_length];

        for (int i = 0; i < m_length; ++i)
            m_array[i] = arr.m_array[i];

        return *this;
    }

    int getLength() const { return m_length; }
    T &operator[](int index) { return m_array[index]; }
    const T &operator[](int index) const { return m_array[index]; }

};

class Timer {
private:
    // Используем псевдонимы типов для удобного доступа к вложенным типам
    using clock_t = std::chrono::high_resolution_clock;
    using second_t = std::chrono::duration<double, std::ratio<1> >;

    std::chrono::time_point<clock_t> m_beg;
public:
    Timer() : m_beg(clock_t::now()) {}

    void reset() { m_beg = clock_t::now(); }

    double elapsed() const {
        return std::chrono::duration_cast<second_t>(clock_t::now() - m_beg).count();
    }
};

// Возвращаем копию arr со значениями, умноженными на 2
DynamicArray<int> cloneArrayAndDouble(const DynamicArray<int> &arr) {
    DynamicArray<int> dbl(arr.getLength());
    for (int i = 0; i < arr.getLength(); ++i)
        dbl[i] = arr[i] * 2;

    return dbl;
}

int main() {
    Timer t;
    DynamicArray<int> arr(1000000);

    for (int i = 0; i < arr.getLength(); i++)
        arr[i] = i;

    arr = cloneArrayAndDouble(arr);

    std::cout << t.elapsed();
}
```
**Резултат: 0.0225438 с**


**Перемещение:**
```c++
#include <iostream>
#include <chrono> // для функций из std::chrono

template<class T>
class DynamicArray {
private:
    T *m_array;
    int m_length;

public:
    explicit DynamicArray(int length) : m_array(new T[length]), m_length(length) {}

    ~DynamicArray() { delete[] m_array; }

    // Конструктор копирования
    DynamicArray(const DynamicArray &arr) = delete;

    // Оператор присваивания копированием
    DynamicArray &operator=(const DynamicArray &arr) = delete;

    // Конструктор перемещения
    DynamicArray(DynamicArray &&arr)  noexcept : m_length(arr.m_length), m_array(arr.m_array) {
        arr.m_length = 0;
        arr.m_array = nullptr;
    }

    // Оператор присваивания перемещением
    DynamicArray &operator=(DynamicArray &&arr)  noexcept {
        if (&arr == this)
            return *this;

        delete[] m_array;

        m_length = arr.m_length;
        m_array = arr.m_array;
        arr.m_length = 0;
        arr.m_array = nullptr;

        return *this;
    }

    int getLength() const { return m_length; }
    T &operator[](int index) { return m_array[index]; }
    const T &operator[](int index) const { return m_array[index]; }
};

class Timer {
private:
    // Используем псевдонимы типов для удобного доступа к вложенным типам
    using clock_t = std::chrono::high_resolution_clock;
    using second_t = std::chrono::duration<double, std::ratio<1> >;

    std::chrono::time_point<clock_t> m_beg;
public:
    Timer() : m_beg(clock_t::now()) {}

    void reset() { m_beg = clock_t::now(); }

    double elapsed() const {
        return std::chrono::duration_cast<second_t>(clock_t::now() - m_beg).count();
    }
};

// Возвращаем копию arr со значениями, умноженными на 2
DynamicArray<int> cloneArrayAndDouble(const DynamicArray<int> &arr) {
    DynamicArray<int> dbl(arr.getLength());
    for (int i = 0; i < arr.getLength(); ++i)
        dbl[i] = arr[i] * 2;

    return dbl;
}

int main() {
    Timer t;
    DynamicArray<int> arr(1000000);

    for (int i = 0; i < arr.getLength(); i++)
        arr[i] = i;

    arr = cloneArrayAndDouble(arr);

    std::cout << t.elapsed();
}
```
**Результат: 0.0131518 с**

**Версия с использованием семантики перемещения была почти на 42%\
быстрее версии с использованием семантики копирования**

## [Урок №200. Функция `std::move()`](#урок-200-функция-stdmove)
### Функция `std::move()`
**Функция `std::move()`** — это стандартная библиотечная функция, которая\
конвертирует передаваемый аргумент в r-value. Мы можем передать l-value в\
функцию `std::move()`, и `std::move()` вернет нам ссылку r-value.

```c++
#include <iostream>
#include <string>
#include <utility>

template<class T>
void swap_elements(T &x, T &y) {
    T tmp{std::move(x)}; // вызывает конструктор перемещения
    x = std::move(y); // вызывает оператор присваивания перемещением
    y = std::move(tmp); // вызывает оператор присваивания перемещением
}

int main() {
    std::string x{"Anton"};
    std::string y{"Max"};

    std::cout << "x: " << x << '\n'; // Antor
    std::cout << "y: " << y << '\n'; // Max

    swap_elements(x, y);

    std::cout << "x: " << x << '\n'; // Max
    std::cout << "y: " << y << '\n'; // Anton

    return 0;
}
```

При инициализации `tmp`, вместо создания копии `x`, мы используем `std::move()` для\
конвертации переменной `x`, которая является l-value, в r-value. А поскольку\
параметром становится r-value, то с помощью семантики перемещения `x` перемещается\
в `tmp`.

### Еще один пример
```c++
#include <iostream>
#include <string>
#include <utility>
#include <vector>

int main() {
    std::vector<std::string> v;
    std::string str = "Bye";

    std::cout << "Copying str\n";
    v.push_back(str); // вызывает версию l-value метода push_back(), которая копирует str в элемент массива

    std::cout << "str: " << str << '\n'; // str: Bye
    std::cout << "vector: " << v[0] << '\n'; // vector: Bye

    std::cout << "\nMoving str\n";

    v.push_back(std::move(str)); // вызывает версию r-value метода push_back(), которая перемещает str в элемент массива

    std::cout << "str: " << str << '\n'; // str: (значение "украдено")
    std::cout << "vector: " << v[0] << ' ' << v[1] << '\n'; // vector: Bye Bye

    return 0;
}
```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: не используйте `std::move()` с любым «постоянным» объектом,\
> который вы не хотите изменять.</b>

### Резюме
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/success.svg">
>   <img alt="Success" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/success.svg">
> </picture><br>
>
> Функция `std::move()` может использоваться всякий раз, когда нужно обрабатывать l-\
> value как r-value с целью использования семантики перемещения вместо семантики\
> копирования.

## [Урок №201. Умный указатель `std::unique_ptr`](#урок-201-умный-указатель-stduniqueptr)
Стандартная библиотека в C++11 имеет **4 класса умных указателей**:
* `std::auto_ptr` (который не следует использовать — он удален в C++17);
* `std::unique_ptr`;
* `std::shared_ptr`;
* `std::weak_ptr`.

### Умный указатель std::unique_ptr
```c++
#include <iostream>
#include <memory> // для std::unique_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    // Выделяем объект класса Item и передаем право собственности на него std::unique_ptr
    std::unique_ptr<Item> item(new Item);

    return 0;
} // item выходит из области видимости здесь, соответственно, Item уничтожается также здесь

// Item acquired
// Item destroyed
```

Когда `std::unique_ptr` выходит из области видимости, он удаляет `Item`, которым\
владеет.

```c++
#include <iostream>
#include <memory> // для std::unique_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    std::unique_ptr<Item> item1(new Item); // выделение Item - Item acquired
    std::unique_ptr<Item> item2; // присваивается значение nullptr

    std::cout << "item1 is " << (static_cast<bool>(item1) ? "not null\n" : "null\n"); // item1 is not null
    std::cout << "item2 is " << (static_cast<bool>(item2) ? "not null\n" : "null\n"); // item2 is null

    // item2 = item1; // не скомпилируется: семантика копирования отключена
    item2 = std::move(item1); // item2 теперь владеет item1, а для item1 присваивается значение null

    std::cout << "Ownership transferred\n";

    std::cout << "item1 is " << (static_cast<bool>(item1) ? "not null\n" : "null\n"); // item1 is null
    std::cout << "item2 is " << (static_cast<bool>(item2) ? "not null\n" : "null\n"); // item2 is not null

    return 0;
} // Item уничтожается здесь, когда item2 выходит из области видимости - Item destroyed
```

### Доступ к объекту, который хранит умный указатель
Умный указатель `std::unique_ptr` имеет перегруженные операторы `*` и `->`, которые\
используются для доступа к хранимым объектам. Оператор `*` возвращает ссылку на\
управляемый ресурс, а оператор `->` возвращает указатель.

`std::unique_ptr` имеет неявное преобразование в тип `bool` для проверки, владеет\
ли он ресурсом, и возвращает `true`, если да:
```c++
#include <iostream>
#include <memory> // для std::unique_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }

    friend std::ostream &operator<<(std::ostream &out, const Item &item) {
        out << "I am an Item!\n";
        return out;
    }
};

int main() {
    std::unique_ptr<Item> item(new Item); // Item acquired

    if (item) // используем неявное преобразование item в тип bool, чтобы убедиться, что item владеет Item-ом
        std::cout << *item; // выводим Item, которым владеет item (разыменовываем) - I am an Item! 

    return 0;
} // Item destroyed
```

### Умный указатель `std::unique_ptr` и динамические массивы
В отличие от `std::auto_ptr`, `std::unique_ptr` достаточно умен, чтобы знать, когда\
использовать единичный оператор `delete`, а когда форму оператора `delete` для\
массива, поэтому `std::unique_ptr` можно использовать как с единичными объектами,\
так и с динамическими массивами.

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: Используйте `std::vector` вместо использования умного указателя,\
> который владеет динамическим массивом.</b>

### Функция `std::make_unique()`
В C++14 добавили новую функцию — `std::make_unique()`. Это шаблон функции,\
который создает объект типа шаблона и инициализирует его аргументами,\
переданными в функцию:
```c++
#include <iostream>
#include <memory> // для std::unique_ptr и std::make_unique

class Fraction {
private:
    int m_numerator = 0;
    int m_denominator = 1;

public:
    explicit Fraction(int numerator = 0, int denominator = 1) :
            m_numerator(numerator), m_denominator(denominator) {}

    friend std::ostream &operator<<(std::ostream &out, const Fraction &f1) {
        out << f1.m_numerator << "/" << f1.m_denominator;
        return out;
    }
};


int main() {
    // Создаем объект с динамически выделенным Fraction с numerator = 7 и denominator = 9
    std::unique_ptr<Fraction> f1 = std::make_unique<Fraction>(7, 9);
    std::cout << *f1 << '\n'; // 7/9

    // Создаем объект с динамически выделенным массивом Fraction длиной 5.
    // Используем автоматическое определение типа данных с помощью ключевого слова auto
    auto f2 = std::make_unique<Fraction[]>(5); // вместо std::unique_ptr<Fraction[]> f2(new Fraction[5]);
    std::cout << f2[0] << '\n'; // 0/1

    return 0;
}
```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: Используйте функцию `std::make_unique()` вместо создания умного\
> указателя `std::unique_ptr` и использования оператора `new`.</b>

### Возврат умного указателя `std::unique_ptr` из функции
Умный указатель `std::unique_ptr` можно возвращать из функции **по значению**:
```c++
std::unique_ptr<Item> createItem() {
    return std::make_unique<Item>();
}

int main() {
    std::unique_ptr<Item> ptr = createItem();

    // Делаем что-либо

    return 0;
}
```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: не возвращайте `std::unique_ptr` по адресу (вообще) или по\
> ссылке (если у вас нет на это веских причин).</b>

### Передача умного указателя `std::unique_ptr` в функцию
Если вы хотите, чтобы функция стала владельцем содержимого умного указателя, то\
передавать `std::unique_ptr` в функцию нужно по значению. Обратите внимание,\
поскольку семантика копирования отключена, то вам придется использовать\
`std::move()` для фактической передачи ресурса в функцию:
```c++
#include <iostream>
#include <memory> // для std::unique_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }

    friend std::ostream &operator<<(std::ostream &out, const Item &item) {
        out << "I am an Item!\n";
        return out;
    }
};

void takeOwnership(std::unique_ptr<Item> item) {
    if (item)
        std::cout << *item; // I am an Item!
} // Item уничтожается здесь - Item destroyed

int main() {
    auto ptr = std::make_unique<Item>(); // Item acquired

    // takeOwnership(ptr); // это не скомпилируется. Мы должны использовать семантику перемещения
    takeOwnership(std::move(ptr)); // используем семантику перемещения

    std::cout << "Ending program\n"; // Ending program

    return 0;
}
```

Однако лучше передавать сам объект по адресу или по ссылке (в зависимости\
от того, является ли `null` допустимым аргументом). Это позволит функции оставаться\
в стороне от управления объектом. Чтобы получить необработанный указатель на\
объект из `std::unique_ptr`, вы можете использовать метод `get()`:
```c++
#include <iostream>
#include <memory> // для std::unique_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }

    friend std::ostream &operator<<(std::ostream &out, const Item &item) {
        out << "I am an Item!\n";
        return out;
    }
};

// Эта функция использует только Item, поэтому мы передаем указатель на Item, 
// а не ссылку на весь std::unique_ptr<Item>
void useItem(Item *item) {
    if (item)
        std::cout << *item; // I am an Item!
}

int main() {
    auto ptr = std::make_unique<Item>(); // Item acquired

    useItem(ptr.get()); // примечание: Метод get() используется для получения указателя на Item

    std::cout << "Ending program\n"; // Ending program

    return 0;
} // Item уничтожается здесь - Item destroyed
```

### Умный указатель `std::unique_ptr` и классы
Конечно, вы можете использовать `std::unique_ptr` в качестве члена композиции\
вашего класса. Таким образом, вам не нужно будет беспокоиться о том, удалит ли\
деструктор вашего класса ресурс `std::unique_ptr`, так как `std::unique_ptr` будет\
автоматически уничтожен при уничтожении объекта класса. Тем не менее, если\
объект вашего класса выделяется динамически, то сам ресурс `std::unique_ptr`\
подвергается риску неправильного удаления, и в таком случае даже умный\
указатель не поможет.

### Неправильное использование умного указателя `std::unique_ptr`
1. Не позволяйте нескольким классам «владеть» одним и тем же ресурсом:
    ```c++
    Item *item = new Item;
    std::unique_ptr<Item> item1(item);
    std::unique_ptr<Item> item2(item);
    ```
   Хотя это синтаксически допустимо, конечным результатом будет то, что и `item1`, и\
   `item2` попытаются удалить `Item`, что приведет к неопределенному поведению/результатам.
2. Не удаляйте выделенный ресурс вручную из-под `std::unique_ptr`:
    ```c++
    Item *item = new Item;
    std::unique_ptr<Item> item1(item);
    delete item;
    ```
   Если вы это сделаете, `std::unique_ptr` попытается удалить уже удаленный ресурс, что\
   опять приведет к неопределенному поведению/результатам.

Обратите внимание, функция `std::make_unique()` предотвращает непреднамеренное\
возникновение обеих ситуаций, приведенных выше.

### Тест
**Задание №1.**\
_Если в вашем классе есть умный указатель в качестве члена вашего класса, то\
почему вы должны стараться избегать динамического выделения объектов этого\
класса?_

Умные указатели в качестве членов класса удаляют свой ресурс только в том\
случае, если объект класса выходит из области видимости. Если вы выделите\
объект класса динамически и не удалите его должным образом, то объект\
класса никогда не выйдет из области видимости, и умный указатель не сможет\
очистить ресурс, который он хранит.

**Задание №2.**
```c++
#include <iostream>
#include <memory> // для std::unique_ptr

class Fraction {
private:
    int m_numerator = 0;
    int m_denominator = 1;
public:
    explicit Fraction(int numerator = 0, int denominator = 1) :
            m_numerator(numerator), m_denominator(denominator) {
    }

    friend std::ostream &operator<<(std::ostream &out, const Fraction &f1) {
        out << f1.m_numerator << "/" << f1.m_denominator;
        return out;
    }
};

// Эта функция использует объект класса Fraction, поэтому мы только его и передаем.
// Таким образом, мы можем не беспокоиться о том, какой умный указатель
// использует caller (если вообще использует)
void printFraction(const Fraction *const ptr) {
    if (ptr)
        std::cout << *ptr;
}

int main() {
    auto ptr = std::make_unique<Fraction>(7, 9);
    printFraction(ptr.get()); // 7/9
    return 0;
}
```

## [Урок №202. Умный указатель `std::shared_ptr`](#урок-202-умный-указатель-stdsharedptr)
В отличие от `std::unique_ptr`, который предназначен для единоличного владения и\
управления переданным ему ресурсом/объектом, `std::shared_ptr` предназначен для\
случаев, когда несколько умных указателей совместно владеют одним динамически\
выделенным ресурсом.

### Умный указатель `std::shared_ptr`
Вы можете иметь несколько умных указателей `std::shared_ptr`, указывающих на\
один и тот же ресурс. Умный указатель `std::shared_ptr` отслеживает количество\
владельцев у каждого полученного ресурса. До тех пор, пока хотя бы один\
`std::shared_ptr` владеет ресурсом, этот ресурс не будет уничтожен, даже если\
удалить все остальные `std::shared_ptr` (которые также владеют этим ресурсом). Как\
только последний `std::shared_ptr`, владеющий ресурсом, выйдет из области\
видимости (или ему передадут другой ресурс для управления), тогда ресурс будет\
уничтожен.

```c++
#include <iostream>
#include <memory> // для std::shared_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    // Выделяем Item и передаем его в std::shared_ptr
    Item *item = new Item; // Item acquired
    std::shared_ptr<Item> ptr1(item);
    {
        std::shared_ptr<Item> ptr2{ptr1}; // используем копирующую инициализацию для
        // создания второго std::shared_ptr из ptr1, указывающего на тот же Item
        std::cout << "Killing one shared pointer\n";
    } // ptr2 выходит из области видимости здесь, но больше ничего не происходит

    std::cout << "Killing another shared pointer\n";

    return 0;
} // ptr1 выходит из области видимости здесь, и выделенный Item уничтожается также здесь - Item destroyed
```

**Обратите внимание, мы создали второй умный указатель из первого умного\
указателя (используя оператор присваивания копированием). Это важно.**

```c++
#include <iostream>
#include <memory> // для std::shared_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    Item *item = new Item; // Item acquired
    std::shared_ptr<Item> ptr1(item);
    {
        std::shared_ptr<Item> ptr2(item); // создаем ptr2 напрямую из item (вместо ptr1)

        std::cout << "Killing one shared pointer\n";
    } // ptr2 выходит из области видимости здесь, и выделенный Item уничтожается также здесь - Item destroyed

    std::cout << "Killing another shared pointer\n";

    return 0;
} // ptr1 выходит из области видимости здесь, и уже удаленный Item опять уничтожается здесь - Item destroyed

// Исключение: free(): double free detected in tcache 2
```

Разница здесь в том, что мы создали два отдельных, независимых друг от друга\
умных указателя `std::shared_ptr`. Хотя они оба указывают на один и тот же `Item`, они\
не знают о существовании друг друга. Когда `ptr2` выходит из области видимости,\
он думает, что является единственным владельцем `Item`-а, поэтому уничтожает его.\
Когда позже `ptr1` выходит из области видимости, он думает так же и пытается\
снова удалить (уже удаленный) `Item`. Бум!

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/warning.svg">
>   <img alt="Warning" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/warning.svg">
> </picture><br>
>
> <b>Правило: Всегда выполняйте копирование существующего `std::shared_ptr`, если\
> вам нужно более одного `std::shared_ptr`, указывающего на один и тот же
> динамически выделенный ресурс.</b>

### Функция `std::make_shared()`
Как функцию `std::make_unique()` можно использовать для создания `std::unique_ptr` в\
C++14, так и функцию `std::make_shared()` можно (и нужно) использовать для\
создания `std::shared_ptr`.
```c++
#include <iostream>
#include <memory> // для std::shared_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    // Выделяем Item и передаем его в std::shared_ptr
    auto ptr1 = std::make_shared<Item>(); // Item acquired
    {
        auto ptr2 = ptr1; // создаем ptr2 из ptr1, используя семантику копирования

        std::cout << "Killing one shared pointer\n";
    } // ptr2 выходит из области видимости здесь, но ничего больше не происходит

    std::cout << "Killing another shared pointer\n";

    return 0;
} // ptr1 выходит из области видимости здесь, и выделенный Item также уничтожается здесь - Item destroyed
```

### Детали реализации умного указателя `std::shared_ptr`
В отличие от `std::unique_ptr`, который использует внутри ("под капотом") один\
указатель, `std::shared_ptr` использует внутри два указателя. Один указывает на\
передаваемый ресурс, а второй — на **«блок управления»** — динамически\
выделенный объект, который отслеживает кучу разных вещей, включая и то,\
сколько умных указателей `std::shared_ptr` одновременно указывают на каждый\
полученный ресурс. При создании `std::shared_ptr` с помощью конструктора\
`std::shared_ptr`, память для полученного ресурса и блока управления (который также\
создает конструктор) выделяется отдельно. Однако в `std::make_shared()` это\
оптимизировано в единое выделение памяти, что, соответственно, повышает\
производительность.

Это также объясняет то, почему независимое создание двух `std::shared_ptr`,\
указывающих на один и тот же ресурс, приводит к проблемам. Каждый\
`std::shared_ptr` имеет один указатель, указывающий на полученный ресурс. Однако\
каждый `std::shared_ptr` еще и независимо выделяет свой собственный блок\
управления, который сообщает указателю, что он является единственным\
«владельцем» полученного ресурса (даже если это не так). Таким образом, когда\
данный `std::shared_ptr` выходит из области видимости, он уничтожает ресурс,\
которым владеет, не осознавая того, что могут быть еще другие умные указатели\
`std::shared_ptr`, которые владеют этим же ресурсом.

Однако, когда `std::shared_ptr` клонируется с использованием семантики\
копирования, данные в блоке управления соответствующим образом обновляются\
и говорят о том, что появился еще один `std::shared_ptr`, указывающий на\
полученный ресурс.

### Создание `std::shared_ptr` из `std::unique_ptr`
Умный указатель `std::unique_ptr` может быть конвертирован в умный указатель\
`std::shared_ptr` через специальный конструктор `std::shared_ptr`, который принимает\
`std::unique_ptr` в качестве r-value. Таким образом, содержимое `std::unique_ptr`\
перемещается в `std::shared_ptr`.\
Однако `std::shared_ptr` нельзя безопасно конвертировать в `std::unique_ptr`.

### Резюме
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/success.svg">
>   <img alt="Success" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/success.svg">
> </picture><br>
>
> Умный указатель `std::shared_ptr` дает возможность сразу нескольким умным\
> указателям управлять одним динамически выделенным ресурсом. Ресурс\
> уничтожается лишь в том случае, когда уничтожены все `std::shared_ptr`,\
> указывающие на него.

## [Урок №203. Умный указатель `std::weak_ptr`](#урок-203-умный-указатель-stdweakptr)
### Пересечение умных указателей
```c++
#include <iostream>
#include <memory> // для std::shared_ptr
#include <string>
#include <utility>

class Human {
    std::string m_name;
    std::shared_ptr<Human> m_partner; // изначально пустой

public:
    Human(std::string name) : m_name(std::move(name)) { std::cout << m_name << " created\n"; }
    ~Human() { std::cout << m_name << " destroyed\n"; }

    friend bool partnerUp(std::shared_ptr<Human> &h1, std::shared_ptr<Human> &h2) {
        if (!h1 || !h2)
            return false;

        h1->m_partner = h2;
        h2->m_partner = h1;

        std::cout << h1->m_name << " is now partnered with " << h2->m_name << "\n";

        return true;
    }
};

int main() {
    // создание умного указателя с объектом Anton класса Human
    auto anton = std::make_shared<Human>("Anton"); // Anton created
    // создание умного указателя с объектом Ivan класса Human
    auto ivan = std::make_shared<Human>("Ivan"); // Ivan created
    
    // Anton указывает на Ivan-а, а Ivan указывает на Anton-а 
    partnerUp(anton, ivan); // Anton is now partnered with Ivan

    return 0;
}
```

Здесь мы динамически выделяем два объекта (`Anton` и `Ivan`) класса `Human` и,\
используя `std::make_shared`, передаем их в два отдельно созданных умных\
указателя типа `std::shared_ptr`. Затем «связываем» их с помощью дружественной\
функции `partnerUp()`. Но никаких уничтожений не происходит...

После вызова функции `partnerUp()` у нас образуется 4 умных указателя типа\
`std::shared_ptr`:
* Два умных указателя указывают на объект `Ivan`: `ivan` (из функции `main()`) и\
  `m_partner` (из класса `Human`) объекта `Anton`.
* Два умных указателя указывают на объект `Anton`: `anton` и `m_partner`
  объекта `Ivan`.

В конце функции `partnerUp()` умный указатель `ivan` выходит из области видимости\
первым. Когда это происходит, он проверяет, есть ли другие умные указатели,\
которые владеют объектом `Ivan` класса `Human`. Есть — `m_partner` объекта `Anton`.\
Из-за этого умный указатель не уничтожает `Ivan`-а (если он это сделает, то\
`m_partner` объекта `Anton` останется висячим указателем). Таким образом у нас\
остается один умный указатель, владеющий `Ivan`-ом (`m_partner` объекта `Anton`)\
и два умных указателя, владеющие `Anton`-ом (`anton` и `m_partner` объекта `Ivan`).

Затем умный указатель `anton` выходит из области видимости, и происходит то же\
самое. `anton` проверяет, есть ли другие умные указатели, которые также владеют\
объектом `Anton` класса `Human`. Есть — `m_partner` объекта `Ivan`, поэтому объект\
`Anton` не уничтожается. Таким образом, остаются два умных указателя:
* `m_partner` объекта `Ivan`, который указывает на `Anton`-а;
* `m_partner` объекта `Anton`, который указывает на `Ivan`-а.

Затем программа завершает свое выполнение, и ни объект `Anton`, ни объект\
`Ivan` не уничтожаются! По сути, `Anton` не дает уничтожить `Ivan`-а, а `Ivan` не дает\
уничтожить `Anton`-а.

Это тот случай, когда умные указатели типа `std::shared_ptr` формируют **циклическую\
зависимость**.

### Циклическая зависимость
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> **Циклическая зависимость** (или **«циклические ссылки»**) — это серия «ссылок», где\
> текущий объект ссылается на следующий, а последний объект ссылается на первый.

**Практическая ценность** такой циклической зависимости в том, что текущий объект\
«оставляет в живых» (не дает уничтожить) следующий объект.

### Упрощенная циклическая зависимость
Оказывается, проблема циклической ссылки может возникнуть даже с одним\
умным указателем типа `std::shared_ptr`. Такая **циклическая зависимость называется\
упрощенной**. Хотя это редко случается на практике, но все же рассмотрим и этот\
случай:
```c++
#include <iostream>
#include <memory> // для std::shared_ptr

class Item {
public:
    std::shared_ptr<Item> m_ptr; // изначально пустой
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    auto ptr1 = std::make_shared<Item>(); // Item acquired
    ptr1->m_ptr = ptr1; // m_ptr теперь является владельцем Item-а, членом которого он является сам

    return 0;
}
```

Когда `ptr1` выходит из области видимости, он не уничтожает `Item`, поскольку\
член `m_ptr` класса `Item` также владеет `Item`-ом. Таким образом, не остается\
никого, кто мог бы удалить `Item` (`m_ptr` никогда не выходит из области\
видимости, поэтому он этого не сделает).

### Так что же такое умный указатель `std::weak_ptr`?
Умный указатель `std::weak_ptr` был разработан для решения проблемы\
«циклической зависимости», описанной выше. **`std::weak_ptr` является\
наблюдателем — он может наблюдать и получать доступ к тому же объекту, на\
который указывает `std::shared_ptr` (или другой `std::weak_ptr`), но не считаться\
владельцем этого объекта**. Помните, когда `std::shared_ptr` выходит из области\
видимости, он проверяет, есть ли другие владельцы `std::shared_ptr`. `std::weak_ptr`\
владельцем не считается!

```c++
#include <iostream>
#include <memory> // для std::shared_ptr и std::weak_ptr
#include <string>
#include <utility>

class Human {
    std::string m_name;
    std::weak_ptr<Human> m_partner; // обратите внимание, здесь std::weak_ptr
public:
    explicit Human(std::string name) : m_name(std::move(name)) {
        std::cout << m_name << " created\n";
    }
    
    ~Human() { std::cout << m_name << " destroyed\n"; }

    friend bool partnerUp(std::shared_ptr<Human> &h1, std::shared_ptr<Human> &h2) {
        if (!h1 || !h2)
            return false;

        h1->m_partner = h2;
        h2->m_partner = h1;

        std::cout << h1->m_name << " is now partnered with " << h2->m_name << "\n";

        return true;
    }
};

int main() {
    auto anton = std::make_shared<Human>("Anton"); // Anton created
    auto ivan = std::make_shared<Human>("Ivan"); // Ivan created

    partnerUp(anton, ivan); // Anton is now partnered with Ivan

    return 0;
}

// Ivan destroyed
// Anton destroyed
```

Теперь, когда `ivan` выходит из области видимости, он видит, что нет другого\
`std::shared_ptr`, указывающего на `Ivan`-а (`std::weak_ptr` из `Anton`-а не считается).\
Поэтому он уничтожает `Ivan`-а. То же самое происходит и с `Anton`-ом.

### Использование умного указателя `std::weak_ptr`
Недостатком умного указателя `std::weak_ptr` является то, что его нельзя\
использовать напрямую (нет оператора `->`). Чтобы использовать `std::weak_ptr`, вы\
сначала должны конвертировать его в `std::shared_ptr` (с помощью **метода `lock()`**), а\
затем уже использовать `std::shared_ptr`:
```c++
#include <iostream>
#include <memory> // для std::shared_ptr и std::weak_ptr
#include <string>
#include <utility>

class Human {
    std::string m_name;
    std::weak_ptr<Human> m_partner; // обратите внимание, здесь std::weak_ptr

public:

    explicit Human(std::string name) : m_name(std::move(name)) {
        std::cout << m_name << " created\n";
    }

    ~Human() { std::cout << m_name << " destroyed\n"; }

    friend bool partnerUp(std::shared_ptr<Human> &h1, std::shared_ptr<Human> &h2) {
        if (!h1 || !h2)
            return false;

        h1->m_partner = h2;
        h2->m_partner = h1;

        std::cout << h1->m_name << " is now partnered with " << h2->m_name << "\n";

        return true;
    }

    // используем метод lock() для конвертации std::weak_ptr в std::shared_ptr
    std::shared_ptr<Human> getPartner() const { return m_partner.lock(); }
    const std::string &getName() const { return m_name; }
};

int main() {
    auto anton = std::make_shared<Human>("Anton"); // Anton created
    auto ivan = std::make_shared<Human>("Ivan"); // Ivan created
    partnerUp(anton, ivan); // Anton is now partnered with Ivan

    // передаем partner-у содержимое умного указателя, которым владеет ivan
    auto partner = ivan->getPartner();
    std::cout << ivan->getName() << "'s partner is: " << partner->getName() << '\n'; // Ivan's partner is: Anton

    return 0;
}
// Ivan destroyed
// Anton destroyed
```

Нам не нужно беспокоиться о циклической зависимости с переменной\
`partner` (типа `std::shared_ptr`), так как она является простой локальной\
переменной внутри функции `main()` и уничтожается при завершении выполнения\
функции `main()`.

### Резюме
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/success.svg">
>   <img alt="Success" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/success.svg">
> </picture><br>
>
> Умный указатель `std::shared_ptr` используется для владения одним динамически\
> выделенным ресурсом сразу несколькими умными указателями. Ресурс будет\
> уничтожен, когда последний std::shared_ptr выйдет из области видимости.\
> std::weak_ptr используется, когда нужен умный указатель, который имеет доступ к\
> ресурсу, но не считается его владельцем.

### Тест 
```c++
#include <iostream>
#include <memory> // для std::shared_ptr и std::weak_ptr

class Item {
public:
    std::weak_ptr<Item> m_ptr; // используем std::weak_ptr, чтобы m_ptr не поддерживал Item-а
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    auto ptr1 = std::make_shared<Item>(); // Item acquired
    ptr1->m_ptr = ptr1; // m_ptr теперь является владельцем Item-а, членом которого он является сам
    return 0;
}
// Item destroyed
```

## [Глава №15. Итоговый тест](#глава-15-итоговый-тест)
### Теория
**Класс умного указателя** — это класс, предназначенный для управления\
динамически выделенной памятью и для её удаления, когда объект умного\
указателя выходит из области видимости.

**Семантика копирования** позволяет копировать классы с помощью конструктора\
копирования и оператора присваивания копированием.

**Семантика перемещения** позволяет классу передать владение\
ресурсами/объектом с помощью конструктора перемещения и оператора\
присваивания перемещением другому объекту (без выполнения копирования).

**Умный указатель `std::auto_ptr`** устарел и его следует избегать.

**Ссылка r-value** — это ссылка, которая инициализируется значениями r-values.\
Ссылка r-value создается с использованием двойного амперсанда. Помните, что\
писать функции, которые принимают в качестве параметров ссылки r-value —\
хорошо, но возвращать ссылки r-value — плохо.

Если мы создаем объект или выполняем операцию присваивания, где **аргументом\
является l-value**, то единственное разумное, что мы можем сделать — это\
скопировать l-value. Дело в том, что не всегда безопасно изменять l-value, так как он\
еще может быть использован позже в программе. Если у нас есть выражение\
`a = b`, то мы предполагаем, что `b` не будет каким-либо образом изменен позже.

Однако, если мы создаем объект или выполняем операцию присваивания, где\
**аргументом является r-value**, то мы знаем, что r-value — это всего лишь временный\
объект. Вместо того, чтобы копировать его (что может быть затратно), мы можем\
просто переместить его ресурсы (что не очень затратно) в объект, который создаем\
или которому присваиваем. Это безопасно, поскольку временный объект в любом\
случае будет уничтожен в конце выражения, и мы можем быть уверены, что он\
больше никогда не будет использован снова!

Вы можете использовать ключевое слово `delete` для **отключения семантики\
копирования** в создаваемых классах, удалив конструктор копирования и оператор\
присваивания копированием.

**Функция `std::move()`** позволяет конвертировать l-value в r-value. Это полезно, когда\
вы хотите использовать семантику перемещения вместо семантики копирования с\
аргументом, который изначально является l-value.

**Умный указатель `std::unique_ptr`** — это класс умного указателя, который\
единолично владеет переданным ему динамически выделенным ресурсом.\
**Функция `std::make_unique()`** — это функция, добавленная в C++14, которую вы\
должны использовать для создания нового `std::unique_ptr`. В `std::unique_ptr`\
семантика копирования по умолчанию отключена.

**Умный указатель `std::shared_ptr`** — это класс умного указателя, который следует\
использовать, когда нужно, чтобы одним динамически выделенным ресурсом\
владели сразу несколько умных указателей. Ресурс не будет уничтожен до тех пор,\
пока не будет уничтожен последний владеющий им `std::shared_ptr`. Для создания\
нового `std::shared_ptr` предпочтительнее использовать **функцию `std::make_shared(`)**.\
Для создания дополнительного `std::shared_ptr`, который бы указывал на текущий\
ресурс, вы должны использовать семантику копирования.

**Умный указатель `std::weak_ptr`** — это класс умного указателя, который вы должны\
использовать, когда вам нужен один или несколько указателей с возможностью\
просмотра и доступа к динамически выделенному ресурсу, которым управляет\
другой `std::shared_ptr`, но, чтобы этот указатель не участвовал в уничтожении\
ресурса. `std::weak_ptr` владельцем ресурса не считается.

### Тест
**Задание №1.**\
_Объясните, когда следует использовать следующие типы умных указателей:_
1. **Умный указатель `std::unique_ptr`**:\
   Умный указатель `std::unique_ptr` следует использовать, когда нужно, чтобы\
   умный указатель единолично владел динамически выделенным ресурсом.
2. **Умный указатель std::shared_ptr**:\
   Умный указатель `std::shared_ptr` следует использовать, когда нужно, чтобы сразу\
   несколько умных указателей владело одним динамически выделенным ресурсом.\
   Ресурс не будет уничтожен до тех пор, пока не будут уничтожены все\
   `std::shared_ptr`, владеющие им.
3. **Умный указатель `std::weak_ptr`**\
   Умный указатель `std::weak_ptr` следует использовать, когда нужно иметь доступ\
   к ресурсу, которым управляет другой умный указатель `std::shared_ptr`, но\
   продолжительности жизни этих двух умных указателей не должны быть связаны\
   между собой.
4. **Умный указатель std::`auto_ptr`**:\
   Умный указатель `std::auto_ptr` устарел, и его использования следует избегать.

**Задание №2.**\
_Объясните, как ссылки r-value участвуют в реализации семантики перемещения._\

Поскольку значения r-values являются временными, то мы знаем, что они будут\
уничтожены сразу же после их использования. При передаче или возврате r-\
value по значению менее эффективным будет выполнять копирование, а затем\
уничтожать оригинал. Вместо этого гораздо эффективнее будет просто\
переместить (передать) ресурсы r-value в нужный объект.

**Задание №3.**
_Что не так со следующими программами? Обновите их в соответствии с\
рекомендациями, полученными в этой главе._\
**a)**
```c++
#include <iostream>
#include <memory> // для std::shared_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    Item *item = new Item;
    std::shared_ptr<Item> ptr1(item);
    std::shared_ptr<Item> ptr2(item);

    return 0;
}
```

`ptr2` был создан из `item`-а, а не из `ptr1`. Это означает, что теперь у нас есть два\
отдельных умных указателя `std::shared_ptr`, каждый из которых пытается\
независимо управлять `Item`-ом (они не знают о существовании друг друга). Когда\
один из этих умных указателей выйдет из области видимости, то другой\
останется висячим указателем.\
`ptr2` должен быть скопирован из `ptr1`, и для создания `std::shared_ptr` следует\
использовать функцию `std::make_shared()`:
```c++
#include <iostream>
#include <memory> // для std::shared_ptr

class Item {
public:
    Item() { std::cout << "Item acquired\n"; }
    ~Item() { std::cout << "Item destroyed\n"; }
};

int main() {
    auto ptr1 = std::make_shared<Item>();
    auto ptr2(ptr1);
    return 0;
}
```

**b)**
```c++
#include <iostream>
#include <memory> // для std::shared_ptr

class Something; // предположим, что Something - это класс, который может выбросить исключение

int main() {
    doSomething(std::shared_ptr<Something>(new Something), std::shared_ptr<Something>(new Something));

    return 0;
}
```

Если конструктор класса `Something` выбросит исключение, то один из объектов\
класса `Something`, вполне вероятно, не будет уничтожен должным образом.\
Решение заключается в использовании функции `std::make_shared()`:
```c++
#include <iostream>
#include <memory> // для std::shared_ptr

class Something; // предположим, что Something - это класс, который может выбросить исключение
int main() {
    doSomething(std::make_shared<Something>(), std::make_shared<Something>());
    return 0;
}
```