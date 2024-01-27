---
{"dg-publish":true,"permalink":"/1-leet-code/leet-code/","noteIcon":"","created":"2024-01-27T08:08:41.943+01:00","updated":"2024-01-27T17:25:38.907+01:00"}
---


| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 28 |   Easy   | https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/description/?company_slug=bytedance |

```markdown
Given two strings needle and haystack, return the index of the first occurrence of needle in haystack, or -1 if needle is not part of haystack.

Example 1:
Input: haystack = "sadbutsad", needle = "sad"
Output: 0
Explanation: "sad" occurs at index 0 and 6.
The first occurrence is at index 0, so we return 0.

Example 2:
Input: haystack = "leetcode", needle = "leeto"
Output: -1
Explanation: "leeto" did not occur in "leetcode", so we return -1.
```

> Solution
```c
int strStr(char* haystack, char* needle) {
    int m = strlen(haystack);
    int n = strlen(needle);

    for (int i = 0; i <= m - n; ++i) {
        int k = 0;
        for (int j = i; k < n && haystack[j] == needle[k]; ++j, ++k);
        if (k == n) return i;
    }

    return -1;
}
```
---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 21 |   Easy   | https://leetcode.cn/problems/merge-two-sorted-lists/description/ |

```markdown
You are given the heads of two sorted linked lists list1 and list2.

Merge the two lists into one sorted list. The list should be made by splicing together the nodes of the first two lists.

Return the head of the merged linked list.

Example 1:
Input: list1 = [1,2,4], list2 = [1,3,4]
Output: [1,1,2,3,4,4]

Example 2:
Input: list1 = [], list2 = []
Output: []

Example 3:
Input: list1 = [], list2 = [0]
Output: [0]
```

> Solution
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* mergeTwoLists(struct ListNode* list1, struct ListNode* list2) {
    struct ListNode* list = malloc(sizeof(struct ListNode));
    struct ListNode* sentinel = list;
    while (list1 != NULL && list2 != NULL) {
        if (list1->val < list2->val) {
            sentinel->next = list1;
            list1 = list1->next;
        } else {
            sentinel->next = list2;
            list2 = list2->next;
        }
        sentinel = sentinel->next;
    }

    /*Add remaining nodes to the list if either of the list is NULL*/
    if (list1 == NULL) {
        sentinel->next = list2;
    }

    if (list2 == NULL) {
        sentinel->next = list1;
    }

    return list->next;
}
```
---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 88 |   Easy   | https://leetcode.cn/problems/merge-sorted-array/description/ |
```markdown
You are given two integer arrays nums1 and nums2, sorted in non-decreasing order, and two integers m and n, representing the number of elements in nums1 and nums2 respectively.

Merge nums1 and nums2 into a single array sorted in non-decreasing order.

The final sorted array should not be returned by the function, but instead be stored inside the array nums1. To accommodate this, nums1 has a length of m + n, where the first m elements denote the elements that should be merged, and the last n elements are set to 0 and should be ignored. nums2 has a length of n.

Example 1:

Input: nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
Output: [1,2,2,3,5,6]
Explanation: The arrays we are merging are [1,2,3] and [2,5,6].
The result of the merge is [1,2,2,3,5,6] with the underlined elements coming from nums1.
Example 2:

Input: nums1 = [1], m = 1, nums2 = [], n = 0
Output: [1]
Explanation: The arrays we are merging are [1] and [].
The result of the merge is [1].
Example 3:

Input: nums1 = [0], m = 0, nums2 = [1], n = 1
Output: [1]
Explanation: The arrays we are merging are [] and [1].
The result of the merge is [1].
Note that because m = 0, there are no elements in nums1. The 0 is only there to ensure the merge result can fit in nums1.
```
> Solution
```c
void merge(int* nums1, int nums1Size, int m, int* nums2, int nums2Size, int n) {
    int i = 0, j = 0;
    /* j < n as n nums2 can be empty */
    for (; i < nums1Size && j < n; ) {
        if (nums1[i] < nums2[j]) {
            i++;
        } else {
            /* Right shift elements [i, m - 1] in nums1 by one index as nums1[i] >= nums2[j] */
            for (int k = m - 1; k >= i; --k) {
                nums1[k + 1] = nums1[k];
            }
            /* Right-most element has been updated in the above shift*/
            m += 1;
            /* Fill in the smaller element from nums2 in nums1 */
            nums1[i] = nums2[j];
            j++;
        }
    }
    if (j < n) {
        /* Copy remaining elements from nums2 in nums1 */
        for (int i = nums1Size - (n - j); i < nums1Size;) {
            nums1[i++] = nums2[j++];
        }
    }
}
```

```c
void merge(int* nums1, int nums1Size, int m, int* nums2, int nums2Size, int n) {
    /* Append the larger number to the nums1 */
    int nums1Idx = m - 1;
    int nums2Idx = n - 1;
    int nums1TailIdx = nums1Size - 1;
    int nextItem;

    while (nums1Idx >= 0 || nums2Idx >= 0) {
        if (nums1Idx < 0) {
            nextItem = nums2[nums2Idx--];
        } else if (nums2Idx < 0) {
            nextItem = nums1[nums1Idx--];
        } else if (nums1[nums1Idx] > nums2[nums2Idx]) {
            nextItem = nums1[nums1Idx--];
        } else {
            nextItem = nums2[nums2Idx--];
        }

        nums1[nums1TailIdx--] = nextItem;
    }
}
```
---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 977 |   Easy   | https://leetcode.cn/problems/squares-of-a-sorted-array/description/ |
```markdown
Given an integer array nums sorted in non-decreasing order, return an array of the squares of each number sorted in non-decreasing order.

