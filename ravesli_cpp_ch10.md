# Глава №10. Введение в связи между объектами в C++

## Содержание
1. [Урок №155. Композиция объектов](#урок-155-композиция-объектов)
2. [Урок №156. Агрегация](#урок-156-агрегация)
3. [Урок №157. Ассоциация](#урок-157-ассоциация)
4. [Урок №158. Зависимость](#урок-158-зависимость)
5. [Урок №159. Контейнерные классы](#урок-159-контейнерные-классы)
6. [Урок №160. Список инициализации `std::initializer_list`](#урок-160-список-инициализации-stdinitializerlist)
7. [Глава №10. Итоговый тест](#глава-10-итоговый-тест)

## [Урок №155. Композиция объектов](#урок-155-композиция-объектов)
**Процесс построения сложных объектов из более простых называется\
композицией объекта.**

### Типы композиции объектов

Сложный объект часто называют **"родителем"**, в то время как простой -\
**"дочерним"**.

Существует два основных подтипа композиции объекта: **композиция** и **агрегация**.

### Композиция

**Для реализации композиции объект и часть должны иметь следующие\
отношения:**

* Часть (член) является частью объекта (класса).
* Часть (член) может принадлежать только одному объекту (классу) в моменте.
* Часть (член) существует, управляемая объектом (классом).
* Часть (член) не знает о существовании объекта (класса).

Части в композиции могут быть как **сингулярными** (единственными в своем роде),\
так и **мультипликативными** (таких частей может быть несколько).

### Реализация композиций

**Point2D.h:**

```c++
#ifndef RAVESLI_CPP_POINT2D_H
#define RAVESLI_CPP_POINT2D_H

#include <iostream>

class Point2D {
private:
    int m_x;
    int m_y;
public:
    // Конструктор по умолчанию
    Point2D() : m_x(0), m_y(0) {}

    // Специфический конструктор
    Point2D(int x, int y) : m_x(x), m_y(y) {}

    // Перегрузка оператора вывода
    friend std::ostream &operator<<(std::ostream &out, const Point2D &point) {
        out << "(" << point.m_x << ", " << point.m_y << ")";
        return out;
    }

    // Функция доступа
    void setPoint(int x, int y) {
        m_x = x;
        m_y = y;
    }
};

#endif //RAVESLI_CPP_POINT2D_H
```

**Creature.h:**

```c++
#ifndef RAVESLI_CPP_CREATURE_H
#define RAVESLI_CPP_CREATURE_H

#include <iostream>
#include <string>
#include "Point2D.h"

class Creature {
private:
    std::string m_name;
    Point2D m_location;
public:
    Creature(const std::string &name, const Point2D &location)
            : m_name(name), m_location(location) {}

    friend std::ostream &operator<<(std::ostream &out, const Creature &creature) {
        out << creature.m_name << " is at " << creature.m_location;
        return out;
    }

    void moveTo(int x, int y) {
        m_location.setPoint(x, y);
    }
};

#endif //RAVESLI_CPP_CREATURE_H
```

**main.cpp:**

```c++
#include <iostream>
#include <string>
#include "Creature.h"
#include "Point2D.h"


int main() {
    std::cout << "Enter a name for your creature: ";
    std::string name;
    std::cin >> name;
    Creature creature(name, Point2D(5, 6));
    while (true) {
        // Выводим имя существа и его местоположение
        std::cout << creature << '\n';
        std::cout << "Enter new X location for creature (-1 to quit): ";
        int x = 0;
        std::cin >> x;
        if (x == -1)
            break;
        std::cout << "Enter new Y location for creature (-1 to quit): ";
        int y = 0;
        std::cin >> y;
        if (y == -1)
            break;
        creature.moveTo(x, y);
    }

    return 0;
}
```

### Композиция и подклассы

**Преимущества композиций:**

1. Каждый отдельный класс можно сохранить относительно простым/понятным и\
   сфокусировать на выполнение одной конкретной задачи.
2. Каждый подкласс может быть автономным, что делает его многоразовым.
3. Родительский класс может оставить выполнение большей части сложной работы\
   на подклассы, а сам сосредоточиться на координации потока данных между\
   подклассами.

**Один класс должен выполнять одну конкретную задачу.**

## [Урок №156. Агрегация](#урок-156-агрегация)

### Агрегация

**Для реализации агрегации целое и его части должны соответствовать следующим\
отношениям:**

* Часть (член) является частью целого (класса).
* Часть (член) может принадлежать более чем одному целому (классу) в\
  моменте.
* Часть (член) существует, не управляемая целым (классом).
* Часть (член) не знает о существовании целого (класса).

В отличие от композиции, части могут принадлежать более чем одному целому\
в моменте, и целое не управляет существованием и продолжительностью жизни\
частей. При создании/уничтожении агрегации, целое не несет ответственности за\
создание/уничтожение своих частей.

Подобно композиции, части агрегации могут быть сингулярными или\
мультипликативными.

### Реализация агрегации

```c++
#include <iostream>
#include <string>

class Worker {
private:
    std::string m_name;
public:
    explicit Worker(std::string name) : m_name(std::move(name)) {}

    std::string getName() { return m_name; }
};

class Department {
private:
    Worker *m_worker; // чтобы было проще, в этом Отделе работает только один
    // работник, но их может быть и несколько
public:
    explicit Department(Worker *worker = nullptr) : m_worker(worker) {}
};

int main() {
    // Создаем Работника вне области видимости класса Department
    auto *worker = new Worker("Anton"); // создаем Работника
    {
        // Создаем Отдел и передаем Работника в Отдел через параметр конструктора
        Department department(worker);
    } // department выходит из области видимости и уничтожается здесь

    // worker продолжает существовать дальше
    std::cout << worker->getName() << " still exists!"; // Anton still exists!
    delete worker;

    return 0;
}
```

Здесь Работник создается независимо от Отдела, а затем переходит в параметр\
конструктора класса Отдела. Когда `department` уничтожается, то указатель\
`m_worker` уничтожается также, но сам Работник то не удаляется, он продолжает\
существовать до тех пор, пока не будет уничтожен в функции `main()`.

**Правило: Реализовывайте самые простые отношения, которые соответствуют
потребностям вашей программы, а не то, что, как вам кажется, будет лучше.**

### Композиция и агрегация

**В композиции:**

* Используются обычные переменные-члены.
* Используются указатели, если класс реализовывает собственное управление\
  памятью (происходит динамическое выделение/освобождение памяти).
* Класс ответственный за создание/уничтожение своих частей.

**В агрегации:**

* Используются указатели/ссылки, которые указывают/ссылаются на части вне\
  класса.
* Класс не несет ответственности за создание/уничтожение своих частей.

**Примечание:** По разным историческим и другим причинам, в отличие от\
композиции, определение агрегации не является единственно правильным — вы\
можете увидеть, что другие ресурсы/сайты/учебники определяют агрегацию\
несколько иначе, нежели изложено здесь. Это нормально, просто знайте об этом.

### Тест

**Задание №2.**

```c++
#include <iostream>
#include <string>
#include <vector>

class Worker {
private:
    std::string m_name;
public:
    explicit Worker(std::string name) : m_name(std::move(name)) {}

    std::string getName() { return m_name; }
};

class Department {
private:
    std::vector<Worker *> m_worker;
public:
    Department() = default;

    void add(Worker *worker) {
        m_worker.push_back(worker);
    }

    friend std::ostream &operator<<(std::ostream &out, const Department &department) {
        out << "Department: ";
        for (auto count: department.m_worker)
            out << count->getName() << ' ';
        out << '\n';
        return out;
    }
};

int main() {
    // Создаем Работников вне области видимости класса Department
    std::vector<Worker*> workers {new Worker("Egor"),
                                  new Worker("Kirill"),
                                  new Worker("Ivan")};
    {
        // Создаем Отдел и передаем Работников в качестве параметров конструктора
        Department department; // создаем пустой Отдел
        for (auto w: workers)
            department.add(w);
        std::cout << department;
    } // department выходит из области видимости и уничтожается здесь
    for (auto w: workers) {
        std::cout << w->getName() << " still exists!\n";
        delete w;
    }

    return 0;
}
```

## [Урок №157. Ассоциация](#урок-157-ассоциация)

### Ассоциация

**В ассоциации два несвязанных объекта должны соответствовать следующим\
отношениям:**

* Первый объект (член) не связан со вторым объектом (классом).
* Первый объект (член) может принадлежать одновременно сразу нескольким\
  объектам (классам).
* Первый объект (член) существует, не управляемый вторым объектом\
  (классом).
* Первый объект (член) может знать или не знать о существовании второго\
  объекта (класса).

В отличие от композиции или агрегации, где часть является частью целого, в\
ассоциации объекты между собой никак не связаны. Подобно агрегации, первый\
объект может принадлежать сразу нескольким объектам одновременно и не\
управляется ими. Однако, в отличие от агрегации, где отношения\
однонаправленные, в ассоциации отношения могут быть как однонаправленными,\
так и двунаправленными (когда оба объекта знают о существовании друг друга).

### Реализация ассоциаций

Ассоциации реализованы по-разному. Однако чаще всего они реализованы через\
указатели, где классы указывают на объекты друг друга.

В следующем примере мы реализуем двунаправленную связь между Врачом и\
Пациентом, так как Врач должен знать своих Пациентов в лицо, а Пациенты могут\
обращаться к разным Врачам:

```c++
#include <iostream>
#include <string>
#include <utility>
#include <vector>

// Поскольку отношения между этими классами двунаправленные, то для класса
// Doctor здесь нужно использовать предварительное объявление
class Doctor;

class Patient {
private:
    std::string m_name;
    std::vector<Doctor *> m_doctor; // благодаря вышеприведенному
    // предварительному объявлению Doctor, эта строка не вызовет ошибку компиляции
    // Мы объявляем метод addDoctor() закрытым, так как не хотим его публичного
    // использования.
    // Вместо этого доступ к нему будет осуществляться через
    // Doctor::addPatient().
    // Мы определим этот метод после определения класса Doctor, так как нам
    // сначала нужно определить Doctor, чтобы использовать что-либо, связанное с ним
    void addDoctor(Doctor *doc);

public:
    Patient(std::string name)
            : m_name(std::move(name)) {
    }

    // Мы реализуем перегрузку оператора вывода ниже определения класса Doctor,
    // так как он как раз и требуется для реализации перегрузки
    friend std::ostream &operator<<(std::ostream &out, const Patient &pat);

    std::string getName() const { return m_name; }

    // Мы делаем класс Doctor дружественным, чтобы иметь доступ к закрытому
    // методу addDoctor().
    // Примечание: Мы бы хотели сделать дружественным только один метод
    // addDoctor(), но мы не можем это сделать, так как Doctor предварительно объявлен
    friend class Doctor;
};

class Doctor {
private:
    std::string m_name;
    std::vector<Patient *> m_patient;
public:
    Doctor(std::string name) :
            m_name(std::move(name)) {
    }

    void addPatient(Patient *pat) {
        // Врач добавляет Пациента
        m_patient.push_back(pat);
        // Пациент добавляет Врача
        pat->addDoctor(this);
    }

    friend std::ostream &operator<<(std::ostream &out, const Doctor &doc) {
        if (doc.m_patient.empty()) {
            out << doc.m_name << " has no patients right now";
            return out;
        }
        out << doc.m_name << " is seeing patients: ";
        for (auto const d: doc.m_patient)
            out << d->getName() << ' ';
        return out;
    }

    std::string getName() const { return m_name; }
};

void Patient::addDoctor(Doctor *doc) {
    m_doctor.push_back(doc);
}

std::ostream &operator<<(std::ostream &out, const Patient &pat) {
    if (pat.m_doctor.empty()) {
        out << pat.getName() << " has no doctors right now";
        return out;
    }
    out << pat.m_name << " is seeing doctors: ";
    for (auto const p: pat.m_doctor)
        out << p->getName() << ' ';
    return out;
}

int main() {
    // Создаем Пациентов вне области видимости класса Doctor
    std::vector<Patient*> patients {
        new Patient("Anton"),
        new Patient("Ivan"),
        new Patient("Derek"),
    };
    std::vector<Doctor*> doctors {
            new Doctor("John"),
            new Doctor("Tom"),
    };
    doctors.at(0)->addPatient(patients.at(0));
    doctors.at(1)->addPatient(patients.at(0));
    doctors.at(1)->addPatient(patients.at(2));

    for (auto const d: doctors)
        std::cout << *d << '\n';
    for (auto const p: patients)
        std::cout << *p << '\n';

    for (auto const d: doctors)
        delete d;
    for (auto const p: patients)
        delete p;
    
    return 0;
}
```

Если говорить в общем, то **лучше избегать двунаправленных ассоциаций**, если для\
решения задания подходит и однонаправленная связь, так как двунаправленную\
связь написать сложнее (с учетом возникновения возможных ошибок) и она\
усложняет логику программы.

### Рефлексивная ассоциация

Иногда объекты могут иметь отношения с другими объектами того же типа. Это\
называется **рефлексивной ассоциацией**.

```c++
#include <string>

class Course
{
private:
    std::string m_name;
    Course *m_condition;
public:
    Course(std::string &name, Course *condition=nullptr):
        m_name(name), m_condition(condition) {}
};
```

Это может привести к цепочке ассоциаций (курс имеет необходимое условие,\
выполнение которого включает еще одно условие и т.д.).

### Ассоциации могут быть косвенными

В примерах, приведенных выше, мы использовали указатели для связывания\
объектов. Однако в ассоциации это не является обязательным условием. Можно\
использовать любые данные, которые позволяют связать два объекта:

```c++
#include <iostream>
#include <string>
#include <utility>

class Car {
private:
    std::string m_name;
    int m_id;
public:
    Car(std::string name, int id) : m_name(std::move(name)), m_id(id) {}

    std::string &getName() { return m_name; }

    int &getId() { return m_id; }
};

// Наш CarLot, по сути, является статическим массивом, содержащим Автомобили,
// и имеет функцию для "выдачи" Автомобилей.
// Поскольку массив является статическим, то нам не нужно создавать объекты
// для использования класса CarLot
class CarLot {
private:
    static Car s_carLot[4];
public:
    CarLot() = delete;

    static Car *getCar(int id) {
        for (auto &lot: s_carLot)
            if (lot.getId() == id)
                return &lot;
        return nullptr;
    }
};

Car CarLot::s_carLot[4] = {
        Car("Camry", 5), Car("Focus", 14),
        Car("Vito", 73), Car("Levante", 58)};

class Driver {
private:
    std::string m_name;
    int m_carId; // для связывания классов, вместо указателя, используется
    // Идентификатор (целочисленное значение)

public:
    Driver(std::string name, int carId) : m_name(std::move(name)), m_carId(carId) {}

    std::string &getName() { return m_name; }

    int &getCarId() { return m_carId; }
};

int main() {
    Driver d("Ivan", 14); // Ivan использует машину с ID 14
    Car *car = CarLot::getCar(d.getCarId()); // получаем этот Автомобиль из CarLot
    if (car)
        std::cout << d.getName() << " is driving a " << car->getName() << '\n'; // Ivan is driving a Focus
    else
        std::cout << d.getName() << " couldn't find his car\n";
    return 0;
}
```

### Композиция vs. Агрегация vs. Ассоциация

| Свойства                                                                 | Композиция       | Агрегация        | Ассоциация                                     |
|--------------------------------------------------------------------------|------------------|------------------|------------------------------------------------|
| Отношения                                                                | Части-целое      | Части-целое      | Объекты не связаны<br />между собой            |
| Члены могут<br />принадлежать одновременно<br />сразу нескольким классам | Нет              | Да               | Да                                             |
| Существование членов<br />управляется классами                           | Да               | Нет              | Нет                                            |
| Вид отношений                                                            | Однонаправленные | Однонаправленные | Однонаправленные<br />или<br />двунаправленные |
| Тип отношений                                                            | «Часть чего-то»  | «Имеет»          | «Использует»                                   |

## [Урок №158. Зависимость](#урок-158-зависимость)
**Зависимость** возникает, когда один объект обращается к функционалу другого\
объекта для выполнения определенного задания. Зависимость всегда является\
однонаправленной.

Хорошим примером зависимости, которую вы уже видели много раз, являются\
отношения классов и `std::cout` (типа `std::ostream`). Классы используют `std::cout` для\
вывода чего-либо в консоль, но не наоборот:
```c++
// класс Point не имеет прямого отношения к std::cout, но имеет зависимость от
// std::cout, так как функция перегрузки оператора << использует std::cout для вывода
// объектов класса Point в консоль.
std::ostream& operator<< (std::ostream &out, const Point &point) {
    out << "Point(" << point.m_x << ", " << point.m_y << ", " << point.m_z << ")";
    
    return out;
}
```

### Зависимость vs. Ассоциация
В языке C++ **ассоциация** — это отношения между двумя классами на уровне классов.\
То есть, первый класс сохраняет прямую или косвенную связь со вторым классом\
через переменную-член.\
**Зависимости** обычно не представлены на уровне классов, то есть зависимый объект\
не связан со вторым объектом через переменную-член. Зависимый объект обычно\
создается при необходимости.

## [Урок №159. Контейнерные классы](#урок-159-контейнерные-классы)
### Контейнерные классы
**Контейнерный класс** (или **"класс-контейнер"**) в языке C++ — это класс,\
предназначенный для хранения и организации нескольких объектов определенного\
типа данных (пользовательских или фундаментальных).

### Типы контейнерных классов
**Контейнерные классы обычно бывают двух типов:**
* **Контейнеры значения** – это композиции, которые хранят копии объектов (и,\
  следовательно, ответственные за создание/уничтожение этих копий).
* **Контейнеры ссылки** — это агрегации, которые хранят указатели или ссылки\
  на другие объекты (и, следовательно, не ответственные за\
  создание/уничтожение этих объектов).

### Контейнерный класс-массив
Целочисленный класс-массив с нуля, реализуя функционал контейнеров в языке С++.

**ArrayInt.h:**
```c++
#ifndef RAVESLI_CPP_ARRAYINT_H
#define RAVESLI_CPP_ARRAYINT_H

#include <cassert> // для assert()

class ArrayInt {
private:
    int m_length;
    int *m_data;

public:
    ArrayInt() :
            m_length(0), m_data(nullptr) {
    }

    ArrayInt(int length) :
            m_length(length) {
        assert(length >= 0);
        if (length > 0)
            m_data = new int[length];
        else
            m_data = nullptr;
    }

    ~ArrayInt() {
        delete[] m_data;
    }

    void erase() {
        delete[] m_data;
        // Здесь нужно указать m_data значение nullptr, чтобы на выходе не было висячего указателя
        m_data = nullptr;
        m_length = 0;
    }

    int &operator[](int index) {
        assert(index >= 0 && index < m_length);
        return m_data[index];
    }

    // Функция reallocate() изменяет размер массива. Все существующие элементы внутри массива будут уничтожены. 
    // Процесс быстрый
    void reallocate(int newLength) {
        // Удаляем все существующие элементы внутри массива
        erase();

        // Если наш массив должен быть пустым, то выполняем возврат здесь
        if (newLength <= 0)
            return;

        // Затем выделяем новые элементы
        m_data = new int[newLength];
        m_length = newLength;
    }

    // Функция resize() изменяет размер массива. Все существующие элементы сохраняются. Процесс медленный
    void resize(int newLength) {
        // Если массив нужной длины, то выполняем return
        if (newLength == m_length)
            return;

        // Если нужно сделать массив пустым, то делаем это и затем выполняем return
        if (newLength <= 0) {
            erase();
            return;
        }

        // Теперь предположим, что newLength состоит, по крайней мере, из одного элемента. 
        // Выполняется следующий алгоритм действий:
        // 1. Выделяем новый массив.
        // 2. Копируем элементы из существующего массива в наш только что выделенный массив.
        // 3. Уничтожаем старый массив и даем команду m_data указывать на новый массив.

        // Выделяем новый массив
        int *data = new int[newLength];

        // Затем нам нужно разобраться с количеством копируемых элементов в новый массив.
        // Нам нужно скопировать столько элементов, сколько их есть в меньшем из массивов
        if (m_length > 0) {
            int elementsToCopy = (newLength > m_length) ? m_length : newLength;

            // Поочередно копируем элементы
            for (int index = 0; index < elementsToCopy; ++index)
                data[index] = m_data[index];
        }

        // Удаляем старый массив, так как он нам уже не нужен
        delete[] m_data;

        // И используем вместо старого массива новый! Обратите внимание, m_data указывает на тот же адрес, на который 
        // указывает наш новый динамически выделенный массив.
        // Поскольку данные были динамически выделены, то они не будут уничтожены, когда выйдут из области видимости
        m_data = data;
        m_length = newLength;
    }

    void insertBefore(int value, int index) {
        // Проверка корректности передаваемого индекса
        assert(index >= 0 && index <= m_length);

        // Создаем новый массив на один элемент больше старого массива
        int *data = new int[m_length + 1];

        // Копируем все элементы аж до index
        for (int before = 0; before < index; ++before)
            data[before] = m_data[before];

        // Вставляем наш новый элемент в наш новый массив
        data[index] = value;

        // Копируем все значения после вставляемого элемента
        for (int after = index; after < m_length; ++after)
            data[after + 1] = m_data[after];

        // Удаляем старый массив и используем вместо него новый массив
        delete[] m_data;
        m_data = data;
        ++m_length;
    }

    void remove(int index) {
        // Проверка на корректность передаваемого индекса
        assert(index >= 0 && index < m_length);

        // Если это последний элемент массива, то делаем массив пустым и выполняем return
        if (m_length == 1) {
            erase();
            return;
        }

        // Cоздаем новый массив на один элемент меньше нашего старого массива
        int *data = new int[m_length - 1];

        // Копируем все элементы аж до index
        for (int before = 0; before < index; ++before)
            data[before] = m_data[before];

        // Копируем все значения после удаляемого элемента
        for (int after = index + 1; after < m_length; ++after)
            data[after - 1] = m_data[after];

        // Удаляем старый массив и используем вместо него новый массив
        delete[] m_data;
        m_data = data;
        --m_length;
    }

    // Несколько дополнительных функций просто для удобства
    void insertAtBeginning(int value) { insertBefore(value, 0); }

    void insertAtEnd(int value) { insertBefore(value, m_length); }

    int &getLength() { return m_length; }
};

#endif //RAVESLI_CPP_ARRAYINT_H
```

**main.cpp:**
```c++
#include <iostream>
#include "ArrayInt.h"

int main()
{
    // Объявляем массив с 10 элементами
    ArrayInt array(10);

    // Заполняем массив числами от 1 до 10
    for (int i=0; i<array.getLength(); i++)
        array[i] = i+1;

    // Изменяем размер массива до 7 элементов
    array.resize(7);

    // Вставляем число 15 перед элементом с индексом 4
    array.insertBefore(15, 4);

    // Удаляем элемент с индексом 5
    array.remove(5);

    // Добавляем числа 35 и 50 в конец и в начало
    array.insertAtEnd(35);
    array.insertAtBeginning(50);

    // Выводим все элементы массива
    for (int j=0; j<array.getLength(); j++)
        std::cout << array[j] << " "; // 50 1 2 3 4 15 6 7 35

    return 0;
}
```

**Примечание:** Если класс из Стандартной библиотеки C++ полностью соответствует\
вашим потребностям, то используйте его вместо написания своего контейнерного\
класса.

## [Урок №160. Список инициализации `std::initializer_list`](#урок-160-список-инициализации-stdinitializerlist)
```c++
int main() {
	ArrayInt array { 7, 6, 5, 4, 3, 2, 1 }; // эта строка вызовет ошибку компиляции
	for (int count=0; count < 7; ++count)
		std::cout << array[count] << ' ';
 
	return 0;
}
```

### Инициализация классов через `std::initializer_list`
Чтобы избежать ошибки, используем `std::initializer_list`:
```c++
// позволяем инициализацию ArrayInt через список инициализации
ArrayInt(const std::initializer_list<int> &list) : // список по конст ссылке, чтобы избежать копирования
        ArrayInt(list.size()) { // используем концепцию делегирования конструкторов для создания 
    // начального массива, в который будет выполняться копирование элементов
    // Инициализация нашего начального массива значениями из списка инициализации
    int count = 0;
    for (auto &element: list) {
        m_data[count] = element;
        ++count;
    }
}

int main() {
    ArrayInt array { 7, 6, 5, 4, 3, 2, 1 }; // список инициализации
    for (int count = 0; count < array.getLength(); ++count)
        std::cout << array[count] << ' '; // 7 6 5 4 3 2 1
    
    return 0;
}
```

### Тест
**Перегрузка оператора присвоения.**

**ArrayInt.h:**
```c++
#ifndef RAVESLI_CPP_ARRAYINT_H
#define RAVESLI_CPP_ARRAYINT_H

#include <iostream>
#include <cassert> // для assert()
#include <initializer_list>

class ArrayInt {
private:
    int m_length;
    int *m_data;

public:
    ArrayInt() :
            m_length(0), m_data(nullptr) {
    }

    ArrayInt(int length) :
            m_length(length) {
        assert(length >= 0);
        if (length > 0)
            m_data = new int[length];
        else
            m_data = nullptr;
    }

    // позволяем инициализацию ArrayInt через список инициализации
    ArrayInt(const std::initializer_list<int> &list) :
            ArrayInt(list.size()) { // используем концепцию делегирования конструкторов для создания
        // начального массива, в который будет выполняться копирование элементов
        // Инициализация нашего начального массива значениями из списка инициализации
        int count = 0;
        for (auto &element: list) {
            m_data[count] = element;
            ++count;
        }
    }

    ~ArrayInt() {
        delete[] m_data;
    }

    int &operator[](int index) {
        assert(index >= 0 && index < m_length);
        return m_data[index];
    }

    ArrayInt &operator=(const std::initializer_list<int> &list) {
        // Если новый список имеет другой размер, то перевыделяем его
        if (list.size() != static_cast<size_t>(m_length)) { // size_t тип результата, возвращаемого методом size
            // Удаляем все существующие элементы
            delete[] m_data;
            // Перевыделяем массив
            m_length = list.size();
            m_data = new int[m_length];
        }
        // Теперь инициализируем наш массив значениями из списка
        int count = 0;
        for (auto &element: list) {
            m_data[count] = element;
            ++count;
        }
        return *this;
    }

    void erase() {
        delete[] m_data;
        // Здесь нужно указать m_data значение nullptr, чтобы на выходе не было висячего указателя
        m_data = nullptr;
        m_length = 0;
    }

    // Функция reallocate() изменяет размер массива. Все существующие элементы внутри массива будут уничтожены.
    // Процесс быстрый
    void reallocate(int newLength) {
        // Удаляем все существующие элементы внутри массива
        erase();

        // Если наш массив должен быть пустым, то выполняем возврат здесь
        if (newLength <= 0)
            return;

        // Затем выделяем новые элементы
        m_data = new int[newLength];
        m_length = newLength;
    }

    // Функция resize() изменяет размер массива. Все существующие элементы сохраняются. Процесс медленный
    void resize(int newLength) {
        // Если массив нужной длины, то выполняем return
        if (newLength == m_length)
            return;

        // Если нужно сделать массив пустым, то делаем это и затем выполняем return
        if (newLength <= 0) {
            erase();
            return;
        }

        // Теперь предположим, что newLength состоит, по крайней мере, из одного элемента.
        // Выполняется следующий алгоритм действий:
        // 1. Выделяем новый массив.
        // 2. Копируем элементы из существующего массива в наш только что выделенный массив.
        // 3. Уничтожаем старый массив и даем команду m_data указывать на новый массив.

        // Выделяем новый массив
        int *data = new int[newLength];

        // Затем нам нужно разобраться с количеством копируемых элементов в новый массив.
        // Нам нужно скопировать столько элементов, сколько их есть в меньшем из массивов
        if (m_length > 0) {
            int elementsToCopy = (newLength > m_length) ? m_length : newLength;

            // Поочередно копируем элементы
            for (int index = 0; index < elementsToCopy; ++index)
                data[index] = m_data[index];
        }

        // Удаляем старый массив, так как он нам уже не нужен
        delete[] m_data;

        // И используем вместо старого массива новый! Обратите внимание, m_data указывает на тот же адрес, на который
        // указывает наш новый динамически выделенный массив.
        // Поскольку данные были динамически выделены, то они не будут уничтожены, когда выйдут из области видимости
        m_data = data;
        m_length = newLength;
    }

    void insertBefore(int value, int index) {
        // Проверка корректности передаваемого индекса
        assert(index >= 0 && index <= m_length);

        // Создаем новый массив на один элемент больше старого массива
        int *data = new int[m_length + 1];

        // Копируем все элементы аж до index
        for (int before = 0; before < index; ++before)
            data[before] = m_data[before];

        // Вставляем наш новый элемент в наш новый массив
        data[index] = value;

        // Копируем все значения после вставляемого элемента
        for (int after = index; after < m_length; ++after)
            data[after + 1] = m_data[after];

        // Удаляем старый массив и используем вместо него новый массив
        delete[] m_data;
        m_data = data;
        ++m_length;
    }

    void remove(int index) {
        // Проверка на корректность передаваемого индекса
        assert(index >= 0 && index < m_length);

        // Если это последний элемент массива, то делаем массив пустым и выполняем return
        if (m_length == 1) {
            erase();
            return;
        }

        // Cоздаем новый массив на один элемент меньше нашего старого массива
        int *data = new int[m_length - 1];

        // Копируем все элементы аж до index
        for (int before = 0; before < index; ++before)
            data[before] = m_data[before];

        // Копируем все значения после удаляемого элемента
        for (int after = index + 1; after < m_length; ++after)
            data[after - 1] = m_data[after];

        // Удаляем старый массив и используем вместо него новый массив
        delete[] m_data;
        m_data = data;
        --m_length;
    }

    // Несколько дополнительных функций просто для удобства
    void insertAtBeginning(int value) { insertBefore(value, 0); }

    void insertAtEnd(int value) { insertBefore(value, m_length); }

    int &getLength() { return m_length; }
};

#endif //RAVESLI_CPP_ARRAYINT_H
```

**main.cpp:**
```c++
#include <iostream>
#include "ArrayInt.h"


int main() {
    ArrayInt array{7, 6, 5, 4, 3, 2, 1}; // список инициализации
    // foreach невозможен, ввиду неявной длины
    for (int count = 0; count < array.getLength(); ++count)
        std::cout << array[count] << ' '; // 7 6 5 4 3 2 1 
    std::cout << '\n';
    array = {1, 4, 9, 12, 15, 17, 19, 21};
    for (int count = 0; count < array.getLength(); ++count) 
        std::cout << array[count] << ' '; // 1 4 9 12 15 17 19 21 
    return 0;
}
```

## [Глава №10. Итоговый тест](#глава-10-итоговый-тест)
### Теория
**Список инициализации `std::initializer_list`** может использоваться для реализации\
конструкторов, перегрузки оператора присваивания и других функций, которые\
принимают список инициализации в качестве параметра. `std::initailizer_list`\
находится в заголовочном файле `initializer_list`.

### Тест
**Задание №1.**\
Какие типы отношений (композиция, агрегация, ассоциация или зависимость)\
описываются ниже?

<ol type="a">
 <li>Класс Животное, который содержит Тип Животного и его Имя.</li>
 <li>Класс TextEditor с методом Save(), который принимает объект file. Метод Save()<br>
     записывает содержимое редактора на диск.</li>
 <li>Класс Авантюрист, который может хранить разные Предметы: мечи, копья, зелья<br>
     или книги заклинаний. Эти Предметы могут быть отброшены и подняты другими<br>
     Авантюристами.</li>
 <li>Программист использует Компьютер для просмотра видео с котами в Интернете.</li> </li>
 <li>Класс Компьютер, который содержит класс Процессор. Процессор можно вынуть<br>
     из Компьютера и посмотреть.</li> </li>
</ol>

**Ответ №1.a)**\
Композиция. Тип Животного и его Имя не используются вне класса Животное.

**Ответ №1.b)**\
Зависимость. Класс TextEditor использует объект file для выполнения\
определенного задания — сохранения объекта на диск.

**Ответ №1.c)**\
Агрегация. Когда Предметы связаны с Авантюристом, то Авантюрист «имеет» эти\
предметы. Меч, используемый Авантюристом, не может одновременно быть\
использован другим Авантюристом. Но Авантюрист не управляет существованием\
самих Предметов.

**Ответ №1.d)**\
Ассоциация. Программист и Компьютер не связаны между собой, за\
исключением случаев, когда Программист использует Компьютер для просмотра\
видео с котами.\

**Ответ №1.e)**\
Агрегация. Компьютер имеет Процессор, но не управляет его существованием.

**Задание №2**\
Если вы можете создать класс, используя композицию, агрегацию, ассоциацию или\
зависимость, то что вы выберите?

**Ответ №2.**\
Композиция.
