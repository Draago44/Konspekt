#Код #Программа #Задача #Исключения #Параметры #Компьютер #Python 
### Задача 1


```python
class Solution:
    def twoSum(self, nums: List[int], target: int) -> List[int]:
        
        first_op = 0
        second_op = 0

        for i in range(len(nums)):
            first_op = nums[i]
            for j in range(i+1, len(nums)):
                second_op = nums[j]
                if first_op + second_op == target:
                    return[i, j]
```


Инициализируются переменные first_op и second_op для хранения значений элементов (хотя можно обойтись без них, используя прямую nums[i] и nums[j]). Внешний цикл for i in rage(len(nums)) перебирает каждый элемент как первый в паре. Внутренний цикл for j in rage(i+1, len(nums)) перебирает элименты после i как второй в паре, что бы избежать повторений. Внутри циклов проверяется условие if first_op == target, и если оно истинно, возвращается список индексов [i, j]. Функция завершается сразу после нахождения пары.

### Задача 2

```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def addTwoNumbers(self, l1: Optional[ListNode], l2: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode(0)
        current = dummy
        carry = 0
        
        while l1 or l2 or carry:
            val1 = l1.val if l1 else 0
            val2 = l2.val if l2 else 0
            
            total = val1 + val2 + carry
            carry = total // 10
            digit = total % 10
            
            current.next = ListNode(digit)
            current = current.next
            
            if l1:
                l1 = l1.next
            if l2:
                l2 = l2.next
        
        return dummy.next
```

Мы используем фиктивный узел для упрощения построения списка связанных результатов. Мы перебираем как связанные списки (и), так и несущую |1|2. Для каждой позиции мы добавляем число от и (или 0, если один список короче) плюс Керри. |1|2. Цифра для нового узла - , а для нового переноса -.total % 10total // 10. Мы создаём новый узел с этой цифрой и добавляем его в список результатов. Переходим к следующим узлам в и если они есть.|1|2. Цикл продолжается до тех пор, пока оба списка не будут исчерпаны Керри не останется. Наконец, мы возвращаем, который является началом списка связанных сумм.dummy.next.