Example 1:

Input: nums = [-4,-1,0,3,10]
Output: [0,1,9,16,100]
Explanation: After squaring, the array becomes [16,1,0,9,100].
After sorting, it becomes [0,1,9,16,100].
Example 2:

Input: nums = [-7,-3,2,3,11]
Output: [4,9,9,49,121]

Constraints:

1 <= nums.length <= 104
-104 <= nums[i] <= 104
nums is sorted in non-decreasing order.
```
> Solution
```c
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
int* sortedSquares(int* nums, int numsSize, int* returnSize) {
    int *returnNums = malloc(sizeof(int) * numsSize);
    *returnSize = numsSize;
    for (int i = 0, j = numsSize - 1, idx = j; i <= j; idx--) {
        int left = nums[i] * nums[i];
        int right = nums[j] * nums[j];
        if (left > right) {
            returnNums[idx] = left;
            i++;
        } else {
            returnNums[idx] = right;
            j--;
        }
    }
    return returnNums;
}
```
- Optimization (no need to compare two large squared values)
```c
/**
 * Note: The returned array must be malloced, assume caller calls free().
 */
int* sortedSquares(int* nums, int numsSize, int* returnSize) {
    int *returnNums = malloc(sizeof(int) * numsSize);
    *returnSize = numsSize;
    for (int i = 0, j = numsSize - 1, idx = j; i <= j; idx--) {
        int left = nums[i], right = nums[j];
        /* left is a negative number with larger absolute value */
        if (left + right < 0) {
            returnNums[idx] = left * left;
            i++;
        } else {
            returnNums[idx] = right * right;
            j--;
        }
    }
    return returnNums;
}
```
---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 83 |   Easy   | https://leetcode.cn/problems/remove-duplicates-from-sorted-list/description/ |
```markdown
Given the head of a sorted linked list, delete all duplicates such that each element appears only once. Return the linked list sorted as well.

Example 1:

Input: head = [1,1,2]
Output: [1,2]
Example 2:

Input: head = [1,1,2,3,3]
Output: [1,2,3]

Constraints:

The number of nodes in the list is in the range [0, 300].
-100 <= Node.val <= 100
The list is guaranteed to be sorted in ascending order.
```

> Solution
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* deleteDuplicates(struct ListNode* head) {
    if (head == NULL) {
        return NULL;
    }

    struct ListNode* sentinel = malloc(sizeof(struct ListNode));
    struct ListNode* list = sentinel;
    while (head->next != NULL) {
        if (head->val != head->next->val) {
            list->next = head;
            list = list->next;
        }

        head = head->next;
    }

    /* Append the last element */
    list->next = head;

    return sentinel->next;
}
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* deleteDuplicates(struct ListNode* head) {
    if (head == NULL) {
        return NULL;
    }

    struct ListNode* sentinel = malloc(sizeof(struct ListNode));
    sentinel->val = 0;
    sentinel->next = head;
    struct ListNode* slow = sentinel->next, *fast = sentinel->next;
    while (fast != NULL) {
        if (fast->val != slow->val) {
            slow->next = fast;
            slow = slow->next;
        }
        fast = fast->next;
    }
    /* Disconnect all nodes after slow as they are duplicates */
    slow->next = NULL;
    return sentinel->next;
}
```

---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 141 |   Easy   | https://leetcode.cn/problems/linked-list-cycle/description/ |
```markdown
Given head, the head of a linked list, determine if the linked list has a cycle in it.

There is a cycle in a linked list if there is some node in the list that can be reached again by continuously following the next pointer. Internally, pos is used to denote the index of the node that tail's next pointer is connected to. Note that pos is not passed as a parameter.

Return true if there is a cycle in the linked list. Otherwise, return false.

Example 1:
Input: head = [3,2,0,-4], pos = 1
Output: true
Explanation: There is a cycle in the linked list, where the tail connects to the 1st node (0-indexed).

Example 2:
Input: head = [1,2], pos = 0
Output: true
Explanation: There is a cycle in the linked list, where the tail connects to the 0th node.

Example 3:
Input: head = [1], pos = -1
Output: false
Explanation: There is no cycle in the linked list.

Constraints:
The number of the nodes in the list is in the range [0, 104].
-105 <= Node.val <= 105
pos is -1 or a valid index in the linked-list.
```
> Solution
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
bool hasCycle(struct ListNode *head) {
    if (head == NULL) {
        return false;
    }
 
    struct ListNode *slow = head, *fast = head->next;
    while (fast != NULL && fast->next != NULL && fast != slow) {
        slow = slow->next;
        fast = fast->next->next;
    }

    if (fast != slow) {
        return false;
    }

    return true;
}
```
- fast pointer is `1` node faster than slow pointer because this is the only way that two pointers can reach the same node


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 160 |   Easy   | https://leetcode.cn/problems/intersection-of-two-linked-lists/description/ |
```markdown
Given the heads of two singly linked-lists headA and headB, return the node at which the two lists intersect. 
If the two linked lists have no intersection at all, return null.

