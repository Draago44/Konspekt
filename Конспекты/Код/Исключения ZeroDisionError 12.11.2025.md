#Код #Исключения #Программа #Циклы #Функции 
```

is_running = True

  

while is_running:

    raw = input("Программа для проверки DivisionByZero. Выход или продлить?\n")

    if raw == "exit" :

         is_running = False                        

         print("Прграмма завершена.")

  

    elif raw == "Продолжить":

        x = int(input("Делимое"))

        y = int(input("Делитель"))

        z = 0

        is_running = False

        print("Программа завершена.")

  

    try:

         print("Программа работает штатно:", z)

         z = x/y

    except ZeroDivisionError:

         z = 0

         print("Исключение: деление на 01", z)

  
  

else:

     print("Неизвестная команда.")

```