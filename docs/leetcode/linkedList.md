
### leetcode25

>简单说下思路，可以用栈，弹出node的时候顺序就是翻转了

>或者是我最开始的思路，递归，当前指针下移，链表右边翻转（也就是快慢指针）

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} k
 * @return {ListNode}
 */
var reverseKGroup = function(head, k) {
    if(!head) return head
   if(k === 1) return head

  let l = null
  let r = null
  let p = head
  let _k = k
  let arr = []
  
  while(p !== null) {
    l = null
    let z = JSON.parse(JSON.stringify(p))
    while(p && _k--) {
      r = p.next
      p.next = l
      l = p
      p = r
    }
    if(_k > 0) {
      arr.push(z)
    } else {
      arr.push(l)
    }
    _k = k
  }
  let node = arr[0]
  for(let i = 0; i < arr.length; i++) {
    while(arr[i].next) arr[i] = arr[i].next
    arr[i].next = arr[i + 1]
  }
  return node
};
```

别人的优秀代码

```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} k
 * @return {ListNode}
 */
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} head
 * @param {number} k
 * @return {ListNode}
 */
var reverseKGroup = function(head, k) {
    let newHead = new ListNode();
    newHead.next = head;
    let prev = null;
    curr = newHead;

    let count = 0;
    let temp = [];
    while (curr) {
        prev = curr;
        curr = curr.next;
        
        let innerPoint = curr;
        while (temp.length < k && innerPoint) {
            temp.push(innerPoint);
            innerPoint = innerPoint.next;
        }

        if (temp.length < k) {
            break;
        } else {
            let len = temp.length
            const lastNext = temp[temp.length - 1].next;
            while (len) {
                const node = temp.pop();
                prev.next = node;
                prev = node;
                len -= 1;
            }
            temp = [];
            curr.next = lastNext;
        }
    }

    return newHead.next;
};
```