Examples: see the LeetCode
```

> Intereseting Solution with HashTable in C :)
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct HashTable {
    struct ListNode *key;
    UT_hash_handle hh;
};

struct ListNode *getIntersectionNode(struct ListNode *headA, struct ListNode *headB) {
    struct HashTable *hashTable = NULL;
    struct ListNode *temp = headA;
    while (temp != NULL) {
        struct HashTable *tmp;
        HASH_FIND(hh, hashTable, &temp, sizeof(struct HashTable *), tmp);
        if (tmp == NULL) {
            tmp = malloc(sizeof(struct HashTable));
            tmp->key = temp;
            HASH_ADD(hh, hashTable, key, sizeof(struct HashTable *), tmp);
        }
        temp = temp->next;
    }
    temp = headB;
    while (temp != NULL) {
        struct HashTable *tmp;
        HASH_FIND(hh, hashTable, &temp, sizeof(struct HashTable *), tmp);
        if (tmp != NULL) {
            return temp;
        }
        temp = temp->next;
    }
    return NULL;
}
```

> My Solution (naive...)
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }

        HashMap<ListNode, Integer> nodeA = new HashMap<>();
        while (headA != null) {
            nodeA.put(headA, 1);
            headA = headA.next;
        }

        while (headB != null) {
            if (nodeA.containsKey(headB)) {
                return headB;
            }
            headB = headB.next;
        }

        return null;
    }
}
```

> LeetCode Official Solution (interesting)
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) {
 *         val = x;
 *         next = null;
 *     }
 * }
 */
public class Solution {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode pNode1 = headA, pNode2 = headB;
        while (pNode1 != pNode2) {
            if (pNode1 == null) {
                pNode1 = headB;
            } else {
                pNode1 = pNode1.next;
            }

            if (pNode2 == null) {
                pNode2 = headA;
            } else {
                pNode2 = pNode2.next;
            }
        }

        return pNode1;
    }
}
```
- two cases:
    - no intersection and two pointers end up with pointing to NULL
    - has intersection and two pointers end up with pointing to the first same object
        - iterates m + n + c0 nodes, where c0 is the nodes before the intersection node


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 203 |   Easy   | https://leetcode.cn/problems/remove-linked-list-elements/description/ |
```markdown
Given the head of a linked list and an integer val, remove all the nodes of the linked list that has Node.val == val, and return the new head.

Example 1:
Input: head = [1,2,6,3,4,5,6], val = 6
Output: [1,2,3,4,5]

Example 2:
Input: head = [], val = 1
Output: []

Example 3:
Input: head = [7,7,7,7], val = 7
Output: []

Constraints:
The number of nodes in the list is in the range [0, 104].
1 <= Node.val <= 50
0 <= val <= 50
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* removeElements(struct ListNode* head, int val) {
    if (head == NULL) {
        return NULL;
    }

    struct ListNode* prev = NULL;
    struct ListNode* curr = head;
    struct ListNode* next = NULL;
    while (curr != NULL) {
        /* Points next to the child node of currrent */
        next = curr->next;

        if (curr->val == val) {
            if (prev == NULL) {
                /* Head Node */
                curr = curr->next;
                head = head->next;
                if (curr != NULL) next = curr->next;
                continue;
            } else if (next == NULL) {
                /* Tail Node */
                prev->next = NULL;
                break;
            } else {
                prev->next = next;
                curr = curr->next;
            }
        } else {
            prev = curr;
            curr = curr->next;
        }
    }

    return head;
}
```

> LeetCode Official Solution
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* removeElements(struct ListNode* head, int val) {
    if (head == NULL) {
        return NULL;
    }

    head->next = removeElements(head->next, val);
    return head->val == val ? head->next : head;
}
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* removeElements(struct ListNode* head, int val) {
    struct ListNode* sentinel = malloc(sizeof(struct ListNode));
    sentinel->next = head;
    struct ListNode* curr = sentinel;
    while (curr->next != NULL) {
        if (curr->next->val == val) {
            curr->next = curr->next->next;
        } else {
            curr = curr->next;
        }
    }

    return sentinel->next;
}
```
> **trick: add additional sentinel node pointing to the head to include head in the check**


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 206 |   Easy   | https://leetcode.cn/problems/reverse-linked-list/ |
```markdown
Given the head of a singly linked list, reverse the list, and return the reversed list.
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* reverseList(struct ListNode* head) {
    struct ListNode* prev = NULL;
    struct ListNode* curr = head;
    // struct ListNode* next = NULL;

    while (curr != NULL) {
        /* Save a copy of the next node for next iteration */
        struct ListNode* node = curr->next;
        curr->next = prev;

        /* Return if the next node is NULL - reach the tail */
        if (node == NULL) break;
        prev = curr;
        curr = node;
    }

    return curr;
}

