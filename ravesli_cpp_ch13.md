# Глава №13. Шаблоны в C++
## Содержание
1. [Урок №181. Шаблоны функций](#урок-181-шаблоны-функций)
2. [Урок №182. Экземпляры шаблонов функций](#урок-182-экземпляры-шаблонов-функций)
3. [Урок №175. Шаблоны классов](#урок-175-шаблоны-классов)
4. [Урок №184. Параметр `non-type` в шаблоне](#урок-184-параметр-non-type-в-шаблоне)

## [Урок №181. Шаблоны функций](#урок-181-шаблоны-функций)
В языке C++ **шаблоны функций — это функции, которые служат образцом для\
создания других подобных функций**. Главная идея — создание функций без\
указания точного типа(ов) некоторых или всех переменных. Для этого мы\
определяем функцию, указывая **тип параметра шаблона**, который используется\
вместо любого типа данных.

### Создание шаблонов функций
```c++
template <typename T> // объявление параметра шаблона функции
T max(T a, T b) {
    return (a > b) ? a : b;
}
```
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> Примечание: Поскольку тип аргумента функции, передаваемый в тип T, может\
> быть классом, а классы, как правило, не рекомендуется передавать по значению,\
> то лучше сделать параметры и возвращаемое значение нашего шаблона функции\
> константными ссылками, например:
> ```c++ 
> template <typename T>
> const T& max(const T& a, const T& b) {
>   return (a > b) ? a : b;
> }
> ```

### Использование шаблонов функций
```c++
#include <iostream>

// объявление параметра шаблона функции
template<typename T>

const T &max(const T &a, const T &b) {
    return (a > b) ? a : b;
}

int main() {
    int i = max(4, 8);
    std::cout << i << '\n'; //8

    double d = max(7.56, 21.434);
    std::cout << d << '\n'; // 21.434

    char ch = max('b', '9');
    std::cout << ch << '\n'; // b

    return 0;
}
```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> <b>Примечание:</b> Стандартная библиотека C++ имеет в своем арсенале шаблон\
> функции `max()` (который находится в заголовочном файле `algorithm`), поэтому вы\
> можете не реализовывать эту функцию вручную в будущем. Кроме этого, если вы\
> пишете свои собственные шаблоны функций и используете стейтмент `using`\
> `namespace std;`, то не забывайте о возможности возникновения конфликтов\
> имен, так как компилятор не сможет определить, хотите ли вы использовать свою\
> версию функции `max()` или версию `std::max()`.

## [Урок №182. Экземпляры шаблонов функций](#урок-182-экземпляры-шаблонов-функций)
Язык C++ не компилирует шаблоны функций напрямую. Вместо этого, когда\
компилятор встречает вызов шаблона функции, он копирует шаблон функции и\
заменяет типы параметров шаблона функции фактическими (передаваемыми)\
типами данных. Функция с фактическими типами данных называется **экземпляром\
шаблона функции** (или **«объектом шаблона функции»**).

Так, к примеру, компилятор видит, что оба числа являются целочисленными, поэтому\
он копирует шаблон функции и создает экземпляр шаблона `max(int, int)`:
```c++
// Вместо этого
template<typename T>
const T &max(const T &a, const T &b) {
    return (a > b) ? a : b;
}

// на самом деле вызовется это
const int& max(const int &a, const int &b) {
    return (a > b) ? a : b;
}

int main() {
    int i = max(4, 8);
    return 0;
}
```

Или же если нам нужно снова вызвать функцию `max()`,\
но уже с другим типом данных:
```c++
// Вместо этого
template<typename T>
const T &max(const T &a, const T &b) {
    return (a > b) ? a : b;
}

// на самом деле вызовется это
const double& max(const double &a, const double &b) {
    return (a > b) ? a : b;
}

int main() {
    double d = max(7.58, 19.378);
}
```

Также стоит отметить, что, если вы создадите шаблон функции, но\
не вызовете его, экземпляры этого шаблона созданы не будут.

### Операторы, вызовы функций и шаблоны функций
```c++
#include <iostream>

// объявление параметра шаблона функции
template<typename T>
const T &max(const T &a, const T &b) {
    return (a > b) ? a : b;
}

class Dollars {
private:
    int m_dollars;
public:
    Dollars(int dollars) : m_dollars(dollars) {}

    // Для сравнения обязательным является перегрузка оператора > 
    friend bool operator>(const Dollars &d1, const Dollars &d2) {
        return (d1.m_dollars > d2.m_dollars);
    }
    
    // Просто для вывода
    friend std::ostream &operator<<(std::ostream &out, const Dollars &d) {
        out << d.m_dollars;
        return out;
    }
};


int main() {
    Dollars seven(7);
    Dollars twelve(12);

    Dollars bigger = max(seven, twelve);
    std::cout << bigger;

    return 0;
}
```

### Еще один пример
```c++
#include <iostream>

class Dollars {
private:
    int m_dollars;
public:
    Dollars(int dollars) : m_dollars(dollars) {}

    friend bool operator>(const Dollars &d1, const Dollars &d2) {
        return (d1.m_dollars > d2.m_dollars);
    }

    friend std::ostream &operator<<(std::ostream &out, const Dollars &dollars) {
        out << dollars.m_dollars << " dollars ";
        return out;
    }

    Dollars &operator+=(Dollars dollars) {
        m_dollars += dollars.m_dollars;
        return *this;
    }

    Dollars &operator/=(int value) {
        m_dollars /= value;
        return *this;
    }
};

template<class T>
T average(T *array, int length) {
    T sum = 0;
    for (int count = 0; count < length; ++count)
        sum += array[count];

    sum /= length;
    return sum;
}

int main() {
    Dollars array3[] = {
            Dollars(7), Dollars(12), Dollars(18), Dollars(15)};
    std::cout << average(array3, 4) << '\n';

    return 0;
}
```

## [Урок №175. Шаблоны классов](#урок-175-шаблоны-классов)
### Шаблоны и контейнерные классы
**Создание шаблона класса аналогично созданию шаблона функции.**

**Array.h:**
```c++
#ifndef ARRAY_H
#define ARRAY_H

#include <cassert> // для assert()

template<class T> // это шаблон класса с T вместо фактического (передаваемого) типа данных
class Array {
private:
    int m_length;
    T *m_data;
public:
    Array() {
        m_length = 0;
        m_data = nullptr;
    }

    Array(int length) {
        m_data = new T[length];
        m_length = length;
    }

    ~Array() {
        delete[] m_data;
    }

    void Erase() {
        delete[] m_data;
        // Присваиваем значение nullptr для m_data, чтобы на выходе не получить висячий указатель!
        m_data = nullptr;
        m_length = 0;
    }
    
    T &operator[](int index) {
        assert(index >= 0 && index < m_length);
        return m_data[index];
    }

    // Длина массива всегда является целочисленным значением, она не зависит от типа элементов массива
    int getLength(); // определяем метод и шаблон метода getLength() ниже
};

template<typename T>
// метод, определенный вне тела класса, нуждается в собственном определении шаблона метода
int Array<T>::getLength() { return m_length; } // обратите внимание, имя класса - Array<T>, а не просто Array

#endif

```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> Каждый метод шаблона класса, объявленный вне тела класса, нуждается в собственном\
> объявлении шаблона. Также обратите внимание, что имя шаблона класса — `Array<T>`,\
> а не `Array` (`Array` будет указывать на не шаблонную версию класса `Array`).

```c++
#include <iostream>
#include "Array.h"

int main() {
    Array<int> intArray(10);
    Array<double> doubleArray(10);

    for (int count = 0; count < intArray.getLength(); ++count) {
        intArray[count] = count;
        doubleArray[count] = count + 0.5;
    }

    for (int count = intArray.getLength()-1; count >= 0; --count)
        std::cout << intArray[count] << "\t" << doubleArray[count] << '\n';

    return 0;
}
```

Шаблоны классов работают точно так же, как и шаблоны функций: компилятор\
копирует шаблон класса, заменяя типы параметров шаблона класса на фактические\
(передаваемые) типы данных, а затем компилирует эту копию. Если у вас есть\
шаблон класса, но вы его не используете, то компилятор не будет его даже\
компилировать.

### Шаблоны классов и Заголовочные файлы
**Подход трех файлов:**
* Определение шаблона класса хранится в заголовочном файле.
* Определения методов шаблона класса хранятся в отдельном файле `.cpp`.
* Затем добавляем третий файл, который содержит все необходимые нам\
  экземпляры шаблона класса.

**Тогда, Array.h:**
```c++
#ifndef ARRAY_H
#define ARRAY_H

#include <cassert> // для assert()

template<class T>
class Array {
private:
    int m_length;
    T *m_data;

public:
    Array() {
        m_length = 0;
        m_data = nullptr;
    }

    Array(int length) {
        m_data = new T[length];
        m_length = length;
    }

    ~Array() {
        delete[] m_data;
    }

    void Erase() {
        delete[] m_data;
        // Присваиваем значение nullptr для m_data, чтобы на выходе не получить висячий указатель!
        m_data = nullptr;
        m_length = 0;
    }

    T &operator[](int index) {
        assert(index >= 0 && index < m_length);
        return m_data[index];
    }

    // Длина массива всегда является целочисленным значением, она не зависит от типа элементов массива
    int getLength();
};

#endif
```

**Array.cpp:**
```c++
#include "Array.h"

template<typename T>
int Array<T>::getLength() { return m_length; }
```

**templates.cpp:**
```c++
// Таким образом, мы гарантируем, что компилятор увидит полное определение шаблона класса Array
#include "Array.h"
#include "Array.cpp" // мы нарушаем правила хорошего тона в программировании, но только в этом месте

// Здесь вы #include другие файлы .h и .cpp с определениями шаблонов, которые вам нужны

template
class Array<int>; // явно создаем экземпляр шаблона класса Array<int>
template
class Array<double>; // явно создаем экземпляр шаблона класса Array<double>

// Здесь вы явно создаете другие экземпляры шаблонов, которые вам нужны
```

Часть `template class` заставит компилятор явно создать указанные экземпляры\
шаблона класса. В примере, приведенном выше, компилятор создаст `Array<int>`\
и `Array<double>` внутри templates.cpp. Поскольку templates.cpp находится внутри\
нашего проекта, то он скомпилируется и удачно свяжется с другими файлами\
(пройдет линкинг): 
```c++
#include <iostream>
#include "Array.h"

int main() {
    // РАБОТАЕТ
    Array<int> intArray(10);
    Array<double> doubleArray(10);

    for (int count = 0; count < intArray.getLength(); ++count) {
        intArray[count] = count;
        doubleArray[count] = count + 0.5;
    }

    for (int count = intArray.getLength()-1; count >= 0; --count)
        std::cout << intArray[count] << "\t" << doubleArray[count] << '\n';

    return 0;
}
```

## [Урок №184. Параметр `non-type` в шаблоне](#урок-184-параметр-non-type-в-шаблоне)