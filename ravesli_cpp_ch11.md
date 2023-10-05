# Глава №11. Наследование в C++
## Оглавление
1. [Урок №162. Базовое наследование](#урок-162-базовое-наследование)
2. [Урок №163. Порядок построения дочерних классов](#урок-163-порядок-построения-дочерних-классов)
3. [Урок №164. Конструкторы и инициализация дочерних классов](#урок-164-конструкторы-и-инициализация-дочерних-классов)
4. [Урок №165. Наследование и спецификатор доступа `protected`](#урок-165-наследование-и-спецификатор-доступа-protected)

## [Урок №162. Базовое наследование](#урок-162-базовое-наследование)
Класс, от которого наследуют, называется **родительским** (или **"базовым"**,\
**"суперклассом"**), а класс, который наследует, называется **дочерним**\
(или **"производным"**, **"подклассом"**).

```c++
#include <iostream>
#include <string>

class Human {
public:
    std::string m_name;
    int m_age;

    explicit Human(std::string name = "", int age = 0) : m_name(std::move(name)), m_age(age) {}

    std::string &getName() { return m_name; }

    int &getAge() { return m_age; }
};

// BasketballPlayer открыто наследует Human
class BasketballPlayer : public Human {
public:
    double m_gameAverage;
    int m_points;

    explicit BasketballPlayer(double gameAverage = 0.0, int points = 0)
            : m_gameAverage(gameAverage), m_points(points) {}

    void printNameAndPoints() const {
        std::cout << m_name << ": " << m_points << '\n'; // используем переменные родительского класса
    }
};


int main() {
    BasketballPlayer kirill; // Создаем нового Баскетболиста
    kirill.m_name = "Kirill"; // Присваиваем ему имя (public)
    std::cout << kirill.getName(); // используем метод getName(), // Kirill
    // который мы унаследовали от класса Human
    kirill.m_points = 12;
    kirill.printNameAndPoints(); // Kirill: 12
    return 0;
}
```

## [Урок №163. Порядок построения дочерних классов](#урок-163-порядок-построения-дочерних-классов)
Когда C++ создает объекты дочерних классов, то он делает это поэтапно. Сначала\
создается самый верхний класс иерархии (тот, который родитель). Затем создается\
дочерний класс, который идет следующим по порядку, и так до тех пор, пока не будет\
создан последний класс (тот, который находится в самом низу иерархии).

### Резюме
Язык C++ выполняет построение дочерних классов поэтапно, начиная с верхнего\
класса иерархии и заканчивая нижним классом иерархии. По мере построения\
каждого класса для выполнения инициализации вызывается соответствующий\
конструктор соответствующего класса.

## [Урок №164. Конструкторы и инициализация дочерних классов](#урок-164-конструкторы-и-инициализация-дочерних-классов)
### Конструкторы и инициализация
```c++
class Parent {
public:
    int m_id;

    Parent(int id = 0) : m_id(id) {}

    int getId() const { return m_id; }
};

class Child : public Parent {
public:
    double m_value;

    Child(double value = 0.0) : m_value(value) {}

    double getValue() const { return m_value; }
};
```

```c++
int main() {
    Child child(1.5); // вызывается конструктор Child(double)
    return 0;
}
```

Вот что происходит при инициализации объекта `child`:
* выделяется память для объекта дочернего класса (достаточная порция\
  памяти для части `Parent` и части `Child` объекта класса `Child`);
* вызывается соответствующий конструктор класса `Child`;
* создается объект класса `Parent` с использованием соответствующего\
  конструктора класса `Parent`. Если такой конструктор программистом не\
  предоставлен, то будет использоваться конструктор по умолчанию класса\
  `Parent`;
* список инициализации инициализирует переменные;
* выполняется тело конструктора класса `Child`;
* точка выполнения возвращается обратно в `caller`.

Сначала выполняется конструктор родительского класса (для инициализации части\
родительского класса) и только потом уже выполняется конструктор дочернего\
класса.

### Инициализация членов родительского класса
Правильная инициализация `m_id` при создани объекта класса `Child`:
```c++
#include <iostream>

class Parent {
private:
    int m_id;
public:
    Parent(int id = 0) : m_id(id) {}

    int getId() const { return m_id; }
};

class Child : public Parent {
private:
    double m_value;
public:
    Child(double value = 0.0, int id = 0)
            : Parent(id), // вызывается конструктор Parent(int) со значением id!
              m_value(value) {}

    double getValue() const { return m_value; }
};

int main() {
    Child child(1.5, 7); // вызывается конструктор Child(double, int)
    std::cout << "ID: " << child.getId() << '\n'; // 7
    std::cout << "Value: " << child.getValue() << '\n'; // 1.5

    return 0;
}
```

### Еще один пример
```c++
#include <iostream>
#include <string>
#include <utility>

class Human {
private:
    std::string m_name;
    int m_age;
public:
    explicit Human(std::string name = "", int age = 0)
            : m_name(std::move(name)), m_age(age) {}

    std::string getName() const { return m_name; }

    int getAge() const { return m_age; }
};

// BasketballPlayer открыто наследует класс Human
class BasketballPlayer : public Human {
private:
    double m_gameAverage;
    int m_points;
public:
    explicit BasketballPlayer(std::string name = "", int age = 0,
                              double gameAverage = 0.0, int points = 0)
            : Human(std::move(name), age), // вызывается Human(std::string, int) для инициализации членов name и age
              m_gameAverage(gameAverage), m_points(points) {
    }

    double getGameAverage() const { return m_gameAverage; }

    int getPoints() const { return m_points; }
};

int main() {
    BasketballPlayer anton("Anton Ivanovuch", 45, 300, 310);

    std::cout << anton.getName() << '\n'; // Anton Ivanovuch
    std::cout << anton.getAge() << '\n'; // 45
    std::cout << anton.getPoints() << '\n'; // 310

    return 0;
}
```

### Деструкторы
**При уничтожении дочернего класса, каждый деструктор вызывается в порядке\
обратном построению классов.**

### Резюме
При инициализации объектов дочернего класса, конструктор дочернего класса\
отвечает за то, какой конструктор родительского класса вызывать. Если этот\
конструктор явно не указан, то вызывается конструктор по умолчанию\
родительского класса. Если же компилятор не может найти конструктор по\
умолчанию родительского класса (или этот конструктор не может быть создан\
автоматически), то компилятор выдаст ошибку.

### Тест
```c++
#include <iostream>
#include <string>
#include <utility>

class Fruit {
private:
    std::string m_name;
    std::string m_color;
public:
    Fruit(std::string name, std::string color)
            : m_name(std::move(name)), m_color(std::move(color)) {
    }

    std::string getName() const { return m_name; }

    std::string getColor() const { return m_color; }
};

class Apple : public Fruit {
private:
    double m_fiber;
public:
    Apple(std::string name, std::string color, double fiber)
            : Fruit(std::move(name), std::move(color)), m_fiber(fiber) {
    }

    double getFiber() const { return m_fiber; }

    friend std::ostream &operator<<(std::ostream &out, const Apple &a) {
        out << "Apple (" << a.getName() << ", " << a.getColor() << ", " <<
            a.getFiber() << ")\n";
        return out;
    }
};

class Banana : public Fruit {
public:
    Banana(std::string name, std::string color)
            : Fruit(std::move(name), std::move(color)) {
    }

    friend std::ostream &operator<<(std::ostream &out, const Banana &b) {
        out << "Banana (" << b.getName() << ", " << b.getColor() << ")\n";
        return out;
    }
};

int main() {
    const Apple a("Red delicious", "red", 7.3);
    std::cout << a; // Apple (Red delicious, red, 7.3)
    const Banana b("Cavendish", "yellow");
    std::cout << b; // Banana (Cavendish, yellow)
    return 0;
}
```

## [Урок №165. Наследование и спецификатор доступа `protected`](#урок-165-наследование-и-спецификатор-доступа-protected)