```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* helpFunc(struct ListNode* curr, struct ListNode* prev) {
    if (curr == NULL) {
        return prev;
    }

    struct ListNode* next = curr->next;
    curr->next = prev;
    return helpFunc(next, curr);
}

struct ListNode* reverseList(struct ListNode* head) {
    return helpFunc(head, NULL);
}
```

> LeetCode Official Implementation
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* reverseList(struct ListNode* head) {
    /* head == NULL as for head = NULL */
    if (head == NULL || head->next == NULL) {
        return head;
    }

    struct ListNode* node = reverseList(head->next);
    head->next->next = head; /* Good trick ! */
    head->next = NULL; /* head must point to NULL */
    return node;
}
```
- smart trick: `head->next->next = head`
    - this reverse the `two adjacent nodes`


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 234 |   Easy   | https://leetcode.cn/problems/palindrome-linked-list/description/ |
```markdown
Given the head of a singly linked list, return true if it is a palindrome or false otherwise.
```

> Solution
- `naive`
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
bool isPalindrome(struct ListNode* head) {
    int vals[100000] = {0};
    int cnt = 0;
    /* O(n) */
    while (head != NULL) {
        vals[cnt++] = head->val;
        head = head->next;
    }

    /* O(n) */
    for (int i = 0, j = cnt - 1; i < j; i++, j--) {
        if (vals[i] != vals[j]) {
            return false;
        }
    }

    return true;
}
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* front = NULL;

bool helpFunc(struct ListNode* tail) {
    if (tail == NULL) {
        return true;
    }

    bool res = helpFunc(tail->next);
    res &= (front->val == tail->val);
    if (res) {
        front = front->next;
    }

    return res;
}


bool isPalindrome(struct ListNode* head) {
    front = head;

    return helpFunc(head);
}
```

> LeetCode official solution
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
static struct ListNode* front = NULL;

bool helpFunc(struct ListNode* tail) {
    if (tail != NULL) {
        if (!helpFunc(tail->next)) {
            return false;
        }

        if (front->val != tail->val) {
            return false;
        }

        front = front->next;
    }

    return true;
}


bool isPalindrome(struct ListNode* head) {
    front = head;
    return helpFunc(head);
}
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* reverse(struct ListNode* head) {
    struct ListNode* prev = NULL;
    struct ListNode* curr = head;

    while (curr != NULL) {
        /* Save a copy of the next node for next iteration */
        struct ListNode* node = curr->next;
        curr->next = prev;

        /* Return if the next node is NULL - reach the tail */
        if (node == NULL) break;
        prev = curr;
        curr = node;
    }

    return curr;
}


