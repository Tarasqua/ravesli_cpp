## Списки инициализации
Списки инициализации представляют перечисления инициализаторов для\
каждой из переменных и констант через двоеточие после списка параметров конструктора:
```c++
class Person 
{
    const std::string name;
    unsigned age;
public:
    void print() 
    {
        std::cout << "Name: " << name << "\tAge: " << age << std::endl;
    }
    Person(std::string p_name, unsigned p_age) : name(p_name), age(p_age)  // списки инициализации
    { }
};
```