#Python  #Инкапсуляция #Код #Наследование #Полиморфизм #Абстракция #Методы 
### Инкапсуляция

**Инкапсуляция** объединяет данные (атрибуты) и методы в единый объект, скрывая внутренние детали реализации и защищая данные от нежелательных изменений. В Python существует три уровня доступа к атрибутам и методам:​

- **Публичный** (public) — без префикса, доступен везде
    
- **Защищённый** (protected) — с одним подчеркиванием (_атрибут), по соглашению используется только внутри класса и его наследников
    
- **Приватный** (private) — с двойным подчеркиванием (__атрибут), недоступен извне
    

Пример инкапсуляции:


```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance  # приватный атрибут
    
    def get_balance(self):
        return self.__balance
    
    def set_balance(self, balance):
        if balance > 0:
            self.__balance = balance
        else:
            print("Баланс не может быть отрицательным")

# Использование
account = BankAccount(1000)
print(account.get_balance())  # 1000
account.set_balance(1500)
print(account.get_balance())  # 1500
```

В этом примере приватный атрибут `__balance` защищён от прямого доступа, а изменения происходят только через методы, которые проверяют корректность данных.​

### Наследование

**Наследование** позволяет создавать новые классы на основе существующих, наследуя их атрибуты и методы. Класс-наследник (подкласс) может добавлять новые методы, переопределять существующие или расширять функциональность базового класса.​

Пример наследования:

```python
class Animal:
    def __init__(self, name):
        self.name = name
    
    def speak(self):
        return f"{self.name} издаёт звук"

class Dog(Animal):
    def speak(self):
        return f"{self.name} гавкает: Гав!"

class Cat(Animal):
    def speak(self):
        return f"{self.name} мяукает: Мяу!"

# Использование
dog = Dog("Шарик")
cat = Cat("Мурзик")
print(dog.speak())  # Шарик гавкает: Гав!
print(cat.speak())  # Мурзик мяукает: Мяу!
```

В этом примере классы `Dog` и `Cat` наследуют атрибут `name` и метод `__init__` от класса `Animal`, но переопределяют метод `speak()` со своей реализацией.​

### Полиморфизм

**Полиморфизм** — это возможность использовать один интерфейс для объектов разных классов, где одинаковые методы выполняют разные действия в зависимости от типа объекта. Это особенно полезно, когда вы работаете с коллекциями объектов разных типов
Пример полиморфизма:

```python
class Predator:
    def hunt(self):
        pass

class Lion(Predator):
    def hunt(self):
        return "Лев охотится на газель"

class Snake(Predator):
    def hunt(self):
        return "Змея охотится на грызуна"

class Eagle(Predator):
    def hunt(self):
        return "Орёл охотится на рыбу"

# Полиморфная функция
def start_hunting(predator):
    print(predator.hunt())

# Использование
lion = Lion()
snake = Snake()
eagle = Eagle()

start_hunting(lion)    # Лев охотится на газель
start_hunting(snake)   # Змея охотится на грызуна
start_hunting(eagle)   # Орёл охотится на рыбу
```

Функция `start_hunting()` работает с любым объектом класса-наследника `Predator`, хотя каждый из них реализует метод `hunt()` по-своему.​

### Абстракция

**Абстракция** выделяет ключевые характеристики объекта, упрощая его описание и игнорируя несущественные детали. Класс выступает как абстрактная модель, описывающая набор атрибутов и методов, необходимых для решения конкретной задачи.​

Пример абстракции:


```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    @abstractmethod
    def start(self):
        pass
    
    @abstractmethod
    def stop(self):
        pass

class Car(Vehicle):
    def start(self):
        return "Автомобиль заводится"
    
    def stop(self):
        return "Автомобиль останавливается"

class Bicycle(Vehicle):
    def start(self):
        return "Велосипед начинает движение"
    
    def stop(self):
        return "Велосипед останавливается"

# Использование
car = Car()
bike = Bicycle()

print(car.start())    # Автомобиль заводится
print(bike.start())   # Велосипед начинает движение
```

Абстрактный класс `Vehicle` определяет общий интерфейс, который должны реализовать все конкретные виды транспортных средств.​

### Практический пример: система напитков

Вот более сложный пример, объединяющий все принципы:

```python
class Drink:
    volume = 200  # статический атрибут
    
    def __init__(self, name, price):
        self.name = name
        self.price = price
        self._remains = self.volume  # защищённый атрибут
    
    def sip(self):
        if self._remains >= 20:
            self._remains -= 20
            return f"Глоток {self.name}. Осталось {self._remains} мл"
        else:
            return "Напитка недостаточно"
    
    def drink_all(self):
        self._remains = 0
        return f"{self.name} закончился"
    
    def drink_info(self):
        return f"Напиток: {self.name}, цена: {self.price} руб."

class Juice(Drink):
    _juice_name = "Сок"
    
    def __init__(self, taste, price):
        super().__init__(self._juice_name, price)
        self.taste = taste
    
    def drink_info(self):
        return f"Напиток: {self.name} ({self.taste}), цена: {self.price} руб."

# Использование
coffee = Drink("Кофе", 150)
juice = Juice("Апельсиновый", 100)

print(coffee.drink_info())     # Напиток: Кофе, цена: 150 руб.
print(coffee.sip())             # Глоток Кофе. Осталось 180 мл
print(juice.drink_info())      # Напиток: Сок (Апельсиновый), цена: 100 руб.
print(juice.sip())             # Глоток Сок. Осталось 180 мл


```

Этот пример демонстрирует:

- **Инкапсуляцию**: защищённый атрибут `_remains` изменяется только через методы
    
- **Наследование**: класс `Juice` наследует от `Drink`
    
- **Полиморфизм**: метод `drink_info()` переопределён в подклассе
    
- **Абстракцию**: класс `Drink` представляет общую концепцию напитка​
    

### Специальные методы

Python поддерживает специальные методы для расширения функциональности классов:

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __str__(self):
        return f"Точка ({self.x}, {self.y})"
    
    def __add__(self, other):
        return Point(self.x + other.x, self.y + other.y)
    
    def __len__(self):
        return int((self.x**2 + self.y**2)**0.5)

# Использование
p1 = Point(3, 4)
p2 = Point(1, 1)
print(str(p1))          # Точка (3, 4)
p3 = p1 + p2
print(p3.x, p3.y)      # 4 5
print(len(p1))          # 5
```