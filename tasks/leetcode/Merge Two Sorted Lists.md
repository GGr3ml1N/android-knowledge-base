### Задача
You are given the heads of two sorted linked lists `list1` and `list2`.

Merge the two lists into one **sorted** list. The list should be made by splicing together the nodes of the first two lists.

Return _the head of the merged linked list_.

### Перевод
Вам даны головы двух отсортированных связных списков — `list1` и `list2`.

Объедините два списка в один **отсортированный** список. Список нужно сформировать путём соединения узлов из первых двух списков.

Верните _голову объединённого связного списка_.

### Ответ
```kotlin
fun mergeTwoLists(list1: ListNode?, list2: ListNode?): ListNode? {  
    var temp1: ListNode? = list1  
    var temp2: ListNode? = list2  
  
    var pointer: ListNode? = ListNode()  
    val resultNode: ListNode? = pointer  
  
    while (temp1?.`val` != null && temp2?.`val` != null) {  
        when {  
            temp1.`val` <= temp2.`val` -> {  
                pointer?.next = ListNode(temp1.`val`)  
                pointer = pointer?.next  
                temp1 = temp1.next  
            }  
  
            temp1.`val` > temp2.`val` -> {  
                pointer?.next = ListNode(temp2.`val`)  
                pointer = pointer?.next  
                temp2 = temp2.next  
            }  
        }  
    }  
  
    pointer?.next = temp1 ?: temp2  
  
    return resultNode?.next  
}

data class ListNode(var `val`: Int = 0, var next: ListNode? = null)
```