#Код #Python 
```` Python
class Solution:
    def mergeTwoLists(self, list1: Optional[ListNode], list2: Optional[ListNode]) -> Optional[ListNode]:
        dummy = ListNode()
        cur = dummy

        while list1 and list2:
            if list1.val > list2.val:
                cur.next = list2
                list2 = list2.next
            else:
                cur.next = list1
                list1 = list1.next
            
            cur = cur.next
        
        if list1:
            cur.next = list1
        else:
            cur.next = list2
        
        return dummy.next
````


Изучил как работают ООП на сайте metanit, код стал немного понятнее, в целом я понимаю что происходит, но с нуля такое писать ещё мне сложно.

Сначала  мы задаём класс, а в нём функцию, где задаем пустой лист и присваиваем его в переменную. 

Далее задаём цикл while, который работает пока оба списка не пусты, как только один из них закончится цикл завершится. Далее сравниваем узлы list1 и list2, меньший из этих узлов присваиваем cur.next - конечному списку. Далее list1 или list2 сдвигается на следующий узел, например если list1 = 5, а list 2 = 1. то list2 отдаёт своё одно значение и переходит на следующий узел. Далее cur присваиваем конечный список. Далее выходим из цикла, один из списков (list1 или list2 пуст), а один из них ещё содержит элементы, если он не закончился, то все оставшиеся узлы добавляются в него. Далее возвращаем с помощью return начало итогового списка.