bool isPalindrome(struct ListNode* head) {
    /* Find the numbers of nodes in the given list */
    int cnt = 0;
    struct ListNode* tmp = head;
    for (; tmp != NULL; cnt++, tmp = tmp->next);

    /* Single node always return true */
    if (cnt == 1) {
        return true;
    }

    /* Find the first node of the right-half list */
    int half = (cnt >> 1) - 1;
    struct ListNode* right = head;
    for(int i = 0; i <= half; i++, right = right->next);

    /* Reverse the right-half list */
    struct ListNode* tail = reverse(right);

    /* Save the pointer to the tail to reverse it back later */
    struct ListNode* tailCpy = tail;

    /* Check whether it is a palindrome by definition */
    bool result = true;
    while (result && tail != NULL) {
        if (head->val != tail->val) {
            /* Save the result to break the loop, this ensures the reversed list is always recovered */
            result = false;
        }

        head = head->next;
        tail = tail->next;
    }

    /* Reverse the right-half nodes */
    right->next = reverse(tailCpy);

    return result;
}
```

---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 876 |   Easy   | https://leetcode.cn/problems/middle-of-the-linked-list/description/ |
```markdown
Given the head of a singly linked list, return the middle node of the linked list.
If there are two middle nodes, return the second middle node.
```

> Solution
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* middleNode(struct ListNode* head) {
    /* Count number of nodes */
    int cnt;
    struct ListNode *pNode = head;
    for (cnt = 0; pNode != NULL; cnt++, pNode = pNode->next);

    int middle = cnt >> 1;
    for (int i =0; i < middle; i++, head = head->next);

    return head;
}
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* middleNode(struct ListNode* head) {
    struct ListNode *slow = head, *fast = head;
    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
    }

    return slow;
}
```
- `fast` and `slow` pointers solution, `fast` moves by two nodes while `slow` moves by one node


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 1290 |   Easy   | https://leetcode.cn/problems/convert-binary-number-in-a-linked-list-to-integer/|
```markdown
Given head which is a reference node to a singly-linked list. The value of each node in the linked list is either 0 or 1. The linked list holds the binary representation of a number.

Return the decimal value of the number in the linked list.

The most significant bit is at the head of the linked list.
```
> Solution
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
int getDecimalValue(struct ListNode* head) {
    int sum = 0;
    while (head != NULL) {
        sum = (sum << 1) + head->val;
        head = head->next;
    }

    return sum;
}
```

**interesting**
```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
int getDecimalValue(struct ListNode* head) {
    int sum = 0;
    while (head != NULL) {
        sum = (sum << 1) | head->val;
        head = head->next;
    }

    return sum;
}
```
- replacing the add operation with bit or results in longer runtime
    - shouldn't bit or operation be much more efficient?
    - [TODO] check the diassambly code ...


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 2 |   Medium   | https://leetcode.cn/problems/add-two-numbers/description/ |
```markdown
You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order, and each of their nodes contains a single digit. Add the two numbers and return the sum as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    int carry = 0;
    struct ListNode *sentinel = malloc(sizeof(struct ListNode)), *res = sentinel;
    while (l1 != NULL && l2 != NULL) {
        int val1 = l1->val;
        int val2 = l2->val;
        struct ListNode *newNode = malloc(sizeof(struct ListNode));
        newNode->val = (val1 + val2 + carry) % 10;
        newNode->next = NULL;
        sentinel->next = newNode;
        carry = (val1 + val2 + carry) / 10;

        l1 = l1->next;
        l2 = l2->next;
        sentinel = sentinel->next;
    }

    struct ListNode *rest = (l1 == NULL && l2 == NULL) ? NULL : l1 == NULL ? l2 : l1;
    while (rest != NULL) {
        int val = rest->val;
        struct ListNode *newNode = malloc(sizeof(struct ListNode));
        newNode->val = (val + carry) % 10;
        newNode->next = NULL;
        carry = (val + carry) / 10;
        sentinel->next = newNode;

        rest = rest->next;
        sentinel = sentinel->next;
    }

    if (carry > 0) {
        struct ListNode *newNode = malloc(sizeof(struct ListNode));
        newNode->val = carry;
        newNode->next = NULL;
        sentinel->next = newNode;
    }

    return res->next;
}

```




```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* removeNthFromEnd(struct ListNode* head, int n) {
    struct ListNode *pNode = head;
    int size;
    for (size = 0; pNode; size++, pNode = pNode->next);

    if (n == size) {
        struct ListNode *tmp = head->next;
        free(head);
        head = tmp;
        return head;
    }

    struct ListNode *prev = NULL, *curr = head, *res = head;
    int cnt = 0;
    while (curr != NULL) {
        if (cnt == size - n) {
            prev->next = curr->next;
            free(curr);
            return res;
        }

        prev = curr;
        curr = curr->next;
        cnt++;
    }

    return NULL;
}

```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* removeNthFromEnd(struct ListNode* head, int n) {
    struct ListNode *pNode = head;
    int size;
    for (size = 0; pNode; size++, pNode = pNode->next);

    struct ListNode *sentinel = malloc(sizeof(struct ListNode));
    sentinel->next = head;
    struct ListNode *prev = sentinel, *curr = head, *res;
    int cnt = 0;
    while (curr != NULL) {
        if (cnt == size - n) {
            prev->next = curr->next;
            free(curr);
            goto ans;
        }

        prev = curr;
        curr = curr->next;
        cnt++;
    }

ans:
    res = sentinel->next;
    free(sentinel);
    return res;

    return NULL;
}
```

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode sentinel = new ListNode(0, head);
        Deque<ListNode> nodes = new LinkedList<>();
        ListNode pNode = sentinel;
        for(; pNode != null; nodes.push(pNode), pNode = pNode.next);
        for (int i = 1; i <= n; i++, nodes.pop());
        ListNode prev = nodes.peek();
        prev.next = prev.next.next;
        return sentinel.next;
    }
}
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* removeNthFromEnd(struct ListNode* head, int n) {
    struct ListNode *sentinel = malloc(sizeof(struct ListNode));
    sentinel->val = 0;
    sentinel->next = head;
    struct ListNode *first = head;
    for (int i = 0; i < n; ++i, first = first->next);
    struct ListNode *second = sentinel;
    while (first != NULL) {
        first = first->next;
        second = second->next;
    }

    struct ListNode *tmp = second->next;
    second->next = second->next->next;
    free(tmp);

    return sentinel->next;
}
```

```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* swapPairs(struct ListNode* head) {
    if (head == NULL) {
        return NULL;
    }

    struct ListNode *sentinel = malloc(sizeof(struct ListNode));
    sentinel->next = head;
    sentinel->val = 0;

    struct ListNode *prev = sentinel, *curr = head, *next = NULL;
    /* No need to check the last node */
    while (curr != NULL && curr->next != NULL) {
        next = curr->next;
        /* Swap the adjacent nodes */
        curr->next = next->next;
        prev->next = next;
        next->next = curr;

        /* Prepare for the next iteration */
        prev = curr;
        curr = curr->next;
    }

    return sentinel->next;
}
```


```c
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     struct ListNode *next;
 * };
 */
