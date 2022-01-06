---
title: 链表做加法怎么回事？链表做加法算法怎么写？
author: huhansome
date: 2021-12-22 17:38:00 +0800
categories: [流弊技能]
tags: [算法, 链表反转, 算法与数据结构, 数据结构, 链表]
---
链表做加法？，对的你没看错，链表做加法的算法也是比较经典的算法，实现的效果如下：

a = 7->1->2 <br/>
b = 4->3 <br/>
result = 1->5->2

```
public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
  // 计算结果存储的 dummy 结点
  ListNode dummy = new ListNode(0);
  ListNode p = l1, q = l2, curr = dummy;
  // 进位默认为 0
  int carry = 0;
  // 进入循环，以p和q两个链表指针都走到头为结束
  while (p != null || q != null) {
    int x = (p != null) ? p.val : 0;
    int y = (q != null) ? q.val : 0;
    // 进位参与运算
    int sum = carry + x + y;
    // 计算进位
    carry = sum / 10;
    // 构造新的结点存储计算后的位数数值
    curr.next = new ListNode(sum % 10);
    curr = curr.next;
    if (p != null)
      p = p.next;
    if (q != null)
      q = q.next;
  }
  // 处理数字最高位末尾进位情况
  if (carry > 0) {
    curr.next = new ListNode(carry);
  }
  return dummy.next;
}
```
以上就是链表做加法的算法，可以说这个写法是很标准的写法了。你学废了吗？