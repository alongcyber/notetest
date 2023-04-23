# leetcode
## 基本算法

## 数据结构
### 链表
单链表的基本技巧
```
1. 合并两个有序链表

2. 链表的分解

3. 合并 k 个有序链表

4. 寻找单链表的倒数第 k 个节点

5. 寻找单链表的中点

6. 判断单链表是否包含环并找出环起点
 
7. 判断两个单链表是否相交并找出交点
```
#### 合并两个有序链表
![偷的图](https://labuladong.github.io/algo/images/%e9%93%be%e8%a1%a8%e6%8a%80%e5%b7%a7/title.jpg)

首先，我们设定一个哨兵节点 prehead ，这可以在最后让我们比较容易地返回合并后的链表。我们维护一个 prev 指针，我们需要做的是调整它的 next 指针。然后，我们重复以下过程，直到 l1 或者 l2 指向了 null ：如果 l1 当前节点的值小于等于 l2 ，我们就把 l1 当前的节点接在 prev 节点的后面同时将 l1 指针往后移一位。否则，我们对 l2 做同样的操作。不管我们将哪一个元素接在了后面，我们都需要把 prev 向后移一位。

在循环终止的时候， l1 和 l2 至多有一个是非空的。由于输入的两个链表都是有序的，所以不管哪个链表是非空的，它包含的所有元素都比前面已经合并链表中的所有元素都要大。这意味着我们只需要简单地将非空链表接在合并链表的后面，并返回合并链表即可。(这段是从leetcode复制的)
``` cpp
 /**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* list1, ListNode* list2) {
               ListNode* prehead = new ListNode(-1),*pre = prehead;
               while(list1&&list2){
                   if(list1->val < list2->val){
                       prehead->next = list1;
                       list1 = list1->next;
                       prehead = prehead->next;
                   }else{
                       prehead->next = list2;
                       list2 = list2->next;
                       prehead = prehead->next;
                   }
               }
                prehead->next = list1==nullptr?list2:list1;
                return pre->next;
    }
};

```

#### 单链表的分解
``` cpp
ListNode* partition(ListNode* head, int x) {
    // 存放小于 x 的链表的虚拟头结点
    ListNode* dummy1 = new ListNode(-1);
    // 存放大于等于 x 的链表的虚拟头结点
    ListNode* dummy2 = new ListNode(-1);
    // p1, p2 指针负责生成结果链表
    ListNode* p1 = dummy1, * p2 = dummy2;
    // p 负责遍历原链表，类似合并两个有序链表的逻辑
    // 这里是将一个链表分解成两个链表
    ListNode* p = head;
    while (p != nullptr) {
        if (p->val >= x) {
            p2->next = p;
            p2 = p2->next;
        } else {
            p1->next = p;
            p1 = p1->next;
        }
        // 断开原链表中的每个节点的 next 指针
        ListNode* temp = p->next;
        p->next = nullptr;
        p = temp;
    }
    // 连接两个链表
    p1->next = dummy2->next;

    return dummy1->next;
}
```