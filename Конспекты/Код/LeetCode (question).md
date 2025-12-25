#Код #Python 

Для знакового 32-битного целого числа возвращается с обратными цифрами. Если реверс приводит к выходу значения за пределы знакового 32-битного целого диапазона , то верните .xxx[-231, 231 - 1]0

Предположим, что среда не позволяет хранить 64-битные целые числа (подписанные или неподписанные).


```` Python
class Solution:
    def reverse(self, x: int) -> int:

        upper_limit = 2147483648
        lower_limit = -2147483648
        reversed_x = 0

        if x < 0:
            reversed_x = int(str(abs(x))[::-1]) * -1
        else:
            reversed_x = int(str(abs(x))[::-1]) 

        if reversed_x >= upper_limit or reversed_x < lower_limit:
            return 0

        else:
            return reversed_x

````


