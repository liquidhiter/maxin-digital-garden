---
{"dg-publish":true,"permalink":"/3-online-courses/0-cs-61-b/lectures/cs-61-b-2018-spring-learning-notes-chapter-10/","noteIcon":"","created":"2024-03-19T21:08:46.154+01:00","updated":"2024-03-23T17:15:55.191+01:00"}
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
## Topic: Balanced Search Trees
### 2-3-4 and 2-3 Trees (a.k.a. **B-Trees**)
#### Tree Rotation
![Z - assets/images/Pasted image 20240323110404.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323110404.png)
- rotate left
	- promote its right child as the new parent
![Z - assets/images/Pasted image 20240323110505.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323110505.png)
- rotate right
	- promote its left child as the new parent
- Highlight
	- #balance_method rotation allows balancing of any binary search tree
	- rotate after each insertion and deletion to maintain balance

#### B-trees
- different types of search trees![Z - assets/images/Pasted image 20240323111025.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323111025.png)
- over-stuffed leaf nodes ![Z - assets/images/Pasted image 20240323111815.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323111815.png)
	- problems
		- leaf nodes might contain too many elements
			- O(N) when looking up certain element X
- node splitting ![Z - assets/images/Pasted image 20240323112101.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323112101.png)
	- leaf node is limited to contain certain number of nodes
		- an element is given to the parent when leaf node contains more than allowed
		- split the leaf nodes at the same time
	- #question real use case
		- let's say the right leaf node of parent `O` contains `3` elements
			- insert a new element `s`
				- it should go into `p q r`, which reaches the limitation
				- give the left middle element (leaf nodes need sort?) to its parent to accommodate the new element
				- `p r s` still reaches the limitation
					- split the leaf nodes into two partitions
						- left most element is split into a separate node (smallest?)
						- the rest remains in the same leaf node
- BTS properties are retained ![Z - assets/images/Pasted image 20240323114136.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323114136.png)
	- `r > o, so comapre vs. q`
		- what if the new element is smaller than `q`
			- that's to say, it needs to go into the middle, namely comparing with the `p` ?
- chain reaction ![Z - assets/images/Pasted image 20240323115417.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323115417.png)
- split root node![Z - assets/images/Pasted image 20240323115748.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323115748.png)
	- #question what if splitting into 3 child nodes ?
		- `q->m, q->u, q->w`
		- [ ] reduce the times of splitting a node, assuming splitting a node consumes considerable resources (mostly the time)
- balance the tree ![Z - assets/images/Pasted image 20240323140149.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323140149.png)
- application ![Z - assets/images/Pasted image 20240323140508.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323140508.png)
- dis-advantages of B-Trees![Z - assets/images/Pasted image 20240323141539.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323141539.png)
- Red-black tree![Z - assets/images/Pasted image 20240323141601.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323141601.png)
- Left-leaning Red Black Binary Search Tree (LLRB) ![Z - assets/images/Pasted image 20240323142127.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323142127.png)
- summary ![Z - assets/images/Pasted image 20240323144154.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240323144154.png)
## Implementation of BST in C++
```c++
struct TreeNode {
    int key;
    TreeNode* left;
    TreeNode* right;
    int size; /* number of nodes in the subtree */
    int count; /* number of duplicated nodes, stored as the same node */

    TreeNode(int value)
        : key(value), size(1), count(1), left(nullptr), right(nullptr) {
        }
};

/**
 * Time Complexity: O(N)
 */
void inorderTraversal(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    inorderTraversal(root->left);
    std::cout << root->key << " ";
    inorderTraversal(root->right);
}

/**
 * Time Complexity: O(H)
 */
int findMin(TreeNode* root) {
    if (root == nullptr) {
        return -1;
    }

    while (root->left != nullptr) {
        root = root->left;
    }

    return root->key;
}

/**
 * Time Complexity: O(H)
 */
int findMax(TreeNode* root) {
    if (root == nullptr) {
        return -1;
    }

    while(root->right != nullptr) {
        root = root->right;
    }

    return root->key;
}

/**
 * Time Complexity: O(H)
 */
bool search(TreeNode* root, int target) {
    if (root == nullptr) {
        return false;
    }

#if 0
    while (root != nullptr) {
        if (root->key == target) {
            return true;
        } else if (root->key > target) {
            root = root->left;
        } else {
            root = root->right;
        }
    }

    return false;
#endif // iteration

    if (root->key == target) {
        return true;
    } else if (root->key > target) {
        return search(root->left, target);
    } else {
        return search(root->right, target);
    }
}

/**
 * Time Complexity: O(H)
 */
TreeNode* insert(TreeNode* root, int value) {
    if (root == nullptr) {
        return new TreeNode(value);
    }

    if (root->key > value) {
        root->left = insert(root->left, value);
    } else if (root->key < value) {
        root->right = insert(root->right, value);
    } else {
        root->count++; /* duplicate value */
    }

    root->size = root->count + 
                (root->left ? root->left->size : 0) + 
                (root->right ? root->right->size : 0);
    
    return root;
}


static TreeNode* findSuccessor(TreeNode* root) {
    while (root->left != nullptr) {
        root = root->left;
    }
    return root;
}

/**
 * Time Complexity: O(H)
 * 
 *!NOTE: Returns the root node of the tree after removing the node with the given value
 */
TreeNode* remove(TreeNode* root, int value) {
    /* No node contains the value */
    if (root == nullptr) {
        return root;
    }

    if (root->key > value) {
        root->left = remove(root->left, value);
    } else if (root->key < value) {
        root->right = remove(root->right, value);
    } else {
        if (root->count > 1) {
            root->count--;
        } else {
            if (root->left == nullptr) {
                TreeNode* tmp = root->right;
                delete root;
                return tmp;
            } else if (root->right == nullptr) {
                TreeNode* tmp = root->left;
                delete root;
                return tmp;
            } else {
                TreeNode* successor = findSuccessor(root->right);
                root->key = successor->key;
                root->count = successor->count;
                /* Delete the successor node */
                root->right = remove(root->right, successor->key);
            }
        }
    }

    return root;
}

int queryRank(TreeNode* root, int v) {
    if (root == nullptr) {
        return 0;
    }

    /* If the key is equal to the value, we need to count the number of nodes in the left subtree. */
    if (root->key == v) {
        return (root->left ? root->left->size : 0) + 1;
    }

    /* If the key is smaller than the value, we need to search in the left subtree. */
    if (root->key > v) {
        return queryRank(root->left, v);
    }

    /* If the key is larger than the value:
     * the left subtree contains all the keys smaller than the value
     * the root contains all the keys smaller than the value
     */
    return queryRank(root->right, v) + (root->left ? root->left->size : 0) + root->count;
}

int querykth(TreeNode* root, int k) {
    if (root == nullptr) {
        return -1;
    }

    if (root->left) {
        /* Left sub-tree contains the kth element */
        if (root->left->size >= k) {
            return querykth(root->left, k);
        }

        /* Root is the  kth element */
        if (root->left->size + root->count >= k) {
            return root->key;
        }
    } else {
        if (k == 1) {
            return root->key;
        }
    }

    /* Right sub-tree contains the kth element
     * root->right == nullptr is considered
     */
    return querykth(root->right,
                    k - (root->left ? root->left->size : 0) - root->count);

}
```