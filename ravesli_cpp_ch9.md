# Глава №9.

## Оглавление

1. [Урок №138. Введение в перегрузку операторов](#урок-138-введение-в-перегрузку-операторов)
2. [Урок №139. Перегрузка операторов через дружественные функции](#урок-139-перегрузка-операторов-через-дружественные-функции)
3. [Урок №140. Перегрузка операторов через обычные функции](#урок-140-перегрузка-операторов-через-обычные-функции)
4. [Урок №141. Перегрузка операторов ввода и вывода](#урок-141-перегрузка-операторов-ввода-и-вывода)
5. [Урок №142. Перегрузка операторов через методы класса](#урок-142-перегрузка-операторов-через-методы-класса)
6. [Урок №143. Перегрузка унарных операторов +, - и логического НЕ](#урок-143-перегрузка-унарных-операторов----и-логического-не)
7. [Урок №144. Перегрузка операторов сравнения](#урок-144-перегрузка-операторов-сравнения)
8. [Урок №145. Перегрузка операторов инкремента и декремента](#урок-145-перегрузка-операторов-инкремента-и-декремента)
9. [Урок №146. Перегрузка оператора индексации `[]`](#урок-146-перегрузка-оператора-индексации-)
10. [Урок №147. Перегрузка оператора `()`](#урок-147-перегрузка-оператора-)
11. [Урок №148. Перегрузка операций преобразования типов данных](#урок-148-перегрузка-операций-преобразования-типов-данных)

## [Урок №138. Введение в перегрузку операторов](#урок-138-введение-в-перегрузку-операторов)

### Ограничения в перегрузке операторов

Почти любой существующий оператор в языке C++ может быть перегружен.\
**Исключениями являются:**

* тернарный оператор (`?:`);
* оператор `sizeof`;
* оператор разрешения области видимости (`::`);
* оператор выбора члена (`.`);
* указатель, как оператор выбора члена (`.*`).

**Вы можете перегрузить только существующие операторы.**

**По крайней мере один из операндов перегруженного оператора\
должен быть пользовательского типа данных.**

**Изначальное количество операндов, поддерживаемых оператором,\
изменить невозможно.**

**Все операторы сохраняют свой приоритет и ассоциативность по\
умолчанию**

**Правило: При перегрузке операторов старайтесь максимально приближенно\
сохранять функционал операторов в соответствии с их первоначальными\
применениями.**

## [Урок №139. Перегрузка операторов через дружественные функции](#урок-139-перегрузка-операторов-через-дружественные-функции)

**Есть три разных способа перегрузки операторов:**

* через дружественные функции;
* через обычные функции;
* через методы класса.

### Перегрузка операторов через дружественные функции

Перегружаем операторы `+` и `-` для выполнения операций сложения и вычитания\
двух объектов:

```c++
#include <iostream>

class Dollars {
private:
    int m_dollars;
public:
    Dollars(int dollars) { m_dollars = dollars; }

    // Выполняем Dollars + Dollars через дружественную функцию
    friend Dollars operator+(const Dollars &d1, const Dollars &d2);

    friend Dollars operator-(const Dollars &d1, const Dollars &d2);

    int getDollars() const { return m_dollars; }
};

// Примечание: Эта функция не является методом класса!
Dollars operator+(const Dollars &d1, const Dollars &d2) {
    // Используем конструктор Dollars и operator+(int, int).
    // Мы имеем доступ к закрытому члену m_dollars, поскольку эта функция
    // является дружественной классу Dollars
    return {d1.m_dollars + d2.m_dollars};
}

Dollars operator-(const Dollars &d1, const Dollars &d2) {
    return {d1.m_dollars - d2.m_dollars};
}

int main() {
    Dollars dollars1(7);
    Dollars dollars2(9);
    Dollars dollarsSum = dollars1 + dollars2;
    std::cout << "I have " << dollarsSum.getDollars() << " dollars."
              << std::endl; // I have 16 dollars.
    Dollars dollarsSub {dollarsSum - dollars2};
    std::cout << "I paid " << dollarsSub.getDollars() << " dollars."; // I paid 7 dollars.
    return 0;
}
```

### Дружественные функции могут быть определены внутри класса

Несмотря на то, что дружественные функции не являются членами класса, они по-\
прежнему могут быть определены внутри класса, если это необходимо:

```c++
#include <iostream>

class Dollars {
private:
    int m_dollars;
public:
    Dollars(int dollars) { m_dollars = dollars; }

    // Эта функция не рассматривается как метод класса, хотя она и
    // определена внутри класса
    friend Dollars operator+(const Dollars &d1, const Dollars &d2) {
        return {d1.m_dollars + d2.m_dollars};
    }

    friend Dollars operator-(const Dollars &d1, const Dollars &d2) {
        return {d1.m_dollars - d2.m_dollars};
    }

    int getDollars() const { return m_dollars; }
};

int main() {
    Dollars dollars1(7);
    Dollars dollars2(9);
    Dollars dollarsSum = dollars1 + dollars2;
    std::cout << "I have " << dollarsSum.getDollars() << " dollars."
              << std::endl; // I have 16 dollars.
    Dollars dollarsSub {dollarsSum - dollars2};
    std::cout << "I paid " << dollarsSub.getDollars() << " dollars."; // I paid 7 dollars.
    return 0;
}
```

**Не рекомендуется так делать, поскольку нетривиальные определения функций\
лучше записывать в отдельном файле .cpp вне тела класса.**

### Перегрузка операторов с операндами разных типов

**При перегрузке бинарных операторов для работы с операндами разных типов,\
нужно писать две функции — по одной на каждый случай:**

```c++
#include <iostream>

class Dollars {
private:
    int m_dollars;
public:
    Dollars(int dollars) { m_dollars = dollars; }

    // Выполняем Dollars + int через дружественную функцию
    friend Dollars operator+(const Dollars &d1, int value);

    // Выполняем int + Dollars через дружественную функцию
    friend Dollars operator+(int value, const Dollars &d1);

    int getDollars() const { return m_dollars; }
};

// Примечание: Эта функция не является методом класса!
Dollars operator+(const Dollars &d1, int value) {
    return {d1.m_dollars + value};
}

// Примечание: Эта функция не является методом класса!
Dollars operator+(int value, const Dollars &d1) {
    return {d1.m_dollars + value};
}


int main() {
    Dollars dollars1{Dollars(5) + 5};
    Dollars dollars2{Dollars(5) + 5};
    std::cout << "I have " << dollars1.getDollars() << " dollars."
              << std::endl; // I have 10 dollars.
    std::cout << "I have " << dollars2.getDollars() << " dollars."
              << std::endl; // I have 10 dollars.
    return 0;
}
```

### Тест

```c++
#include <iostream>

class Fraction {
private:
    int m_numerator;
    int m_denominator;
public:
    Fraction(int numerator = 0, int denominator = 1) :
            m_numerator{numerator}, m_denominator{denominator} {
        // Мы поместили метод reduce() в конструктор, чтобы убедиться, что все
        //дроби, которые у нас есть, будут уменьшены!
        // Поскольку выполнение всех перегруженных операторов осуществляется
        //вместе с созданием новых объектов класса Fraction, то мы можем гарантировать,
        //что эта функция вызовется для каждой дроби
        reduce();
    }

    // Делаем функцию nod() статической, чтобы она могла быть частью класса
    //Fraction и, при этом, для её использования нам не нужно было бы создавать
    //объект класса Fraction
    static int nod(int a, int b) {
        return (b == 0) ? (a > 0 ? a : -a) : nod(b, a % b);
    }

    // Упрощаем дробь
    void reduce() {
        int nod = Fraction::nod(m_numerator, m_denominator);
        m_numerator /= nod;
        m_denominator /= nod;
    }

    friend Fraction operator*(const Fraction &f1, const Fraction &f2);

    friend Fraction operator*(const Fraction &f1, int value);

    friend Fraction operator*(int value, const Fraction &f1);

    void print() {
        std::cout << m_numerator << "/" << m_denominator << "\n";
    }
};

Fraction operator*(const Fraction &f1, const Fraction &f2) {
    return Fraction(f1.m_numerator * f2.m_numerator, f1.m_denominator *
                                                     f2.m_denominator);
}

Fraction operator*(const Fraction &f1, int value) {
    return Fraction(f1.m_numerator * value, f1.m_denominator);
}

Fraction operator*(int value, const Fraction &f1) {
    return Fraction(f1.m_numerator * value, f1.m_denominator);
}

int main() {
    Fraction f1(3, 4);
    f1.print(); // 3/4
    Fraction f2(2, 7);
    f2.print(); // 2/7
    Fraction f3 = f1 * f2;
    f3.print(); // 3/14
    Fraction f4 = f1 * 3;
    f4.print(); // 9/4
    Fraction f5 = 3 * f2;
    f5.print(); // 6/7
    Fraction f6 = Fraction(1, 2) * Fraction(2, 3) * 
            Fraction(3, 4);
    f6.print(); // 1/4
    return 0;
}
```

## [Урок №140. Перегрузка операторов через обычные функции](#урок-140-перегрузка-операторов-через-обычные-функции)

**Dollars.h:**

```c++
class Dollars {
private:
int m_dollars;
public:
Dollars(int dollars) { m_dollars = dollars; }

int getDollars() const { return m_dollars; }
};

// Указываем прототип operator+(), чтобы иметь возможность использовать
// перегруженный оператор + в других файлах
Dollars operator+(const Dollars &d1, const Dollars &d2);
```

**Dollars.cpp:**

```c++
#include "Dollars.h"

// Примечание: Эта функция не является ни методом класса, ни дружественной
//классу Dollars!
Dollars operator+(const Dollars &d1, const Dollars &d2) {
    // Используем конструктор Dollars и operator+(int, int).
    // Здесь нам не нужен прямой доступ к закрытым членам класса Dollars
    return {d1.getDollars() + d2.getDollars()};
}
```

**main.cpp:**

```c++
#include <iostream>
#include "Dollars.h"

int main() {
    Dollars dollars1(7);
    Dollars dollars2(9);
    Dollars dollarsSum = dollars1 + dollars2; // без явного указания
    // прототипа operator+() в Dollars.h эта строка не скомпилировалась бы
    std::cout << "I have " << dollarsSum.getDollars() << " dollars." <<
              std::endl; // I have 16 dollars.
    return 0;
}
```

**Правило: Используйте перегрузку операторов через обычные функции, вместо\
дружественных, если для этого не требуется добавление дополнительных\
функций в класс.**

## [Урок №141. Перегрузка операторов ввода и вывода](#урок-141-перегрузка-операторов-ввода-и-вывода)

### Перегрузка оператора вывода `<<`

```c++
#include <iostream>

class Point {
private:
    double m_x, m_y, m_z;
public:
    Point(double x = 0.0, double y = 0.0, double z = 0.0) : m_x(x), m_y(y), m_z(z) {}

    friend std::ostream &operator<<(std::ostream &out, const Point &point);
};

std::ostream &operator<<(std::ostream &out, const Point &point) {
    // Поскольку operator<< является другом класса Point, то мы имеем прямой
    // доступ к членам Point
    out << "Point(" << point.m_x << ", " << point.m_y << ", " << point.m_z << ")";
    return out;
}

int main() {
    Point point1(5.0, 6.0, 7.0);
    std::cout << point1;

    return 0;
}
```

### Перегрузка оператора ввода `>>`

```c++
#include <iostream>

class Point {
private:
    double m_x, m_y, m_z;
public:
    Point(double x = 0.0, double y = 0.0, double z = 0.0) : m_x(x), m_y(y), m_z(z) {}

    friend std::ostream &operator<<(std::ostream &out, const Point &point);

    friend std::istream &operator>>(std::istream &in, Point &point);
};

std::ostream &operator<<(std::ostream &out, const Point &point) {
    // Поскольку operator<< является другом класса Point, то мы имеем прямой
    // доступ к членам Point
    out << "Point(" << point.m_x << ", " << point.m_y << ", " << point.m_z << ")";
    return out;
}

std::istream &operator>>(std::istream &in, Point &point) {
    // Поскольку operator>> является другом класса Point, то мы имеем прямой
    // доступ к членам Point.
    // Обратите внимание, параметр point (объект класса Point) должен быть
    //неконстантным, чтобы мы имели возможность изменить члены класса
    in >> point.m_x;
    in >> point.m_y;
    in >> point.m_z;
    return in;
}

int main() {
    std::cout << "Enter a point: \n";
    Point point;
    std::cin >> point; // 4 5.5 8.37
    std::cout << "You entered " << point << '\n'; // You entered: Point(4, 5.5, 8.37)

    return 0;
}
```

### Тест

```c++
#include <iostream>

class Fraction {
private:
    int m_numerator = 0;
    int m_denominator = 1;
public:
    Fraction(int numerator = 0, int denominator = 1) :
            m_numerator(numerator), m_denominator(denominator) {
        // Мы поместили метод reduce() в конструктор, чтобы убедиться, что все
        // дроби, которые у нас есть, будут уменьшены!
        // Любые дроби, которые перезаписаны, должны быть повторно уменьшены
        reduce();
    }

    static int nod(int a, int b) {
        return b == 0 ? a : nod(b, a % b);
    }

    void reduce() {
        int nod = Fraction::nod(m_numerator, m_denominator);
        m_numerator /= nod;
        m_denominator /= nod;
    }

    friend Fraction operator*(const Fraction &f1, const Fraction &f2);

    friend Fraction operator*(const Fraction &f1, int value);

    friend Fraction operator*(int value, const Fraction &f1);

    friend std::ostream &operator<<(std::ostream &out, const Fraction &f1);

    friend std::istream &operator>>(std::istream &in, Fraction &f1);

    void print() {
        std::cout << m_numerator << "/" << m_denominator << "\n";
    }
};

Fraction operator*(const Fraction &f1, const Fraction &f2) {
    return Fraction(f1.m_numerator * f2.m_numerator, f1.m_denominator *
                                                     f2.m_denominator);
}

Fraction operator*(const Fraction &f1, int value) {
    return Fraction(f1.m_numerator * value, f1.m_denominator);
}

Fraction operator*(int value, const Fraction &f1) {
    return Fraction(f1.m_numerator * value, f1.m_denominator);
}

std::ostream &operator<<(std::ostream &out, const Fraction &f1) {
    out << f1.m_numerator << "/" << f1.m_denominator;
    return out;
}

std::istream &operator>>(std::istream &in, Fraction &f1) {
    char c;
    // Перезаписываем значения объекта f1
    in >> f1.m_numerator;
    in >> c; // игнорируем разделитель '/'
    in >> f1.m_denominator;
    // Поскольку мы перезаписали существующий f1, то нам нужно повторно
    // выполнить уменьшение дроби
    f1.reduce();
    return in;
}

int main() {
    Fraction f1;
    std::cout << "Enter fraction 1: ";
    std::cin >> f1; // 3/9
    Fraction f2;
    std::cout << "Enter fraction 2: ";
    std::cin >> f2; // 5/25
    std::cout << f1 << " * " << f2 << " is " << f1 * f2 << '\n'; // 1/3 * 1/5 is 1/15
    return 0;
}
```

## [Урок №142. Перегрузка операторов через методы класса](#урок-142-перегрузка-операторов-через-методы-класса)

### Перегрузка операторов через методы классов

```c++
#include <iostream>

class Dollars {
private:
    int m_dollars;
public:
    Dollars(int dollars) { m_dollars = dollars; }

    // Выполняем Dollars + int
    Dollars operator+(int value) const;

    int &getDollars() { return m_dollars; }
};

// Примечание: Эта функция является методом класса!
// Вместо параметра dollars в перегрузке через дружественную функцию здесь
// неявный параметр, на который указывает указатель *this
Dollars Dollars::operator+(int value) const {
    return {m_dollars + value};
}

int main() {
    Dollars dollars1(7);
    Dollars dollars2 = dollars1 + 3;
    std::cout << "I have " << dollars2.getDollars() << " dollars.\n"; // I have 10 dollars.

    return 0;
}
```

### Какой способ перегрузки и когда следует использовать?

* Для операторов присваивания (`=`), индекса (`[]`), вызова функции (`()`) или\
  выбора члена (`->`) используйте перегрузку через методы класса.
* Для унарных операторов используйте перегрузку через методы класса.
* Для перегрузки бинарных операторов, которые изменяют левый операнд\
  (например, `operator+=()`) используйте перегрузку через методы класса, если\
  это возможно.
* Для перегрузки бинарных операторов, которые не изменяют левый операнд\
  (например, operator+()) используйте перегрузку через\
  обычные/дружественные функции.

### [Урок №143. Перегрузка унарных операторов +, - и логического НЕ](#урок-143-перегрузка-унарных-операторов----и-логического-не)
```c++
#include <iostream>

class Dollars {
private:
    int m_dollars;
public:
    Dollars(int dollars) : m_dollars{dollars} {}

    // Выполняем -Dollars через метод класса
    Dollars operator-() const;

    Dollars operator!() const;
    
    Dollars operator+() const;

    int &getDollars() { return m_dollars; }
};

// Эта функция является методом класса!
Dollars Dollars::operator-() const {
    return {-m_dollars};
}

Dollars Dollars::operator!() const {
    return {!m_dollars};
}

Dollars Dollars::operator+() const {
    return *this;
}

int main() {
    const Dollars dollars1(7);
    std::cout << "My debt is " << (-dollars1).getDollars() << " dollars.\n"; // My debt is -7 dollars.

    return 0;
}
```
**Примечание: Определение метода можно записать и внутри класса. Здесь мы\
определили его вне тела класса для лучшей наглядности.**

### Тест
Так как `Something`, который мы возвращаем, является текущим объектом:
```c++
Something Something::operator+ () const {
    return *this;
}
```

## [Урок №144. Перегрузка операторов сравнения](#урок-144-перегрузка-операторов-сравнения)
Перегрузим оператор равенства `==` и оператор неравенства `!=`\
для класса `Car`:
```c++
#include <iostream>
#include <string>
#include <utility>

class Car {
private:
    std::string m_company;
    std::string m_model;
public:
    Car(std::string company, std::string model)
            : m_company{std::move(company)}, m_model{std::move(model)} {
    }

    friend bool operator==(const Car &c1, const Car &c2);

    friend bool operator!=(const Car &c1, const Car &c2);
};

bool operator==(const Car &c1, const Car &c2) {
    return (c1.m_company == c2.m_company &&
            c1.m_model == c2.m_model);
}

bool operator!=(const Car &c1, const Car &c2) {
    return !(c1 == c2);
}

int main() {
    Car mustang("Ford", "Mustang");
    Car logan("Renault", "Logan");
    if (mustang == logan)
        std::cout << "Mustang and Logan are the same.\n";
    if (mustang != logan)
        std::cout << "Mustang and Logan are not the same.\n"; // Mustang and Logan are not the same.

    return 0;
}
```

**Совет: Не перегружайте операторы, которые являются бесполезными для\
вашего класса.**

### Перегрузим операторы сравнения `>`, `<`, `>=` и `<=`:
```c++
#include <iostream>

class Dollars {
private:
    int m_dollars;
public:
    Dollars(int dollars) { m_dollars = dollars; }

    friend bool operator>(const Dollars &d1, const Dollars &d2);

    friend bool operator<=(const Dollars &d1, const Dollars &d2);

    friend bool operator<(const Dollars &d1, const Dollars &d2);

    friend bool operator>=(const Dollars &d1, const Dollars &d2);
};

bool operator>(const Dollars &d1, const Dollars &d2) {
    return d1.m_dollars > d2.m_dollars;
}

bool operator>=(const Dollars &d1, const Dollars &d2) {
    return d1.m_dollars >= d2.m_dollars;
}

bool operator<(const Dollars &d1, const Dollars &d2) {
    return d1.m_dollars < d2.m_dollars;
}

bool operator<=(const Dollars &d1, const Dollars &d2) {
    return d1.m_dollars <= d2.m_dollars;
}

int main() {
    Dollars ten(10);
    Dollars seven(7);

    if (ten > seven)
        std::cout << "Ten dollars are greater than seven dollars.\n"; // сработает
    if (ten >= seven)
        std::cout << "Ten dollars are greater than or equal to seven dollars. \n"; // сработает
    if (ten < seven)
        std::cout << "Seven dollars are greater than ten dollars.\n";
    if (ten <= seven)
        std::cout << "Seven dollars are greater than or equal to ten dollars. \n";
    return 0;
}
```

### Тест
**Задание №1.**\
Сравнения в `Dollars`, используя логические отрицания:
```c++
bool operator>(const Dollars &d1, const Dollars &d2) {
    return d1.m_dollars > d2.m_dollars;
}

bool operator>=(const Dollars &d1, const Dollars &d2) {
    return d1.m_dollars >= d2.m_dollars;
}

// Логической противоположностью оператора < является >=, поэтому мы можем
// просто инвертировать результат выполнения >=
bool operator<(const Dollars &d1, const Dollars &d2) {
    return !(d1 >= d2);
}

// Логической противоположностью оператора <= является >, поэтому мы можем
// просто инвертировать результат выполнения >
bool operator<=(const Dollars &d1, const Dollars &d2) {
    return !(d1 > d2);
}
```

**Задание №2.**\
Перегрузка операторов `<<` и `<`
```c++
#include <iostream>
#include <string>
#include <utility>
#include <vector>
#include <algorithm>

class Car {
private:
    std::string m_company;
    std::string m_model;
public:
    Car(std::string company, std::string model)
            : m_company(std::move(company)), m_model(std::move(model)) {
    }

    friend bool operator==(const Car &c1, const Car &c2);

    friend bool operator!=(const Car &c1, const Car &c2);

    friend std::ostream &operator<<(std::ostream &out, const Car &c) {
        out << '(' << c.m_company << ", " << c.m_model << ')';
        return out;
    }

    friend bool operator<(const Car &c1, const Car &c2) {
        if (c1.m_company < c2.m_company)
            return true;
        if (c1.m_company > c2.m_company)
            return false;
        if (c1.m_model < c2.m_model)
            return true;
        if (c1.m_model > c2.m_model)
            return false;
        return false;
    }
};

bool operator==(const Car &c1, const Car &c2) {
    return (c1.m_company == c2.m_company &&
            c1.m_model == c2.m_model);
}

bool operator!=(const Car &c1, const Car &c2) {
    return !(c1 == c2);
}

int main() {
    std::vector<Car> v;
    v.emplace_back("Ford", "Mustang");
    v.emplace_back("Renault", "Logan");
    v.emplace_back("Ford", "Ranger");
    v.emplace_back("Renault", "Duster");
    std::sort(v.begin(), v.end()); // требуется перегрузка оператора < для класса Car
    for (auto &car: v)
        std::cout << car << '\n'; // требуется перегрузка оператора << для класса Car
    return 0;
}

<<< (Ford, Mustang)
<<< (Ford, Ranger)
<<< (Renault, Duster)
<<< (Renault, Logan)
```

## [Урок №145. Перегрузка операторов инкремента и декремента](#урок-145-перегрузка-операторов-инкремента-и-декремента)
**Поскольку операторы инкремента и декремента являются унарными и изменяют\
свои операнды, то перегрузку следует выполнять через методы класса.**

### Перегрузка операторов инкремента и декремента версии префикс
Перегрузка операторов инкремента и декремента версии префикс аналогична\
перегрузке любых других унарных операторов:
```c++
#include <iostream>

class Number {
private:
    int m_number;
public:
    Number(int number = 0) : m_number(number) {}

    Number &operator++();
    Number &operator--();
    friend std::ostream &operator<<(std::ostream &out, const Number &n);
};

Number &Number::operator++() {
    // Если значением переменной m_number является 8, то выполняем сброс
    // значения m_number на 0
    if (m_number == 8)
        m_number = 0;
    // В противном случае, просто увеличиваем m_number на единицу
    else
        ++m_number;
    return *this;
}

Number &Number::operator--() {
    // Если значением переменной m_number является 0, то присваиваем m_number
    // значение 8
    if (m_number == 0)
        m_number = 8;
    // В противном случае, просто уменьшаем m_number на единицу
    else
        --m_number;
    return *this;
}

std::ostream &operator<<(std::ostream &out, const Number &n) {
    out << n.m_number;
    return out;
}

int main() {
    Number number(7);
    std::cout << number; // 7
    std::cout << ++number; // 8
    std::cout << ++number; // 0
    std::cout << --number; // 8
    std::cout << --number; // 7

    return 0;
}
```
Здесь класс Number содержит число от `0` до `8`. Мы перегрузили операторы\
инкремента/декремента таким образом, чтобы они увеличивали/уменьшали\
`m_number` в соответствии с заданным диапазоном (если выполняется инкремент и\
`m_number` равно `8`, то сбрасываем значение `m_number` на `0`; если выполняется\
декремент и `m_number` равно `0`, то присваиваем значение `8` переменной\
`m_number`).

Обратите внимание, мы возвращаем скрытый указатель `*this` в функциях перегрузки\
операторов (т.е. текущий объект класса `Number`). Таким образом мы можем связать\
выполнение нескольких операторов в одну «цепочку».
```c++
std::cout << ++number.operator++().operator--().operator++().operator++(); // 1
```

### Перегрузка операторов инкремента и декремента версии постфикс
Язык C++ использует **фиктивную переменную** (или **«фиктивный параметр»**)\
для операторов версии постфикс. Этот фиктивный целочисленный параметр\
используется только с одной целью: отличить версию постфикс операторов\
инкремента/декремента от версии префикс. **Выполним перегрузку операторов\
инкремента/декремента версии префикс и постфикс в одном классе**:
```c++
#include <iostream>

class Number {
private:
    int m_number;
public:
    Number(int number = 0) : m_number(number) {}

    Number &operator++(); // версия префикс
    Number &operator--(); // версия префикс

    Number operator++(int); // версия постфикс
    Number operator--(int); // версия постфикс

    friend std::ostream &operator<<(std::ostream &out, const Number &n);
};

Number &Number::operator++() {
    // Если значением переменной m_number является 8, то выполняем сброс
    // значения m_number на 0
    if (m_number == 8)
        m_number = 0;
    // В противном случае, просто увеличиваем m_number на единицу
    else
        ++m_number;
    return *this;
}

Number &Number::operator--() {
    // Если значением переменной m_number является 0, то присваиваем m_number
    // значение 8
    if (m_number == 0)
        m_number = 8;
    // В противном случае, просто уменьшаем m_number на единицу
    else
        --m_number;
    return *this;
}

Number Number::operator++(int) {
    // Создаем временный объект класса Number с текущим значением переменной m_number
    Number temp(m_number);
    // Используем оператор инкремента версии префикс для реализации перегрузки
    // оператора инкремента версии постфикс
    ++(*this); // реализация перегрузки
    return temp; // Возвращаем временный объект
}

Number Number::operator--(int) {
    // Создаем временный объект класса Number с текущим значением переменной m_number
    Number temp(m_number);
    // Используем оператор декремента версии префикс для реализации
    // перегрузки оператора декремента версии постфикс
    ++(*this); // реализация перегрузки
    return temp; // Возвращаем временный объект
}

std::ostream &operator<<(std::ostream &out, const Number &n) {
    out << n.m_number;
    return out;
}

int main() {
    Number number(6);
    std::cout << number; // 6
    std::cout << ++number; // вызывается Number::operator++(); 7
    std::cout << number++; // вызывается Number::operator++(int); 7
    std::cout << number; // 8
    std::cout << --number; // вызывается Number::operator--(); // 7
    std::cout << number--; // вызывается Number::operator--(int); // 7
    std::cout << number; // 8

    return 0;
}
```

**Здесь есть несколько интересных моментов:**
* Во-первых, мы отделили версию постфикс от версии префикс\
  использованием целочисленного фиктивного параметра в версии постфикс.
* Во-вторых, поскольку фиктивный параметр не используется в реализации\
  самой перегрузки, то мы даже не предоставляем ему имя. Таким образом,\
  компилятор будет рассматривать эту переменную, как простую заглушку\
  (заполнитель места), и даже не будет предупреждать нас о том, что мы\
  объявили переменную, но никогда её не использовали.
* В-третьих, операторы версий префикс и постфикс выполняют одно и то же\
  задание: оба увеличивают/уменьшают значение переменной объекта.\
  Разница между ними только в значении, которое они возвращают.

**Решением является использование временного объекта с текущим значением\
переменной-члена**. Тогда можно будет увеличить/уменьшить исходный объект, а\
временный объект возвратить обратно в caller. Таким образом, caller получит копию\
объекта до того, как фактический объект будет увеличен или уменьшен, и сама\
операция инкремента/декремента выполнится успешно.

## [Урок №146. Перегрузка оператора индексации `[]`](#урок-146-перегрузка-оператора-индексации-)
```c++
#include <iostream>

class IntArray {
private:
    int m_array[10];
public:
    int &operator[](int index);
};

int &IntArray::operator[](const int index) {
    return m_array[index];
}

int main() {
    IntArray array{};
    array[4] = 5; // присваиваем значение
    std::cout << array[4]; // выводим значение - 5

    return 0;
}
```
Всё просто. При обработке `array[4]` компилятор сначала проверяет, есть ли\
функция перегрузки оператора `[]`. Если есть, то он передает в функцию перегрузки\
значение внутри квадратных скобок (в данном случае `4`) в качестве аргумента.


### Почему оператор индексации `[]` использует возврат по ссылке?
Поскольку результат выполнения оператора `[]` может использоваться в\
левой части операции присваивания (например, `array[4] = 5`), то возвращаемое\
значение оператора `[]` должно быть **l-value**. Ссылки же всегда являются l-values,\
так как их можно использовать только с переменными, которые имеют адреса памяти.\
Поэтому, используя возврат по ссылке, компилятор останется доволен, что\
возвращается l-value, и никаких проблем не будет.

### Использование оператора индексации с константными объектами класса
```c++
#include <iostream>

class IntArray {
private:
    // указываем начальные значения
    int m_array[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
public:
    int &operator[](int index);

    const int &operator[](int index) const;
};

// для неконстантных объектов: может использоваться как для присваивания значений элементам,
// так и для их просмотра
int &IntArray::operator[](const int index) {
    return m_array[index];
}

// для константных объектов: используется только для просмотра (вывода) элементов массива
const int &IntArray::operator[](const int index) const {
    return m_array[index];
}

int main() {
    IntArray array{};
    array[4] = 5; // хорошо: вызывается неконстантная версия operator[]()
    std::cout << array[4]; // 5

    const IntArray carray;
    carray[4] = 5; // ошибка компиляции: вызывается константная версия operator[](),
    // которая возвращает константную ссылку. Выполнять операцию присваивания нельзя
    std::cout << carray[4]; // 4

    return 0;
}
```

### Проверка ошибок
```c++
#include <iostream>
#include <cassert> // для assert()

class IntArray{
private:
    int m_array[10];
public:
    int& operator[] (int index);
};

int& IntArray::operator[] (const int index){
    assert(index >= 0 && index < 10);
    return m_array[index];
}

int main() {
    IntArray array{};
    array[2] = 5;
    std::cout << array[2]; // 5
    std::cout << array[10]; // asssertion error

    return 0;
}
```

### Указатели на объекты и перегруженный оператор `[]`
```c++
#include <iostream>
#include <cassert> // для assert()

class IntArray{
private:
    int m_array[10];
public:
    int& operator[] (int index);
};

int& IntArray::operator[] (const int index){
    assert(index >= 0 && index < 10);
    return m_array[index];
}

int main() {
    auto *array = new IntArray;
    (*array)[4] = 5; // сначала разыменовываем указатель для получения объекта array,
    // а затем вызываем operator[]
    std::cout << &array[4]; // 0x55fa7f002350
    delete array;

    return 0;
}
```

**Не используйте указатели на объекты, если это не является обязательным.**

### Передаваемый аргумент не обязательно должен быть целым числом
```c++
#include <iostream>
#include <string>

class Something {
private:
public:
    void operator[](std::string index);
};

// Нет смысла перегружать оператор [] только для вывода чего-либо,
// но это самый простой способ показать, что параметр функции может быть не
// только целочисленным значением
void Something::operator[](std::string index) {
    std::cout << index;
}

int main() {
    Something something;
    something["Hello, world!"]; // Hello, world!

    return 0;
}
```

### Тест
**Задание №1.**\
Контейер `map` (`dict`).
```c++
#include <iostream>
#include <string>
#include <vector>

struct StudentGrade {
    std::string name;
    char grade;
};

class GradeMap {
private:
    std::vector<StudentGrade> m_map;
public:
    GradeMap() = default;

    char &operator[](const std::string &name);
};

char &GradeMap::operator[](const std::string &name) {
    // Смотрим, найдем ли мы имя ученика в векторе
    for (auto &ref: m_map) {
    // Если нашли, то возвращаем ссылку на его оценку
        if (ref.name == name)
            return ref.grade;
    }
    // В противном случае, создаем новый StudentGrade для нового ученика
    StudentGrade temp{name, ' '};
    // Помещаем его в конец вектора
    m_map.push_back(temp);
    // И возвращаем ссылку на его оценку
    return m_map.back().grade;
}

int main() {
    GradeMap grades;
    grades["John"] = 'A';
    grades["Martin"] = 'B';
    std::cout << "John has a grade of " << grades["John"] << '\n'; // John has a grade of A
    std::cout << "Martin has a grade of " << grades["Martin"] << '\n'; // Martin has a grade of B
    return 0;
}
```

**Задание №2.**\
`std::vector` не является изначально отсортированным. Это означает, что каждый\
раз, при вызове `operator[]()`, мы будем перебирать весь `std::vector` для поиска\
элемента. С несколькими элементами это не является проблемой, но, по мере\
того как их количество будет увеличиваться, процесс поиска элемента будет\
становиться все медленнее и медленнее. Мы могли бы это оптимизировать,\
сделав `m_map` отсортированным и используя бинарный поиск. Таким образом,\
количество элементов, которые будут использоваться при просмотре во время\
поиска одного элемента, уменьшится в разы.

**Задание №3.**\
```c++
int main() {
    GradeMap grades;
    char &gradeJohn = grades["John"]; // выполняется push_back
    gradeJohn = 'A';
    char &gradeMartin = grades["Martin"]; // выполняется push_back
    gradeMartin = 'B';
    std::cout << "John has a grade of " << gradeJohn << '\n'; // John has a grade of A
    std::cout << "Martin has a grade of " << gradeMartin << '\n'; // Martin has a grade of B

    return 0;
}
```

## [Урок №147. Перегрузка оператора `()`](#урок-147-перегрузка-оператора-)
**Оператор `()`** является особенно интересным, поскольку позволяет изменять как\
тип параметров, так и их количество.

**Важно помнить о двух вещах:**
* Во-первых, перегрузка круглых скобок должна осуществляться через метод\
  класса.
* Во-вторых, в не объектно-ориентированном С++ оператор `()` является\
  оператором вызова функции. В случае с классами перегрузка круглых скобок\
  выполняется в методе `operator()(){}` (в объявлении функции перегрузки\
  находятся две пары круглых скобок).

```c++
#include <iostream>
#include <cassert> // для assert()

class Matrix {
private:
    double data[5][5]{};
public:
    Matrix() {
        // Присваиваем всем элементам массива значение 0.0
        for (auto & row : data)
            for (double & col : row)
                col = 0.0;
    }

    double &operator()(int row, int col);

    const double &operator()(int row, int col) const; // для константных объектов
};

double &Matrix::operator()(int row, int col) {
    assert(col >= 0 && col < 5);
    assert(row >= 0 && row < 5);
    return data[row][col];
}

const double &Matrix::operator()(int row, int col) const {
    assert(col >= 0 && col < 5);
    assert(row >= 0 && row < 5);
    return data[row][col];
}

int main() {
    Matrix matrix;
    matrix(2, 3) = 3.6;
    std::cout << matrix(2, 3); // 3.6

    return 0;
}
```

Выполним перегрузку оператора `()` еще раз, но уже без использования\
каких-либо параметров:
```c++
#include <iostream>
#include <cassert> // для assert()

class Matrix {
private:
    double data[5][5]{};
public:
    Matrix() {
        // Присваиваем всем элементам массива значение 0.0
        for (auto & row : data)
            for (double & col : row)
                col = 0.0;
    }

    double &operator()(int row, int col);

    const double &operator()(int row, int col) const; // для константных объектов

    void operator()();
};

double &Matrix::operator()(int row, int col) {
    assert(col >= 0 && col < 5);
    assert(row >= 0 && row < 5);
    return data[row][col];
}

const double &Matrix::operator()(int row, int col) const {
    assert(col >= 0 && col < 5);
    assert(row >= 0 && row < 5);
    return data[row][col];
}

void Matrix::operator()() {
    // Сбрасываем значения всех элементов массива на 0.0
    for (auto & row : data)
        for (double & col : row)
            col = 0.0;
}

int main() {
    Matrix matrix;
    matrix(2, 3) = 3.6;
    matrix();
    std::cout << matrix(2, 3); // 0

    return 0;
}
```

Поскольку оператор `()` является очень гибким, то может возникнуть соблазн\
использовать его для самых разных целей. Однако это настоятельно не\
рекомендуется делать, поскольку оператор `()` не является очень информативным и\
зачастую может быть не понятно, что он делает. В примере, приведенном выше,\
функцию очистки массива лучше было бы записать в виде метода `clear()` или `erase()`,\
поскольку `matrix.erase()` выглядит намного информативнее, нежели `matrix()`.

### Функторы в C++
Перегрузка оператора `()` используется в реализации **функторов** (или\
**"функциональных объектов"**) — классы, которые работают как функции.\
Преимущество функтора над обычной функцией заключается в том, что функторы\
могут хранить данные в переменных-членах (поскольку они сами являются\
классами):
```c++
#include <iostream>

class Accumulator {
private:
    int m_counter = 0;
public:
    Accumulator() = default;

    int operator()(int i) { return (m_counter += i); }
};

int main() {
    Accumulator accum;
    std::cout << accum(30) << std::endl; // выведется 30
    std::cout << accum(40) << std::endl; // выведется 70

    return 0;
}
```
Обратите внимание, использование класса `Accumulator` выглядит так же, как и\
вызов обычной функции, но наш объект класса `Accumulator` может хранить\
значение, которое увеличивается.

**С помощью функторов мы можем создать любое количество отдельных функциональных\
объектов, которые нам нужны, и использовать их одновременно.**

### Резюме
Перегрузка оператора `()` с двумя параметрами используется для получения доступа\
к двумерным массивам или для возврата подмножеств одномерного массива (два\
параметра будут конкретизировать условия отбора элементов подмножества). Всё\
остальное лучше реализовать через отдельные методы с более информативными\
названиями, нежели через перегрузку оператора `()`. Перегрузка оператора `()` также\
часто используется при создании функторов.

### Тест
```c++
#include <iostream>
#include <string>
#include <utility>

class Mystring {
private:
    std::string m_string;
public:
    Mystring(std::string string = "")
            : m_string{std::move(string)} {
    }

    std::string operator()(int index1, int length) {
        std::string ret;
        for (int count = 0; count < length; ++count)
            ret += m_string[index1 + count];
        return ret;
    }
};

int main() {
    Mystring string("Hello, world!");
    // начинаем с 7-го символа (индекса) и возвращаем следующие 6 символов
    std::cout << string(7, 6); // world!
    return 0;
}
```

## [Урок №148. Перегрузка операций преобразования типов данных](#урок-148-перегрузка-операций-преобразования-типов-данных)