struct ListNode* rotateRight(struct ListNode* head, int k) {
    if (head == NULL) {
        return head;
    }

    if (k > 0) {
        /* Get the number of nodes */
        struct ListNode* pNode = head;
        int cnt;
        for (cnt = 0; pNode; cnt++, pNode = pNode->next);

        /* Get the actual places need to shift */
        int s = k % cnt;
        /* Namely, k == cnt */
        if (s == 0) {
            return head;
        }

        /* Get the previous node before the first moved node */
        int pos = cnt - s - 1;
        struct ListNode* prev = head;
        for (int i = 0; i < pos; i++, prev = prev->next);
        struct ListNode* newHead = prev->next;
        prev->next = NULL;

        struct ListNode* tmp = newHead;
        for (; tmp->next; tmp = tmp->next);
        tmp->next = head;

        return newHead;
    }

    return head;
}
```



---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 496 |   Easy   | https://leetcode.cn/problems/next-greater-element-i/ |

> Naive solution
```java
class Solution {
    private int findNextGreaterElement(int[] nums, int t) {
        /* First find the index of the targer */
        int idx = 0, n = nums.length;
        for (; idx < n; ++idx) {
            if (nums[idx] == t) {
                break;
            }
        }

        for (int i = idx + 1; i < n; ++i) {
            if (nums[i] > t) {
                return nums[i];
            }
        }

        return -1;
    }

    public int[] nextGreaterElement(int[] nums1, int[] nums2) {
        int n = nums1.length;
        int res[] = new int[n];
        /* O(n*m) */
        for (int i = 0; i < n; ++i) {
            res[i] = findNextGreaterElement(nums2, nums1[i]);
        }

        return res;
    }
}
```

> Monotonic Stack Solution: O(nums1.length + nums2.length)
```java
class Solution {
    private HashMap<Integer, Integer> findNextGreaterElement(int[] nums) {
        int n = nums.length;
        HashMap<Integer, Integer> res = new HashMap<>();
        Stack<Integer> s = new Stack<>();

        /* O(m) */
        int nextGreaterVal;
        for (int i = n - 1; i >= 0; --i) {
            while(!s.isEmpty() && s.peek() <= nums[i]) {
                s.pop();
            }
            nextGreaterVal = s.isEmpty() ? -1 : s.peek();
            res.put(nums[i], nextGreaterVal);
            s.push(nums[i]);
        }

        return res;
    }


    public int[] nextGreaterElement(int[] nums1, int[] nums2) {
        int n1 = nums1.length;
        int[] res = new int[n1];

        HashMap<Integer, Integer> greatVals = findNextGreaterElement(nums2);

        /* O(n) */
        for (int i = 0; i < n1; ++i) {
            res[i] = greatVals.get(nums1[i]); /* O(1) */
        }

        return res;
    }
}
```

---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 739 |   Medium   | https://leetcode.cn/problems/daily-temperatures/ |
```java
class Solution {
    public int[] dailyTemperatures(int[] t) {
        int n = t.length;
        int[] res = new int[n];

        Stack<Integer> s = new Stack<>();
        for (int i = n - 1; i >= 0; --i) {
            while(!s.isEmpty() && t[s.peek()] <= t[i]) {
                s.pop();
            }
            
            res[i] = s.isEmpty() ? 0 : (s.peek() - i);
            s.push(i);
        }

        return res;
    }
}
```

---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 503 |   Medium   | https://leetcode.cn/problems/next-greater-element-ii/description/ |

> Naive Solution
```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] res = new int[n];
        /* O(n) */
        Arrays.fill(res, -1);
        /* O(n^2) */
        for (int i = 0; i < n; ++i) {
            for (int j = i + 1; j < 2 * n - 1; ++j) {
                if (nums[j % n] > nums[i]) {
                    res[i] = nums[j % n];
                    break;
                }
            }
        }

        return res;
    }
}
```

> Monotonic Stack Solution
```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;        
        /* O(n) */
        int[] numsDouble = new int[(n << 1) - 1];
        System.arraycopy(nums, 0, numsDouble, 0, n);
        System.arraycopy(nums, 0, numsDouble, n, n - 1);

        int[] res = new int[n];
        Stack<Integer> s = new Stack<>();
        for (int i = numsDouble.length - 1; i >= 0; --i) {
            while (!s.isEmpty() && s.peek() <= numsDouble[i]) {
                s.pop();
            }

            if (i < n) {
                res[i] = s.isEmpty() ? -1 : s.peek();
            }

            s.push(numsDouble[i]);
        }

        return res;
    }
}
```

- Improved version
```java
class Solution {
    public int[] nextGreaterElements(int[] nums) {
        int n = nums.length;
        int[] res = new int[n];
        Stack<Integer> s = new Stack<>();

        for (int i = n - 1; i >= -n; --i) {
            int realIdx = Math.floorMod(i, n);
            while (!s.isEmpty() && s.peek() <= nums[realIdx]) {
                s.pop();
            }

            if (i < 0) res[realIdx] = s.isEmpty() ? -1 : s.peek();
            s.push(nums[realIdx]); 
        }

        return res;
    }
}
```

```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode partition(ListNode head, int x) {
        if (head == null) {
            return null;
        }

        ListNode sentinel = new ListNode(0, head);
        ListNode slow = sentinel, fast = sentinel;
        /* fast != null: tail node > x, thus, fast = fast.next leads to null
         * fast.next != null: tail node < x
         */
        while (fast != null && fast.next != null) {
            if (fast.next.val < x) {
                /* first few nodes are smaller than x */
                if (slow.next == fast.next) {
                    slow = slow.next;
                    fast = fast.next;
                    continue;
                }

                /* Disconnect the node and insert it to the front */
                ListNode smallNode = fast.next;
                fast.next = fast.next.next;
                ListNode tmp = slow.next;
                slow.next = smallNode;
                smallNode.next = tmp;
                slow = slow.next;
                /* NOTE: next node of fast can still be smaller than x. fast can't foward here */
            } else {
                fast = fast.next;
            }
        }
        
        return sentinel.next;
    }
}
```

---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 23 |   Hard   | https://leetcode.cn/problems/merge-k-sorted-lists/ |


> Naive Solution
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        int n = lists.length;
        if (n == 0) {
            return null;
        }
        
        ListNode sentinel = new ListNode(-100000, lists[0]);
        for (int i = 1; i < n; ++i) {
            /* first few elements in lists[i] can be small enough */
            ListNode res = sentinel;
            ListNode pNew = lists[i];
            /* Insert nodes from lists[i] into res */
            while (pNew != null && res.next != null) {
                if (pNew.val >= res.val && pNew.val <= res.next.val) {
                    ListNode temp = res.next;
                    res.next = pNew;
                    ListNode next = pNew.next;
                    pNew.next = temp;
                    pNew = next;
                }
                res = res.next;
            }

            /* Simply append all nodes to the res */
            if (res.next == null) {
                while (pNew != null) {
                    res.next = pNew;
                    pNew = pNew.next;
                    res = res.next;
                }
            }
        }

        return sentinel.next;
    }
}
```

