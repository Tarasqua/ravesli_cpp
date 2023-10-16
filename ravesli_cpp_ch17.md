# Глава №17. `std::string` в C++
## Содержание
1. [Урок №208. Строковые классы `std::string` и `std::wstring`](#урок-208-строковые-классы-stdstring-и-stdwstring)
2. [Урок №209. Создание, уничтожение и конвертация `std::string`](#урок-209-создание-уничтожение-и-конвертация-stdstring)
3. [Урок №210. Длина и ёмкость `std::string`](#урок-210-длина-и-ёмкость-stdstring)
4. [Урок №211. Доступ к символам `std::string`. Конвертация `std::string` в строки C-style](#урок-211-доступ-к-символам-stdstring-конвертация-stdstring-в-строки-c-style)
5. [Урок №212. Присваивание и перестановка значений с `std::string`](#урок-212-присваивание-и-перестановка-значений-с-stdstring)

## Урок №208. Строковые классы `std::string` и `std::wstring`
### Класс `std::string`
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/info.svg">
>   <img alt="Info" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/info.svg">
> </picture><br>
>
> `std::string` используется для стандартных ASCII-строк (кодировка UTF-8),\
> а `std::wstring` используется для Unicode-строк (кодировка UTF-16).

### Функционал `std::string`
**Создание и удаление:**
* **конструктор** — создает или копирует строку;
* **деструктор** — уничтожает строку.

**Размер и ёмкость:**
* `capacity()` — возвращает количество символов, которое строка может хранить\
  без дополнительного перевыделения памяти;
* `empty()` — возвращает логическое значение, указывающее, является ли\
  строка пустой;
* `length()`, `size()` — возвращают количество символов в строке;
* `max_size()` — возвращает максимальный размер строки, который может быть\
  выделен;
* `reserve()` — расширяет или уменьшает ёмкость строки.

**Доступ к элементам:**
* `[]`, `at()` — доступ к элементу по заданному индексу.

**Изменение:**
* `=`, `assign()` — присваивают новое значение строке;
* `+=`, `append()`, `push_back()` — добавляют символы к концу строки;
* `insert()` — вставляет символы в произвольный индекс строки;
* `clear()` — удаляет все символы строки;
* `erase()` — удаляет символы по произвольному индексу строки;
* `replace()` — заменяет символы произвольных индексов строки другими\
  символами;
* `resize()` — расширяет или уменьшает строку (удаляет или добавляет\
  символы в конце строки);
* `swap()` — меняет местами значения двух строк.

**Ввод/вывод:**
* `>>`, `getline()` — считывают значения из входного потока в строку;
* `<<` — записывает значение строки в выходной поток;
* `c_str()` — конвертирует строку в строку C-style с нуль-терминатором в конце;
* `copy()` — копирует содержимое строки (которое без нуль-терминатора) в\
  массив типа `char`;
* `data()` — возвращает содержимое строки в виде массива типа `char`, который\
  не заканчивается нуль-терминатором.

**Сравнение строк:**
* `==`, `!=` — сравнивают, являются ли две строки равными/неравными\
  (возвращают значение типа `bool`);
* `<`, `<=`, `>`, `>=` — сравнивают, являются ли две строки меньше или больше друг\
  друга (возвращают значение типа `bool`);
* `compare()` — сравнивает, являются ли две строки равными/неравными\
  (возвращает `-1`, `0` или `1`).

**Подстроки и конкатенация:**
* `+` — соединяет две строки;
* `substr()` — возвращает подстроку.

**Поиск:**
* `find` — ищет индекс первого символа/подстроки;
* `find_first_of` — ищет индекс первого символа из набора символов;
* `find_first_not_of` — ищет индекс первого символа НЕ из набора символов;
* `find_last_of` — ищет индекс последнего символа из набора символов;
* `find_last_not_of` — ищет индекс последнего символа НЕ из набора символов;
* `rfind` — ищет индекс последнего символа/подстроки.

**Поддержка итераторов и распределителей (allocators):**
* `begin()`, `end()` — возвращают "прямой" итератор, указывающий на первый и\
  последний (элемент, который идет за последним) элементы строки;
* `get_allocator()` — возвращает распределитель;
* `rbegin()`, `rend()` — возвращают "обратный" итератор, указывающий на\
  последний (т.е. "обратное" начало) и первый (элемент, который\
  предшествует первому элементу строки — "обратный" конец) элементы\
  строки. Отличие от `begin()` и `end()` в том, что движение итераторов происходит\
  в обратную сторону.

## [Урок №209. Создание, уничтожение и конвертация `std::string`](#урок-209-создание-уничтожение-и-конвертация-stdstring)
#### `string::string()`
* Конструктор по умолчанию, который создает пустую строку.

```c++
#include <iostream>
#include <string>
 
int main() {
    std::string sSomething;
    std::cout << sSomething; // 
 
    return 0;
}
```

#### `string::string(const string& strString)`
* Конструктор копирования, который создает новую строку путем\
  копирования `strString`.

```c++
#include <iostream>
#include <string>

int main() {
    std::string sSomething("What a string!");
    std::string sOutput(sSomething);
    std::cout << sOutput; // What a string!

    return 0;
}
```

#### `string::string(const string& strString, size_type unIndex)` <br> `string::string(const string& strString, size_type unIndex, size_type unLength)`
* Конструкторы, которые создают новые строки, которые состоят из строки\
  `strString` (начиная с индекса `unIndex`) и количества символов,\
  указанных в `unLength`.
* Если компилятор встречает `NULL`, то копирование строки завершается,\
  даже если `unLength` не был достигнут.
* Если `unLength` не был указан, то все символы, начиная с `unIndex`, будут\
  использованы.
* Если `unIndex` больше, чем размер строки, то выбрасывается\
  исключение `out_of_range`.

```c++
#include <iostream>
#include <string>

int main() {
    std::string sSomething("What a string!");
    std::string sOutput(sSomething, 3); // с 3 элемента до конца
    std::cout << sOutput << std::endl; // t a string!
    std::string sOutput2(sSomething, 5, 6); // с 5 длиной 6
    std::cout << sOutput2 << std::endl; // a stri
    
    return 0;
}
```

#### `string::string(const char *szCString)`
* Конструктор, который создает новую строку из передаваемой строки C-\
  style `szCString` вплоть до нуль-терминатора (но его не включает). Если\
  размер результата превышает максимальную длину строки, то генерируется\
  исключение `length_error`.

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/danger.svg">
>   <img alt="Danger" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/danger.svg">
> </picture><br>
>
> **Предупреждение:** `szCString` не должен быть `NULL`.

```c++
#include <iostream>
#include <string>

int main() {
    const char *szSomething("What a string!");
    std::string sOutput(szSomething);
    std::cout << sOutput << std::endl; // What a string!

    return 0;
}
```

#### `string::string(const char *szCString, size_type unLength)`
* Конструктор, который создает новую строку из строки C-style `szCString` с\
  количеством символов, указанных в `unLength`.
* Если размер результата превышает максимальную длину строки, то\
  генерируется исключение `length_error`.

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/danger.svg">
>   <img alt="Danger" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/danger.svg">
> </picture><br>
>
> **Предупреждение:** Только для этого конструктора значение `NULL` не\
> обрабатывается как объект, указывающий на завершение строки `szCString`! Это\
> означает, что компилятор дойдет до конца строки (если это позволяет `unLength`),\
> даже если встретит `NULL`.

```c++
#include <iostream>
#include <string>

int main() {
    const char *szSomething("What a string!");
    std::string sOutput(szSomething, 7); // 7 первых символов
    std::cout << sOutput << std::endl; // What a

    return 0;
}
```

#### `string::string(size_type nNum, char chChar)`
* Конструктор, который создает новую строку, инициализированную символом\
  `chChar` и требуемым количеством вхождений этого символа (указывается в\
  `nNum`).
* Если размер результата превышает максимальную длину строки, то\
  генерируется исключение `length_error`.

```c++
#include <iostream>
#include <string>

int main() {
    std::string sOutput(5, 'G');
    std::cout << sOutput << std::endl; // GGGGG

    return 0;
}
```

#### `template string::string(InputIterator itBeg, InputIterator itEnd)`
* Конструктор, который создает новую строку, инициализированную\
  символами диапазона `[itBeg, itEnd)`.
* Если размер результата превышает максимальную длину строки, то\
  генерируется исключение `length_error`.

**Нет примера, так как почти не используется.**

#### `string::~string()`
* Деструктор, который уничтожает строку и освобождает память.

**Примера нет, так как деструктор вызывается неявно.**

### Создание `std::string` из чисел
Задействуем **класс `std::ostringstream`** для конвертации:
```c++
#include <iostream>
#include <sstream>
#include <string>

template<typename T>
inline std::string ToString(T tX) {
    std::ostringstream oStream;
    oStream << tX;
    return oStream.str();
}

int main() {
    std::string sFive(ToString(5));
    std::string sSevenPointEight(ToString(7.8));
    std::string sB(ToString('B'));
    std::cout << sFive << std::endl; // 5
    std::cout << sSevenPointEight << std::endl; // 7.8
    std::cout << sB << std::endl; // B
}
```

### Конвертация `std::string` в числа
С помощью `std::istringstream` можно выполнять обратную конвертацию:
```c++
#include <iostream>
#include <sstream>
#include <string>

template<typename T>
inline bool FromString(const std::string &sString, T &tX) {
    std::istringstream iStream(sString);
    return (iStream >> tX) ? true : false; // извлекаем значение в tX,
    // возвращаем true (если удачно) или false (если неудачно)
}

int main() {
    double dX;
    if (FromString("4.5", dX)) // dX остается типа double
        std::cout << dX << std::endl; // 4.5
    if (FromString("TOM", dX)) // false
        std::cout << dX << std::endl;
}
```

## [Урок №210. Длина и ёмкость `std::string`](#урок-210-длина-и-ёмкость-stdstring)
### Длина `std::string`
**Длина строки** — это количество символов, которые она содержит. Есть две\
идентичные функции для определения длины строки:
* `size_type string::length() const`
* `size_type string::size() const`

Обе эти функции возвращают текущее количество символов, которые содержит\
строка, исключая нуль-терминатор: 
```c++
#include <iostream>

int main() {
    std::string sSomething("012345");
    std::cout << sSomething.length() << std::endl; // 6

    return 0;
}
```

Хотя для определения того, содержит ли строка какие-либо символы или нет, также можно\
использовать функцию `length()`, эффективнее, однако, использовать функцию `empty()`:
* `bool string::empty() const` — эта функция возвращает `true`, если в строке нет\
  символов, и `false` — в противном случае.

```c++
#include <iostream>

int main() {
    std::string sString1("Not Empty");
    std::cout << (sString1.empty() ? "true" : "false") << std::endl; // false
    std::string sString2; // пустая строка
    std::cout << (sString2.empty() ? "true" : "false") << std::endl; // true

    return 0;
}
```

Есть еще одна функция, связанная с длиной строки, которую вы, вероятно, никогда\
не будете использовать, но мы все равно её рассмотрим:
* `size_type string::max_size() const` — эта функция возвращает максимальное\
  количество символов, которое может хранить строка. Это значение может\
  варьироваться в зависимости от операционной системы и архитектуры\
  операционной системы.

```c++
#include <iostream>

int main() {
    std::string  sString("MyString");
    std::cout << sString.max_size() << std::endl; // 9223372036854775807

    return 0;
}
```

### Ёмкость `std::string`
**Ёмкость строки** — это максимальный объем памяти, выделенный строке для\
хранения содержимого. Это значение измеряется в символах строки, исключая\
нуль-терминатор. Например, строка с ёмкостью 8 может содержать 8 символов.
* `size_type string::capacity() const` — эта функция возвращает количество\
  символов, которое может хранить строка без дополнительного\
  перераспределения/перевыделения памяти.

```c++
#include <iostream>

int main() {
    std::string sString("0123456789");
    std::cout << "Length: " << sString.length() << std::endl; // Length: 10
    std::cout << "Capacity: " << sString.capacity() << std::endl; // Capacity: 15

    return 0;
}
```

```c++
#include <iostream>

int main() {
    std::string sString("0123456789abcde");
    std::cout << "Length: " << sString.length() << std::endl; // Length: 15
    std::cout << "Capacity: " << sString.capacity() << std::endl; // Capacity: 15

    // Добавляем новый символ
    sString += "f";
    std::cout << "Length: " << sString.length() << std::endl; // Length: 16
    std::cout << "Capacity: " << sString.capacity() << std::endl; // Capacity: 30 - буфер данных
    // увеличивается в 2 раза

    return 0;
}
```

Есть еще одна функция (а точнее 2 варианта этой функции) для работы с ёмкостью\
строки:
* `void string::reserve(size_type unSize)` — при вызове этой функции мы\
  устанавливаем ёмкость строки, равную, как минимум, `unSize` (она может\
  быть и больше). Обратите внимание, для выполнения этой функции может\
  потребоваться перераспределение.
* `void string::reserve()` — если вызывается эта функция или вышеприведенная\
  функция с `unSize` меньше текущей ёмкости, то компилятор попытается\
  срезать (уменьшить) ёмкость строки до размера её длины. Это\
  необязательный запрос.

```c++
#include <iostream>

int main() {
    std::string sString("0123456789");
    std::cout << "Length: " << sString.length() << std::endl; // Length: 10
    std::cout << "Capacity: " << sString.capacity() << std::endl; // Capacity: 15

    sString.reserve(300);
    std::cout << "Length: " << sString.length() << std::endl; // Length: 10
    std::cout << "Capacity: " << sString.capacity() << std::endl; // Capacity: 300

    sString.shrink_to_fit(); // так как reserve() для подгонки к размеру строки - deprecated
    std::cout << "Length: " << sString.length() << std::endl; // Length: 10
    std::cout << "Capacity: " << sString.capacity() << std::endl; // Capacity: 15
    
    return 0;
}
```

Так, зная, что нужно строка побольше, мы можем сразу же зарезервировать\
больше памяти:
```c++
#include <iostream>
#include <string>
#include <cstdlib> // для rand() и srand()
#include <ctime> // для time()

int main() {
    srand(time(nullptr)); // генерация случайного числа

    std::string sString; // длина 0
    sString.reserve(80); // сразу резервируем 80 символов 

    // Заполняем строку случайными строчными символами
    for (int nCount = 0; nCount < 80; ++nCount)
        sString += 'a' + rand() % 26;

    std::cout << sString; // разыный рандомный результат вида
    // rjjzrgebbcufikxxwtkydqafrsisvapojyndergfubnelkbhgnhjfkpwezpazeolccoivuqrvdvinzqv
}
```

## [Урок №211. Доступ к символам `std::string`. Конвертация `std::string` в строки C-style](#урок-211-доступ-к-символам-stdstring-конвертация-stdstring-в-строки-c-style)
### Доступ к символам `std::string`
Есть два практически идентичных способа доступа к символам `std::string`. Наиболее\
простой и быстрый — использовать перегруженный оператор индексации `[]`.

#### `char& string::operator[](size_type nIndex)` <br> `const char& string::operator[](size_type nIndex) const`
* Обе эти функции возвращают символ под индексом `nIndex`.
* Передача неверного индекса приведет к неопределенным результатам.
* Использование функции `length()` в качестве индекса допустимо только для\
  константных строк и возвращает значение, сгенерированное конструктором\
  по умолчанию `std::string`. Это не рекомендуется делать.
* Поскольку `char&` — это тип возврата, то вы можете использовать его для\
  изменения символов строки.

```c++
#include <iostream>
#include <string>

int main() {
    std::string sSomething("abcdefg");
    std::cout << sSomething[4] << std::endl; // e
    sSomething[4] = 'A';
    std::cout << sSomething << std::endl; // abcdAfg
}
```

Другой способ доступа к символам `std::string` медленнее, чем вышеприведенный\
вариант, так как использует исключения для проверки корректности `nIndex`. Если\
вы не уверены в корректности передаваемого `nIndex`, то вы должны использовать\
именно этот способ (тот, что описан ниже) для доступа к символам строки.

#### `char& string::at(size_type nIndex)` <br> `const char& string::at(size_type nIndex) const`
* Обе эти функции возвращают символ под индексом `nIndex`.
* Передача неверного индекса приведет к генерации исключения `out_of_range`.
* Поскольку `char&` — это тип возврата, то вы можете использовать его для\
  изменения символов строки.

```c++
#include <iostream>
#include <string>

int main() {
    std::string sSomething("abcdefg");
    std::cout << sSomething.at(4) << std::endl; // e
    sSomething.at(4) = 'A';
    std::cout << sSomething << std::endl; // abcdAfg
}
```

### Конвертация `std::string` в строки C-style
#### `const char* string::c_str() const`
* Возвращает содержимое `std::string` в виде константной строки C-style.
* Добавляется нуль-терминатор.
* Строка C-style принадлежит `std::string` и не должна быть удалена.

```c++
#include <iostream>
#include <string>
#include <cstring>

int main() {
    std::string sSomething("abcdefg");
    std::cout << strlen(sSomething.c_str()); // 7
}
```

#### `const char* string::data() const`
* Возвращает содержимое `std::string` в виде константной строки C-style.
* Не добавляется нуль-терминатор.
* Строка C-style принадлежит `std::string` и не должна быть удалена.

```c++
#include <iostream>
#include <string>
#include <cstring>

int main() {
    std::string sSomething("abcdefg");
    const char *szString = "abcdefg";
    // Функция memcmp() сравнивает две вышеприведенные строки C-style и возвращает 0, если они равны
    if (memcmp(sSomething.data(), szString, sSomething.length()) == 0) // последний аргумент не обязательный
        std::cout << "The strings are equal"; // The strings are equal
    else
        std::cout << "The strings are not equal";
}
```

#### `size_type string::copy(char *szBuf, size_type nLength) const` <br> `size_type string::copy(char *szBuf, size_type nLength, size_type nIndex) const`
* Отличие второго варианта этой функции от первого состоит в том, что\
  копирование не более `nLength` символов передаваемой строки в `szBuf`\
  начинается с символа под индексом `nIndex`. В первой же функции\
  копирование всегда начинается с символа под индексом `[0]`.
* Количество скопированных символов возвращается.
* `Caller` отвечает за то, чтобы не произошло переполнения строки `szBuf`.

```c++
#include <iostream>
#include <string>

int main() {
    std::string sSomething("lorem ipsum dolor sit amet");

    char szBuf[20];
    int nLength = sSomething.copy(szBuf, 5, 6); // 5 символов, начиная с 6
    szBuf[nLength] = '\0'; // завершаем строку в буфере

    std::cout << szBuf << std::endl; // ipsum
}
```

### Резюме
> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/success.svg">
>   <img alt="Success" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/success.svg">
> </picture><br>
>
> Используйте `c_str()`, если не гонитесь за максимальной эффективностью.\
> Это самый простой и безопасный способ конвертации `std::string` в строки C-style.

## [Урок №212. Присваивание и перестановка значений с `std::string`](#урок-212-присваивание-и-перестановка-значений-с-stdstring)

