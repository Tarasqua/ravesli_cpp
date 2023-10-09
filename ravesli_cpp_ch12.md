# Глава №12. Виртуальные функции в C++

## Содержание
1. [Урок №170. Указатели, Ссылки и Наследование](#урок-170-указатели-ссылки-и-наследование)
2. [Урок №171. Виртуальные функции и Полиморфизм](#урок-171-виртуальные-функции-и-полиморфизм)

## [Урок №170. Указатели, Ссылки и Наследование](#урок-170-указатели-ссылки-и-наследование)
```c++
#include <iostream>

class Parent {
public:
    int m_value;
public:
    explicit Parent(int value) : m_value(value) {}

    const char *getName() { return "Parent"; }

    int &getValue() { return m_value; }
};

class Child : public Parent {
public:
    explicit Child(int value) : Parent(value) {}

    const char *getName() { return "Child"; }

    int getValueDoubled() { return m_value * 2; }
};


int main() {
    Child child(7);
    std::cout << "child is a " << child.getName() << " and has value " <<
              child.getValue() << '\n'; // child is a Child and has value 7
              
    // Мы можем дать команду указателям и ссылкам класса Child указывать на другие
    // объекты класса Child:
    Child &rChild = child;
    std::cout << "rChild is a " << rChild.getName() << " and has value " <<
              rChild.getValue() << '\n'; // rChild is a Child and has value 7
    Child *pChild = &child;
    std::cout << "pChild is a " << pChild->getName() <<
              " and has value " << pChild->getValue() << '\n'; // pChild is a Child and has value 7
              
    // также мы можем дать команду указателю или ссылке класса Parent указывать на 
    // объект класса Child:
    Parent &rParent = child;
    Parent *pParent = &child;
    std::cout << "child is a " << child.getName() << " and has value " <<
              child.getValue() << '\n'; // child is a Child and has value 7
    std::cout << "rParent is a " << rParent.getName() << " and has value " <<
              rParent.getValue() << '\n'; // rParent is a Parent and has value 7
    std::cout << "pParent is a " << pParent->getName() <<
              " and has value " << pParent->getValue() << '\n'; // pParent is a Parent and has value 7
    return 0;
}
```
Поскольку `rParent` и `pParent` являются ссылкой и указателем класса `Parent`, то\
они могут видеть только члены класса `Parent` (и члены любых других классов,\
которые наследует `Parent`). Таким образом, указатель/ссылка класса `Parent` не\
может увидеть `Child::getName()`. Следовательно, вызывается `Parent::getName()`, а\
`rParent` и `pParent` сообщают, что они относятся к классу `Parent`, а не к классу\
`Child`.

Обратите внимание, это также означает, что невозможно вызвать\
`Child::getValueDoubled()` через `rParent` или `pParent`. Они не могут видеть что-\
либо в классе `Child`.
```c++
std::cout << rParent.getValueDoubled() // ошибка
std::cout << pParent.getValueDoubled() // ошибка
```

Еще один пример:
```c++
#include <iostream>
#include <string>
#include <utility>

class Animal {
protected:
    std::string m_name;

    // Мы делаем этот конструктор protected так как не хотим, чтобы пользователи
    // создавали объекты класса Animal напрямую,
    // но хотим, чтобы у дочерних классов доступ был открыт
    Animal(std::string name)
            : m_name(std::move(name)) {
    }

public:
    std::string getName() { return m_name; }

    const char *speak() { return "???"; }
};

class Cat : public Animal {
public:
    Cat(std::string name)
            : Animal(std::move(name)) {
    }

    const char *speak() { return "Meow"; }
};

class Dog : public Animal {
public:
    Dog(std::string name)
            : Animal(std::move(name)) {
    }

    const char *speak() { return "Woof"; }
};

int main() {
    Cat cat("Matros");
    std::cout << "cat is named " << cat.getName() <<
              ", and it says " << cat.speak() << '\n'; // cat is named Matros, and it says Meow

    Dog dog("Barsik");
    std::cout << "dog is named " << dog.getName() <<
              ", and it says " << dog.speak() << '\n'; // dog is named Barsik, and it says Woof

    Animal *pAnimal = &cat;
    std::cout << "pAnimal is named " << pAnimal->getName() <<
              ", and it says " << pAnimal->speak() << '\n'; // pAnimal is named Matros, and it says ???

    pAnimal = &dog;
    std::cout << "pAnimal is named " << pAnimal->getName() <<
              ", and it says " << pAnimal->speak() << '\n'; // pAnimal is named Barsik, and it says ???

    return 0;
}
```
Мы видим здесь ту же проблему. Поскольку `pAnimal` является указателем типа\
`Animal`, то он может видеть только часть `Animal`. Следовательно,\
`pAnimal->speak()` вызывает `Animal::speak()`, а не `Dog::Speak()`\
или `Cat::speak()`.

### Указатели, ссылки и родительские классы
```c++
int main() {
    Cat matros("Matros"), ivan("Ivan"), martun("Martun");
    Dog barsik("Barsik"), tolik("Tolik"), tyzik("Tyzik");

    // Создаем массив указателей на наши объекты Cat и Dog
    Animal *animals[] = { &matros, &ivan, &martun, &barsik, &tolik, &tyzik};
    for (auto const animal : animals)
        std::cout << animal->getName() << " says " << animal->speak() << '\n';

    return 0;
}
```
Хотя это скомпилируется и выполнится, но, к сожалению, тот факт, что каждый\
элемент массива `animals` является указателем на `Animal`, означает, что\
`animal->speak()` будет вызывать `Animal::speak()`, вместо методов `speak()`\
дочерних классов.

### Тест
```c++
#include <iostream>
#include <string>
#include <utility>

class Animal {
protected:
    std::string m_name;
    const char *m_speak;

    // Мы делаем этот конструктор protected так как не хотим,
    // чтобы пользователи могли создавать объекты класса Animal напрямую,
    // но хотим, чтобы у дочерних классов доступ был открыт
    Animal(std::string name, const char *speak)
            : m_name(std::move(name)), m_speak(speak) {}

public:
    std::string getName() { return m_name; }

    const char *speak() { return m_speak; }
};

class Cat : public Animal {
public:
    Cat(std::string name) : Animal(std::move(name), "Meow") {}
};

class Dog : public Animal {
public:
    Dog(std::string name) : Animal(std::move(name), "Woof") {}
};

int main() {
    Cat matros("Matros"), ivan("Ivan"), martun("Martun");
    Dog barsik("Barsik"), tolik("Tolik"), tyzik("Tyzik");

    // Создаем массив указателей на наши объекты Cat и Dog
    Animal *animals[] = {&matros, &ivan, &martun, &barsik, &tolik, &tyzik};
    for (auto const animal: animals)
        std::cout << animal->getName() << " says " << animal->speak() << '\n';

    return 0;
}
```

```c++
<<< Matros says Meow
<<< Ivan says Meow
<<< Martun says Meow
<<< Barsik says Woof
<<< Tolik says Woof
<<< Tyzik says Woof
```

## [Урок №171. Виртуальные функции и Полиморфизм](#урок-171-виртуальные-функции-и-полиморфизм)
**Виртуальная функция в языке С++** — это особый тип функции, которая, при её\
вызове, выполняет «наиболее» дочерний метод, который существует между\
родительским и дочерними классами. Это свойство еще известно, как\
**полиморфизм**. Дочерний метод вызывается тогда, когда совпадает сигнатура (имя,\
типы параметров и является ли метод константным) и тип возврата дочернего\
метода с сигнатурой и типом возврата метода родительского класса. Такие методы\
называются **переопределениями** (или **"переопределенными методами"**).

```c++
#include <iostream>
#include <string>
#include <utility>

class Parent {
public:
    virtual const char *getName() { return "Parent"; } // добавили ключевое слово virtual
};

class Child : public Parent {
public:
    virtual const char *getName() { return "Child"; }
};

int main() {
    Child child;
    Parent &rParent = child;
    std::cout << "rParent is a " << rParent.getName(); // rParent is a Child

    return 0;
}
```

### Более сложный пример
```c++
#include <iostream>
#include <string>
#include <utility>

class Animal {
protected:
    std::string m_name;

    // Мы делаем этот конструктор protected так как не хотим, чтобы пользователи
    // создавали объекты класса Animal напрямую,
    // но хотим, чтобы у дочерних классов доступ был открыт
    Animal(std::string name)
            : m_name(std::move(name)) {
    }

public:
    std::string getName() { return m_name; }

    virtual const char *speak() { return "???"; }
};

class Cat : public Animal {
public:
    Cat(std::string name)
            : Animal(std::move(name)) {
    }

    virtual const char *speak() { return "Meow"; }
};

class Dog : public Animal {
public:
    Dog(std::string name)
            : Animal(std::move(name)) {
    }

    virtual char *speak() { return "Woof"; }
};


void report(Animal &animal) {
    std::cout << animal.getName() << " says " << animal.speak() << '\n';
}

int main() {
    Cat cat("Matros");
    Dog dog("Barsik");

    report(cat); // Matros says Meow
    report(dog); // Barsik says Woof

    return 0;
}
```

При обработке `animal.speak()`, компилятор видит, что метод `Animal::speak()`\
является виртуальной функцией. Когда animal ссылается на часть `Animal` объекта\
`cat`, то компилятор просматривает все классы между `Animal` и `Cat`, чтобы найти\
наиболее дочерний метод `speak()`. И находит `Cat::speak()`. В случае, когда\
`animal` ссылается на часть `Animal` объекта `dog`, компилятор находит `Dog::speak()`.

Также сработает:
```c++
Cat matros("Matros"), ivan("Ivan"), martun("Martun");
Dog barsik("Barsik"), tolik("Tolik"), tyzik("Tyzik");

// Создаем массив указателей на наши объекты Cat и Dog
Animal *animals[] = { &matros, &barsik, &ivan, &tolik, &martun, &tyzik};
for (auto const & animal : animals)
    std::cout << animal->getName() << " says " << animal->speak() << '\n';
```

> <picture>
>   <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/light-theme/danger.svg">
>   <img alt="Danger" src="https://raw.githubusercontent.com/Mqxx/GitHub-Markdown/main/blockquotes/badge/dark-theme/danger.svg">
> </picture><br>
>
> Сигнатура виртуального метода дочернего класса должна полностью\
> соответствовать сигнатуре виртуального метода родительского класса.\
> Если у дочернего метода будет другой тип параметров, нежели у\
> родительского, то вызываться этот метод не будет.