> Binary Heap
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        int n = lists.length;
        if (n == 0)
            return null;
        
        ListNode sentinel = new ListNode(-10000);
        ListNode pNode = sentinel;
        PriorityQueue<ListNode> pq = new PriorityQueue<>(
            lists.length, (a, b)->(a.val - b.val));
        for (ListNode head : lists) {
            if (head != null)
                pq.add(head);
        }

        while (!pq.isEmpty()) {
            ListNode node = pq.poll();
            pNode.next = node;
            if (node.next != null) {
                pq.add(node.next);
            }
            pNode = pNode.next;
        }

        return sentinel.next;
    }
}
```


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 26 |   Easy   | https://leetcode.cn/problems/remove-duplicates-from-sorted-array/ |
```java
class Solution {
    public int removeDuplicates(int[] nums) {
        int n = nums.length;
        int slow = 0;
        for (int fast = 1; fast < n; fast++) {
            if (nums[fast] != nums[slow]) {
                slow++;
                /* non-duplicated elements are over-written by itself */
                nums[slow] = nums[fast];
            }
        }

        /* +1 as slow is the index */
        return slow + 1;
    }
}
```


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 27 |   Easy   | https://leetcode.cn/problems/remove-element/description/ |
```java
class Solution {
    public int removeElement(int[] nums, int val) {
        int n = nums.length;
        int slow = 0;
        for (int fast = 0; fast < n; ++fast) {
            if (nums[fast] != val) {
                nums[slow] = nums[fast];
                slow++;
            }
        }

        return slow;
    }
}
```

---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 283 |   Easy   | https://leetcode.cn/problems/move-zeroes/description/ |
```java
class Solution {
    public void moveZeroes(int[] nums) {
        int n = nums.length;
        for (int slow = 0, fast = 0; fast < n; ++fast) {
            if (nums[fast] != 0) {
                /* No need to overwrite at the same index */
                /* Also, nums[fast] = 0 is needed when elements after slow are non-zero */
                if (slow != fast) {
                    nums[slow] = nums[fast];
                    nums[fast] = 0;
                }
                slow++;
            }
        }
    }
}
```
- thinking: `if (slow != fast)` can be used in `removeElement` to avoid overwriting. But assigning the element of an array is much more efficient
than comparing two integers in an if, right?

> Reuse the removeElement
```java
class Solution {
    public void moveZeroes(int[] nums) {
        int n = nums.length;
        int slow = 0;
        for (int fast = 0; fast < n; ++fast) {
            if (nums[fast] != 0) {
                /* [0, slow) */
                nums[slow++] = nums[fast];
            }
        }

        /* Set all tail elements to zero */
        for(; slow < n; nums[slow++] = 0);
    }
}
```


---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 167 |   Medium   | https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted/description/ |
```java
class Solution {
    public int binarySearch(int[] numbers, int left, int target) {
        int right = numbers.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (numbers[mid] < target) {
                left = mid + 1;
            } else if (numbers[mid] > target) {
                right = mid - 1;
            } else {
                return mid;
            }
        }
        return -1;
    }

    public int[] twoSum(int[] numbers, int target) {
        int l = numbers.length;
        for (int i = 0; i < l; ++i) {
            int retIdx = binarySearch(numbers, i + 1, target - numbers[i]);
            if (retIdx > 0) {
                return new int[] {i + 1, retIdx + 1};
            }
        }

        return null;
    }
}
```

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0, right = numbers.length - 1;
        while (left <= right) {
            int sum = numbers[left] + numbers[right];
            if (sum == target) {
                return new int[] {left + 1, right + 1};
            } else if (sum < target) {
                left++;
            } else {
                right--;
            }
        }

        return null;
    }
} 
```
- trick: apply the similar principle of binary search
    - reduce the searching space
    - sorted array hints the usage of `binary search`

