# Глава №13. Шаблоны в C++
## Содержание
1. [Урок №181. Шаблоны функций](#урок-181-шаблоны-функций)
2. [Урок №182. Экземпляры шаблонов функций](#урок-182-экземпляры-шаблонов-функций)
3. [Урок №175. Шаблоны классов](#урок-175-шаблоны-классов)
4. [Урок №184. Параметр `non-type` в шаблоне](#урок-184-параметр-non-type-в-шаблоне)
5. [Урок №185. Явная специализация шаблона функции](#урок-185-явная-специализация-шаблона-функции)
6. [Урок №186. Явная специализация шаблона класса](#урок-186-явная-специализация-шаблона-класса)
7. [Урок №187. Частичная специализация шаблона](#урок-187-частичная-специализация-шаблона)
8. [Урок №188. Частичная специализация шаблонов и Указатели](#урок-188-частичная-специализация-шаблонов-и-указатели)
9. [Глава №13. Итоговый тест](#глава-13-итоговый-тест)

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
### Параметр `non-type`
**Параметр `non-type` в шаблоне** — это специальный параметр шаблона, который\
заменяется не типом данных, а конкретным значением. Этим значением может\
быть:
* целочисленное значение или перечисление; 
* указатель или ссылка на объект класса; 
* указатель или ссылка на функцию; 
* указатель или ссылка на метод класса; 
* `std::nullptr_t`.

```c++
#include <iostream>

template<class T, int size> // size является параметром non-type в шаблоне класса
class StaticArray {
private:
    // Параметр non-type в шаблоне класса отвечает за размер выделяемого массива
    T m_array[size];

public:
    T *getArray();

    T &operator[](int index) {
        return m_array[index];
    }
};

// Синтаксис определения шаблона метода и самого метода вне тела класса с параметром non-type
template<class T, int size>
T *StaticArray<T, size>::getArray() {
    return m_array;
}

int main() {
    // Объявляем целочисленный массив из 10 элементов
    StaticArray<int, 10> intArray{};
    // Заполняем массив значениями
    for (int count = 0; count < 10; ++count)
        intArray[count] = count;
    // Выводим элементы массива в обратном порядке
    for (int count = 9; count >= 0; --count)
        std::cout << intArray[count] << " "; // 9 8 7 6 5 4 3 2 1 0 
    std::cout << '\n';

    // Объявляем массив типа double из 5 элементов
    StaticArray<double, 5> doubleArray{};
    // Заполняем массив значениями
    for (int count = 0; count < 5; ++count)
        doubleArray[count] = 5.5 + 0.1 * count;
    // Выводим элементы массива
    for (int count = 0; count < 5; ++count)
        std::cout << doubleArray[count] << ' '; // 5.5 5.6 5.7 5.8 5.9 

    return 0;
}
```

Примечательно то, что нам не пришлось динамически выделять переменную-член\
`m_array`! Это связано с тем, что для любого созданного объекта класса `StaticArray`\
его размер является конкретно заданным значением (можно сказать константой),\
которое передает пользователь. Например, если мы создадим экземпляр\
`StaticArray<int, 10>`, то компилятор заменит переменную размера массива\
(`size`) на 10. Таким образом, мы получим `m_array` типа `int[10]`, который можно\
выделить статическим образом.

## Урок №185. Явная специализация шаблона функции
Явная специализация шаблонов функций нужна для того, чтобы при работе с разными/
типами данных, функция реализовывала разное поведение:
```c++
#include <iostream>

template<class T>
class Repository {
private:
    T m_value;
public:
    Repository(T value) : m_value{value} {}

    ~Repository() {}

    void print() {
        std::cout << m_value << '\n';
    }
};

// специализация шаблона для char*
template<>
Repository<char *>::Repository(char *value) {
    // Определяем длину value
    int length = 0;
    while (value[length] != '\0')
        ++length;
    ++length; // +1, учитывая нуль-терминатор

    // Выделяем память для хранения значения value
    m_value = new char[length];

    // Копируем фактическое значение value в m_value
    for (int count = 0; count < length; ++count)
        m_value[count] = value[count];
}

// для того, чтобы выделенная память в специальном конструкторе, освободилась
template<>
Repository<char *>::~Repository() {
    delete[] m_value;
}

int main() {
    // Динамически выделяем временную строку
    char *string = new char[40];

    // Просим пользователя ввести свое имя
    std::cout << "Enter your name: ";
    std::cin.getline(string, 40); // Kirill

    // Сохраняем то, что ввел пользователь
    Repository<char *> repository(string);

    // Удаляем временную строку
    delete[] string;

    // Выводим то, что ввел пользователь
    repository.print(); // Kirill
}
```

## [Урок №186. Явная специализация шаблона класса](#урок-186-явная-специализация-шаблона-класса)
```c++
#include <iostream>

// Поскольку это шаблон класса, то он будет работать с любым типом данных
template<class T>
class Repository8 {
private:
    T m_array[8];

public:
    void set(int index, const T &value) {
        m_array[index] = value;
    }

    const T &get(int index) {
        return m_array[index];
    }
};

int main() {
    // Объявляем целочисленный объект-массив
    Repository8<int> intRepository{};
    for (int count = 0; count < 8; ++count)
        intRepository.set(count, count);
    for (int count = 0; count < 8; ++count)
        std::cout << intRepository.get(count) << '\n';

    // Объявляем объект-массив типа bool
    Repository8<bool> boolRepository{};
    for (int count = 0; count < 8; ++count)
        boolRepository.set(count, count % 5);
    for (int count = 0; count < 8; ++count)
        std::cout << (boolRepository.get(count) ? "true" : "false") << '\n';

    return 0;
}
```

Поскольку все переменные должны иметь адрес, а ЦП не может дать адрес чему-либо\
меньшему, чем 1 байт, то размер всех переменных должен быть не менее 1 байта.\
Следовательно, каждая переменная типа `bool` занимает целый байт, хотя технически\
ей нужен только 1 бит для хранения значения `true` или `false`! Таким образом,\
переменная типа `bool` — это 1 бит полезной информации и 7 бит потраченного\
впустую места.

### Специализация шаблона класса
**Специализация шаблона класса** (или **«явная специализация шаблона класса»**)\
позволяет специализировать шаблон класса для работы с определенным типом\
данных (или сразу с несколькими типами данных, если есть несколько параметров\
шаблона).

```c++
template<>
class Repository8<bool> { // специализируем шаблон класса Repository8 для работы с типом bool
private:
    unsigned char m_data;
public:
    Repository8() : m_data{0} {}

    void set(int index, bool value) {
        // Выбираем оперируемый бит
        unsigned char mask = 1 << index; // используем сдвиг влево

        if (value) // если на входе у нас true, то бит нужно "включить"
            m_data |= mask; // используем побитовое ИЛИ, чтобы "включить" бит
        else // если на входе у нас false, то бит нужно "выключить"
            m_data &= ~mask; // используем побитовое И, чтобы "выключить" бит
    }

    bool get(int index) {
        // Выбираем бит
        unsigned char mask = 1 << index;
        // Используем побитовое И для получения значения бита,
        // а затем выполняется его неявное преобразование в тип bool
        return (m_data & mask) != 0;
    }
};
```

Во-первых, начинаем с `template<>`. Ключевое слово `template` сообщает\
компилятору, что это шаблон, а пустые угловые скобки означают, что нет никаких\
параметров. А параметров нет из-за того, что мы заменяем единственный параметр\
шаблона (`T`, который отвечает за тип данных) конкретным типом данных (`bool`).\
Затем мы пишем имя класса и добавляем к нему `<bool>`, сообщая компилятору,\
что будем работать с типом `bool`.
Таким образом, вместо массива из 8 переменных типа `bool` (8 байтов) мы используем 1\
переменную типа `unsigned char` (1 байт).

```c++
int main() {
    // Объявляем целочисленный объект-массив
    // (создается экземпляр Repository8<T>, где T = int)
    Repository8<int> intRepository{};
    for (int count = 0; count < 8; ++count)
        intRepository.set(count, count);
    for (int count = 0; count < 8; ++count)
        std::cout << intRepository.get(count) << '\t'; // 0	1	2	3	4	5	6	7

    // Объявляем объект-массив типа bool
    // (создается экземпляр специализации Repository8<bool>)
    Repository8<bool> boolRepository{};
    for (int count = 0; count < 8; ++count)
        boolRepository.set(count, count % 5);
    for (int count = 0; count < 8; ++count)
        std::cout << (boolRepository.get(count) ? "true" : "false") << '\t';
        // false	true	true	true	true	false	true	true

    return 0;
}
```

## [Урок №187. Частичная специализация шаблона](#урок-187-частичная-специализация-шаблона)
### Частичная специализация шаблона
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> Частичная специализация шаблона позволяет выполнить специализацию\
> шаблона класса (но не функции!), где некоторые (но не все) параметры шаблона\
> явно определены.

```c++
#include <iostream>
#include <cstring>

template<class T, int size> // size является non-type параметром шаблона
class StaticArray {
private:
    // Параметр size отвечает за длину массива
    T m_array[size];

public:
    T *getArray() { return m_array; }

    T &operator[](int index) {
        return m_array[index];
    }
};

template<typename T, int size>
void print(StaticArray<T, size> &array) {
    for (int count = 0; count < size; ++count)
        std::cout << array[count] << ' ';
}

// Шаблон функции print() с частично специализированным шаблоном класса
// StaticArray<char, size> в качестве параметра
template<int size>
void print(StaticArray<char, size> &array) {
    for (int count = 0; count < size; ++count)
        std::cout << array[count];
}

int main() {
    // Объявляем массив типа char длиной 14
    StaticArray<char, 14> char14{};
    strcpy_s(char14.getArray(), 14, "Hello, world!");
    // Выводим элементы массива
    print(char14); // Hello, world!

    // Теперь объявляем массив типа char длиной 12
    StaticArray<char, 12> char12{};
    strcpy_s(char12.getArray(), 12, "Hello, dad!");
    // Выводим элементы массива
    print(char12); // Hello, dad!

    return 0;
}
```

### Частичная специализация шаблонов методов
```c++
#include <iostream>

template<class T, int size> // size является non-type параметром шаблона
class StaticArray_Base { // родительский класс для имплементации основных методов
protected:
    // Параметр size отвечает за длину массива
    T m_array[size]{};

public:
    T *getArray() { return m_array; }

    T &operator[](int index) { return m_array[index]; }

    virtual void print() {
        for (int i = 0; i < size; i++)
            std::cout << m_array[i] << ' ';
        std::cout << "\n";
    }
};

template<class T, int size> // size является non-type параметром шаблона
class StaticArray : public StaticArray_Base<T, size> { // дочерний StaticArray, 
    // который не переопределяет методы родительского класса
public:
    StaticArray() = default;
};

template<int size> // size является non-type параметром шаблона
class StaticArray<double, size> : public StaticArray_Base<double, size> {
    // дочерний, в котором входным является массив типа double с переопределением print'а
public:
    void print() override {
        for (int i = 0; i < size; i++)
            std::cout << std::scientific << this->m_array[i] << " ";
        // Примечание: Префикс this-> на вышеприведенной строке необходим.
        // Почему? Читайте здесь - https://stackoverflow.com/a/6592617
        std::cout << "\n";
    }
};

int main() {
    // Объявляем целочисленный массив длиной 5
    StaticArray<int, 5> intArray;
    // Заполняем его, а затем выводим
    for (int count = 0; count < 5; ++count)
        intArray[count] = count;
    intArray.print(); // 0 1 2 3 4 

    // Объявляем массив типа double длиной 4
    StaticArray<double, 4> doubleArray;
    // Заполняем его, а затем выводим
    for (int count = 0; count < 4; ++count)
        doubleArray[count] = (4. + 0.1 * count);
    doubleArray.print(); // 4.000000e+00 4.100000e+00 4.200000e+00 4.300000e+00 

    return 0;
}
```
Таким образом, используя наследование, мы можем использовать `StaticArray` с разными\
типами переменных и конкретными методами под эти типы.

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> Префикс `this->` нужен потому, что родительский шаблон класса `template` не создается во\
> время прохождения компиляции, при котором сначала проверяется шаблон. Эти имена,\
> по-видимому, не зависят от конкретного экземпляра шаблона, и поэтому определения должны\
> быть доступны. Так, необходимо явно указать компилятору, что имена на самом деле зависят\
> от создания экземпляра родительского объекта.\
> [Оригинал на StackOverflow](https://stackoverflow.com/a/6592617)

## [Урок №188. Частичная специализация шаблонов и Указатели](#урок-188-частичная-специализация-шаблонов-и-указатели)
```c++
#include <iostream>

// Общий шаблон класса Repository
template<class T>
class Repository {
private:
    T m_value;
public:
    Repository(T value) : m_value{value} {}

    ~Repository() {}

    void print() { std::cout << m_value << '\n'; }
};

template<typename T>
class Repository<T *> { // частичная специализация шаблона класса Repository
    // для работы с типами указателей
private:
    T *m_value;
public:
    Repository(T *value) { // T - тип указателя
        // Выполняем глубокое копирование
        m_value = new T(*value); // здесь копируется только одно
        // отдельное значение (не массив значений)
    }

    ~Repository() {
        delete m_value; // а здесь выполняется удаление этого значения
    }

    void print() {
        std::cout << *m_value << '\n';
    }
};

int main() {
    // Объявляем целочисленный объект для проверки работы общего шаблона класса
    Repository<int> myint(6);
    myint.print(); // 6

    // Объявляем объект с типом указатель для проверки работы частичной специализации шаблона класса
    int x = 8;
    Repository<int *> myintptr(&x);

    // Если бы в myintptr выполнилось поверхностное копирование (присваивание указателя),
    // то изменение значения x изменило бы и значение myintptr
    x = 10;
    myintptr.print(); // 8

    return 0;
}
```

При объявлении объекта `myintptr` с типом `int*`, компилятор видит, что мы ранее\
определили частичную специализацию шаблона класса для работы с типами\
указателей, и, учитывая, что мы использовали тип `int*`, компилятор создаст\
экземпляр частичной специализации шаблона для работы с типом указателя.\
Конструктор этой специализации выполняет глубокое копирование параметра `x`.\
Позже, когда мы изменяем значение `x` на `10`, `myintptr.m_value` никак не\
задевается, так как выполнилось глубокое копирование, при котором `m_value`\
получил свою собственную копию `x`.

Если бы этой частичной специализации не существовало, то создался бы экземпляр\
общего шаблона класса, в котором выполнилось бы поверхностное копирование, а\
`myintptr.m_value`, и `x` указывали бы на один и тот же адрес памяти. В таком\
случае, при изменении значения переменной `x` на `10`, мы также затронули бы и\
значение `myintptr` (оно также стало бы равно `10`).

Стоит отметить, что, поскольку в нашей частичной специализации копируется\
только одно значение, при работе со строками `C-style` копироваться будет только\
первый символ (так как строка — это массив, а указатель на массив указывает\
только на первый элемент массива). Если же нужно полностью скопировать строку,\
то специализация конструктора (и деструктора) для типа `char*` должна быть полной.\
В таком случае, полная специализация будет иметь приоритет выше, чем частичная\
специализация.

Использование как частичной, так и полной специализации для работы с типом\
`char*`:
```c++
#include <iostream>

// Общий шаблон класса Repository
template<class T>
class Repository {
private:
    T m_value;
public:
    Repository(T value) : m_value{value} {}

    ~Repository() = default;

    void print() { std::cout << m_value << '\n'; }
};

template<typename T>
class Repository<T *> { // частичная специализация шаблона класса Repository
    // для работы с типами указателей
private:
    T *m_value;
public:
    Repository(T *value) { // T - тип указателя
        // Выполняем глубокое копирование
        m_value = new T(*value); // здесь копируется только одно
        // отдельное значение (не массив значений)
    }

    ~Repository() {
        delete m_value; // а здесь выполняется удаление этого значения
    }

    void print() {
        std::cout << *m_value << '\n';
    }
};

// Полная специализация шаблона конструктора класса Repository для работы с типом char*
template<>
Repository<char *>::Repository(char *value) { // глубокое копирование
    // Определяем длину value
    int length = 0;
    while (value[length] != '\0')
        ++length;
    ++length; // +1, учитывая нуль-терминатор

    // Выделяем память для хранения значения value
    m_value = new char[length];

    // Копируем фактическое значение value в m_value
    for (int count = 0; count < length; ++count)
        m_value[count] = value[count];
}

// Полная специализация шаблона деструктора класса Repository для работы с типом char*
template<>
Repository<char *>::~Repository() {
    delete[] m_value;
}

// Полная специализация шаблона метода print() для работы с типом char*.
// Без этого вывод Repository<char*> привел бы к вызову Repository<T*>::print(),
// который выводит только одно значение (в случае со строкой C-style - только первый символ)
template<>
void Repository<char *>::print() {
    std::cout << m_value;
}

int main() {
    // Объявляем целочисленный объект для проверки работы общего шаблона класса
    Repository<int> myint(6);
    myint.print(); // 6

    // Объявляем объект с типом указатель для проверки работы частичной специализации шаблона класса
    int x = 8;
    Repository<int *> myintptr(&x);

    // Если бы в myintptr выполнилось поверхностное копирование (присваивание указателя),
    // то изменение значения x изменило бы и значение myintptr
    x = 10;
    myintptr.print(); // 8

    // Динамически выделяем временную строку
    char *name = new char[40]{ "Anton" };
    // Сохраняем имя
    Repository<char*> myname(name);
    // Удаляем временную строку
    delete[] name;
    // Выводим имя
    myname.print(); // Anton

    return 0;
}
```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/success.svg">
>   <img alt="Success" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/success.svg">
> </picture><br>
>
> В итоге, использование частичной специализации шаблона класса для работы с\
> типами указателей особенно полезно, так как позволяет предусмотреть все\
> возможные варианты использования кода на практике.

## [Глава №13. Итоговый тест](#глава-13-итоговый-тест)
### Теория
**Шаблоны** позволяют написать одну версию функции или класса, которая будет\
работать с разными типами данных. Функция или класс, реализованная через\
шаблон, с фактическим (одним) типом данных называется **экземпляром**.

Все шаблоны функций или классов должны начинаться с **ключевого слова `template`\
и объявления параметров шаблона**. В объявлении параметров шаблона\
указываются параметры типа и параметры `non-type`.

**Параметр типа шаблона** — это параметр, который отвечает за типы данных, с\
которыми будет работать шаблон, обычно его называют `T`, `T1`, `T2` или другими\
(одиночными) буквами (например, `S`).

**Параметром `non–type`** может быть переменная интегрального типа данных\
(например, `char`, `bool`, `int`, `long`, `short`), указатель/ссылка на функцию или на\
метод/объект класса, `std::nullptr_t`.

Разделение определения шаблонов класса и его методов по разным файлам не\
работает как с обычными классами — вы не можете поместить определение\
шаблона класса в заголовочный файл, а определение шаблонов методов этого\
класса в отдельный файл _.cpp_. Как правило, лучше всё хранить в заголовочном\
файле с определениями шаблонов методов под определением шаблона класса.

**Явная специализация шаблона** используется для определения реализации,\
отличающейся от общей, функции или класса при работе с определенным типом\
данных. Если все параметры специализации шаблона явно определены, то это\
**полная специализация**. Классы также поддерживают **частичную специализацию**,\
при которой не все параметры шаблона должны быть явно определены. В C++14\
частичная специализация шаблонов функций запрещена.

Многие классы в Стандартной библиотеке C++ (такие как `std::array` и `std::vector`)\
используют шаблоны. Шаблоны часто применяются для реализации контейнерных\
классов, которые можно один раз написать и использовать с любыми типами данных.

### Тест
**Задание №1.**
```c++
#include <iostream>

template<class T>
class Pair1 {
private:
    T m_a;
    T m_b;
public:
    Pair1(const T &a, const T &b) : m_a{a}, m_b{b} {}

    T &first() { return m_a; }

    const T &first() const { return m_a; }

    T &second() { return m_b; }

    const T &second() const { return m_b; }
};

int main() {
    Pair1<int> p1(6, 9);
    std::cout << "Pair: " << p1.first() << ' ' << p1.second() << '\n'; // Pair: 6 9

    const Pair1<double> p2(3.4, 7.8);
    std::cout << "Pair: " << p2.first() << ' ' << p2.second() << '\n'; // Pair: 3.4 7.8

    return 0;
}
```

**Задание №2.**\
Разные типы значений:
```c++
#include <iostream>

template<class T, class S>
class Pair {
private:
    T m_a;
    S m_b;
public:
    Pair(const T &a, const S &b) : m_a{a}, m_b{b} {}

    T &first() { return m_a; }

    const T &first() const { return m_a; }

    S &second() { return m_b; }

    const S &second() const { return m_b; }
};

int main() {
    Pair<int, double> p1(6, 7.8);
    std::cout << "Pair: " << p1.first() << ' ' << p1.second() << '\n'; // Pair: 6 7.8

    const Pair<double, int> p2(3.4, 5);
    std::cout << "Pair: " << p2.first() << ' ' << p2.second() << '\n'; // Pair: 3.4 5

    return 0;
}
```

**Задание №3.**\
Первое значение всегда является типа `string`, а второе может быть любого типа:
```c++
#include <iostream>

template<class T, class S>
class Pair {
private:
    T m_a;
    S m_b;
public:
    Pair(const T &a, const S &b) : m_a{a}, m_b{b} {}

    T &first() { return m_a; }

    const T &first() const { return m_a; }

    S &second() { return m_b; }

    const S &second() const { return m_b; }
};

template<class S>
class StringValuePair : public Pair<std::string, S> {
public:
    StringValuePair(const std::string &key, const S &value)
            : Pair<std::string, S>(key, value) {}
};

int main() {
    StringValuePair<int> svp("Amazing", 7);
    std::cout << "Pair: " << svp.first() << ' ' << svp.second() << '\n'; // Pair: Amazing 7

    return 0;
}
```
