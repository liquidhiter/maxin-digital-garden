---
{"dg-publish":true,"permalink":"/3-online-courses/0-cs-61-b/lectures/cs-61-b-2018-spring-learning-notes-chapter-10/","noteIcon":"","created":"2024-03-19T21:08:46.154+01:00","updated":"2024-03-19T22:02:51.503+01:00"}
---

## Slides
<iframe src="https://docs.google.com/presentation/d/1rEHpAx8Xu2LnJBWsRPWy8blL20qb96Q5UhdZtQYFkBI/edit#slide=id.g8dd5cb732_01435" width="700" height="1000" ></iframe>
## Topic: Binary Search Tree (BST)

### Introduction
- array based map
### BST
- binary tree
	- each node has either 0, 1 or 2 children
- BST properties
	- <font color="red">every key in the left subtree is less than X's key</font>
	- <font color="red">every key in the right subtree is greater than X's key</font>
![Z - assets/images/Pasted image 20240319211432.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240319211432.png)
- note the difference between
	- binary tree
	- binary search tree: BST properties
- comparison (ordering) rules
```markdown
Ordering must be complete, transitive, and antisymmetric. Given keys p and q:
Exactly one of p ≺ q and q ≺ p are true.
p ≺ q and q ≺ r imply p ≺ r.
```
- widely used
	- *much (perhaps most?) computation is dedicated towards finding things in response to queries*
- how to insert a new key into a BST
![Z - assets/images/Pasted image 20240319212604.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240319212604.png)
- how to delete a key from a BST
	- key has no children
		- no nothing
	- key has one child
		- replace the key with the child (move up to the upper layer)
	- key has two children (most complicated?)
		- compare the two nodes and decide which one is gona to replace and the other one stays in the same level
	- key is the root ?
		- hibbard deletion
![Z - assets/images/Pasted image 20240319213553.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240319213553.png)
- either the predecessor or successor
	- <font color="red">must follow the BST properties (left < root < right)</font>
		- predecessor: **largest in the left sub-tree**
			- between the `bag` and `dog`
		- successor: **smallest in the right sub-tree**
			- between the `flat` and `dog`
- another example: ![Z - assets/images/Pasted image 20240319214234.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240319214234.png)
- bushy and spindly trees
	- worst case
![Z - assets/images/Pasted image 20240319214557.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240319214557.png)

- mathematical analysis![Z - assets/images/Pasted image 20240319220245.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240319220245.png)
- summary![Z - assets/images/Pasted image 20240319215706.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240319215706.png)