---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 344 |   Easy   | https://leetcode.cn/problems/reverse-string/ |
```java
class Solution {
    public void reverseString(char[] s) {
        int left = 0, right = s.length -1;
        while (left < right) {
            char tmp = s[left];
            s[left] = s[right];
            s[right] = tmp;
            left++; right--;
        }
    }
}
```

---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 5 |   Medium   | https://leetcode.cn/problems/longest-palindromic-substring/description/ |
```java
class Solution {
    public String Palindromic(String s, int left, int right) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            left--; right++;
        }

        return s.substring(left + 1, right);
    }

    public String longestPalindrome(String s) {
        String res = "";
        for (int i = 0; i < s.length(); ++i) {
            String s1 = Palindromic(s, i, i);
            String s2 = Palindromic(s, i, i + 1);
            if (s1.length() > res.length()) {
                res = s1;
            }
            if (s2.length() > res.length()) {
                res = s2;
            }
        }

        return res;
    }
}
```
---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 303 |   Easy   | https://leetcode.cn/problems/range-sum-query-immutable/description/ |
> Naïve Solution
```java
class NumArray {
    int[] nums;

    public NumArray(int[] nums) {
        this.nums = nums;
    }
    
    public int sumRange(int left, int right) {
        int sum = 0;
        for (int i = left; i <= right; ++i) {
            sum += nums[i];
        }
        return sum;
    }
}
```
> Prefix Sum
```java
class NumArray {
    int[] preNums;

    public NumArray(int[] nums) {
        preNums = new int[nums.length + 1];
        preNums[0] = 0;
        for (int i = 1; i <= nums.length; ++i) {
            preNums[i] = preNums[i - 1] + nums[i - 1];
        }
    }
    
    public int sumRange(int left, int right) {
        return preNums[right + 1] - preNums[left];
    }
}
```
- `a[i] + a[i+1] + ... + a[j] = sum[j] - sum[i]` 
	- `right + 1` because the `0th` element in `preNums` is `0` which is needed for `i = 0`
---
| Leetcode Question | Level | Link |
| :---------------: | :------: | :----: |
| 304 |   Medium   | https://leetcode.cn/problems/range-sum-query-2d-immutable/ |
> Prefix Sum: rows only
```java
class NumMatrix {
    int[][] preNums;

    public NumMatrix(int[][] matrix) {
        int rows = matrix.length, cols = matrix[0].length;
        preNums = new int[rows][cols + 1];
        /*O(n^2)*/
        for (int i = 0; i < rows; ++i) {
            for (int j = 1; j <= cols; ++j) {
                preNums[i][j] = preNums[i][j - 1] + matrix[i][j - 1];
            }
        }
    }
    
    public int sumRegion(int row1, int col1, int row2, int col2) {
        /*O(n)*/
        int sum = 0;
        for (int i = row1; i <= row2; ++i) {
            sum += (preNums[i][col2 + 1] - preNums[i][col1]);
        }
        return sum;
    }
}
```
> Prefix Sum: sub-matrixes start from (0, 0)
```java
class NumMatrix {
    int[][] preSums;
    public NumMatrix(int[][] matrix) {
        int row = matrix.length, col = matrix[0].length;
        preSums = new int[row + 1][col + 1];
        for (int i = 1; i <= row; ++i) {
            for (int j = 1; j <= col; ++j) {
	            /*(0, 0, i, j) = (0, 0, i - 1, j) + (0, 0, i, j - 1) + a[i][j] - (0, 0, i-1, j-1)*/
                preSums[i][j] = preSums[i - 1][j] + preSums[i][j - 1] + matrix[i - 1][j - 1] - preSums[i - 1][j - 1];
            }
        }
    }
    
    public int sumRegion(int row1, int col1, int row2, int col2) {
	    /*sub-matrix starts from [0][0]: (0, 0, row2, col2) - (0, 0, row2, col1) - (0, 0, row1, col2) + (0, 0, row1, col1)*/
        return preSums[row2 + 1][col2 + 1] - preSums[row2 + 1][col1] - preSums[row1][col2 + 1] + preSums[row1][col1];
    }
}
```