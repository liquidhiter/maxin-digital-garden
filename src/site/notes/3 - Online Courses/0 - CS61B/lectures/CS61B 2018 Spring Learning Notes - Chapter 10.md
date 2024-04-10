---
{"dg-publish":true,"permalink":"/3-online-courses/0-cs-61-b/lectures/cs-61-b-2018-spring-learning-notes-chapter-10/","noteIcon":"","created":"2024-03-19T21:08:46.154+01:00","updated":"2024-04-10T06:44:20.441+02:00"}
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
## Implementation of BST in Java
```java
import java.util.Random;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.Queue;

public class BST<Key extends Comparable<Key>, Value> {
    private Node root;

    private class Node {
        private Key key;
        private Value val;
        private Node left, right;
        private int N;

        public Node(Key key, Value val, int N) {
            this.key = key;
            this.val = val;
            this.N = N;
        }
    }

    /**
     * Return a empty BST
     */
    public BST() {
        root = null;
    }

    /**
     * @return true if the BST is empty otherwise false
     */
    public Boolean empty() {
        return size() == 0;
    }

    /**
     * 
     * @return
     */
    public int size() {
        return size(root);
    }

    /**
     * 
     * @param n
     * @return
     */
    private int size(Node n) {
        if (n == null) {
            return 0;
        } else {
            return n.N;
        }
    }

    /**
     * 
     * @param k
     * @return
     */
    public Value get(Key k) {
        return get(root, k);
    }

    /**
     * 
     * @param n
     * @param k
     * @return
     */
    private Value get(Node n, Key k) {
        if (n == null) {
            return null;
        }

        if (k.compareTo(n.key) > 0) {
            return get(n.right, k);
        } else if (k.compareTo(n.key) < 0) {
            return get(n.left, k);
        } else {
            return n.val;
        }
    }

    /**
     * 
     * @param key
     * @param val
     */
    public void put(Key key, Value val) {
        root = put(root, key, val);
    }

    /**
     * Return the root node after inserting the new key-value pair
     *
     * @param n
     * @param k
     * @param v
     * @return
     */
    private Node put(Node n, Key k, Value v) {
        if (n == null) {
            return new Node(k, v, 1);
        }
        
        int cmp = k.compareTo(n.key);
        if (cmp > 0) {
            n.right = put(n.right, k, v);
        } else if (cmp < 0) {
            n.left = put(n.left, k, v);
        } else {
            /* Update the value if the key exists */
            n.val = v;
        }

        /* Update the size of subtree */
        n.N = size(n.left) + size(n.right) + 1;

        return n;
    }

    /**
     * 
     * @return
     */
    public Key min() {
        return root == null ? null : min(root).key;
    }

    /**
     * 
     * @param n
     * @return
     */
    private Node min(Node n) {
        if (n.left == null) {
            return n;
        }
        return min(n.left);
    }

    /**
     * 
     * @return
     */ 
    public Key max() {
        return root == null ? null : max(root).key;
    }

    /**
     * 
     * @param n
     * @return
     */
    private Node max(Node n) {
        if (n.right == null) {
            return n;
        }
        return max(n.right);
    }

    /**
     * 
     * @param k
     * @return
     */
    public Key floor(Key k) {
        return floor(root, k).key;
    }

    /**
     * 
     * @param n
     * @param k
     * @return
     */
    private Node floor(Node n, Key k) {
        if (n == null) {
            return null;
        }

        int cmp = k.compareTo(n.key);
        if (cmp == 0) {
            return n;
        }

        if (cmp < 0) {
            return floor(n.left, k);
        }
        /*
         * floor of k can exist in the right subtree only when there is a key not larger than the
         * given key otherwise, the current root (key > root) is the floor
         */
        Node tmp = floor(n.right, k);
        return tmp == null ? n : tmp;
    }

    /**
     * 
     * @param k
     * @return
     */
    public Key ceiling(Key k) {
        return ceiling(root, k).key;
    }

    /**
     * 
     * @param n
     * @param k
     * @return
     */
    private Node ceiling(Node n, Key k) {
        if (n == null) {
            return null;
        }

        int cmp = k.compareTo(n.key);
        if (cmp == 0)
            return n;
        if (cmp > 0)
            return floor(n.right, k);

        /*
         * ceiling of k can exist in the left subtree only when there is a key not smaller than the
         * given key otherwise, the current root (key < root) is the ceiling
         */
        Node tmp = floor(n.left, k);
        return tmp == null ? n : tmp;
    }

    /**
     * 
     * @param k
     * @return
     */
    public Key select(int k) {
        return select(root, k).key;
    }

    /**
     * 
     * @param n
     * @param k
     * @return
     */
    private Node select(Node n, int k) {
        if (n == null) {
            return null;
        }

        /*
         * left subtree of the current node contains N nodes, 0...k-1, the current node is the kth
         * one
         *
         * left subtree contains more than k nodes: the kth node must exist in the left subtree
         *
         * left subtree contains fewer than k nodes: the kth node must exist in the right subtree
         * NOTE: all nodes in the right subtree contains the key larger than the left subtree that's
         * to say, we only need to find the k - size(left subtree) - 1 th node
         */
        int sizeOfLeft = size(n.left);
        if (sizeOfLeft == k) {
            return n;
        } else if (sizeOfLeft > k) {
            return select(n.left, k);
        } else {
            return select(n.right, k - sizeOfLeft - 1);
        }
    }

    /**
     * 
     * @param k
     * @return
     */
    public int rank(Key k) {
        return rank(root, k);
    }

    /**
     * Return the rank of the given key in the tree grows from the tree node n
     * NOTE: if the given key doesn't exist in the tree, its rank as if it is inserted
     *       returns. In general, rank always return the rank of the given key correctly
     * @param n
     * @param k
     * @return
     */
    private int rank(Node n, Key k) {
        if (n == null) {
            return 0;
        }

        int cmp = k.compareTo(n.key);
        if (cmp > 0) {
            /*
             * Case 1: when the key is larger than the current root node's key which means the key
             * must exist in the right subtree and rank n.left.N + 1 + x
             */
            return size(n.left) + 1 + rank(n.right, k);
        } else if (cmp < 0) {
            /*
             * Case 2: when the key is smaller than the current root node's key which means the key
             * must exist in the left subtree
             */
            return rank(n.left, k);
        } else {
            /*
             * Case 3: when the key is the same as the current root node's key which means the key
             * ranks n.left
             */
            return size(n.left);
        }
    }

    /**
     * Delete the node with the minimum key
     */
    public void deleteMin() {
        if (empty()) {
            throw new RuntimeException("Empty BST!");
        }

        root = deleteMin(root);
    }

    /**
     * Helper function
     */
    private Node deleteMin(Node n) {
        if (n.left == null) {
            return n.right;
        }

        n.left = deleteMin(n.left);

        /* Update the size of all subtrees from the bottom to the top */
        n.N = size(n.left) + size(n.right) + 1;

        /* Returns the root node with the minimum key deleted */
        return n;
    }

    /**
     * 
     */
    public void deleteMax() {
        if (empty()) {
            throw new RuntimeException("Empty BST!");
        }

        root = deleteMax(root);
    }

    /**
     * 
     * @param n
     * @return
     */
    private Node deleteMax(Node n) {
        if (n.right == null) {
            return n.left;
        }

        n.right = deleteMax(n.right);

        /* Update the size of all subtrees from the bottom to the top */
        n.N = size(n.left) + size(n.right) + 1;

        /* Returns the root node with the minimum key deleted */
        return n;
    }

    /**
     * Delete the node with the given key
     * Cases:
     * 1> node to be deleted is a leaf node
     * 2> node to be deleted is a parent node (has left or right non-null node)
     * 3> node to be deleted is a root node (left and right non-null nodes)
     * @param k
     */
    public void delete(Key k) {
        if (empty()) {
            throw new RuntimeException("Empty BST!");
        }

        root = delete(root, k);
    }

    /**
     * Helper function. See the comments for delete(Key k)
     * @param n
     * @param k
     * @return
     */
    private Node delete(Node n, Key k) {
        if (n == null) {
            return null;
        }

        int cmp = k.compareTo(n.key);
        if (cmp > 0) {
            /* Key exists in the right subtree */
            n.right = delete(n.right, k);
        } else if (cmp < 0) {
            /* Key exist in the left subtree */
            n.left = delete(n.left, k);
        } else {
            /* the current node is the one to be deleted */
            // case 1 & 2: leaf node or parent nodes
            if (n.left == null) {
                return n.right;
            } else if (n.right == null) {
                return n.left;
            // case 3: root node
            } else  {
                /* First save the current node */
                Node root = n;

                /* Find the successor node - minimum in the right subtree */
                n = min(n.right);

                /* Remember to delete the successor node from the right subtree */
                n.right = delete(root.right, n.key);

                /* Remember to update the left subtree */
                n.left = root.left;

                /* Returned on line 433 */
            }
        }

        /* Update the tree sizes */
        n.N = size(n.left) + size(n.right) + 1;

        return n;
    }

    /**
     * Print the BST in inorder
     */
    public void printInOrder() {
        printInOrder(root);
    }

    /**
     * 
     * @param n
     */
    private void printInOrder(Node n) {
        if (n == null) {
            return;
        }

        /* Print the left subtree */
        printInOrder(n.left);
        /* Print the root node */
        StdOut.println(n.key);
        /* Print the right subtree */
        printInOrder(n.right);
    }

    /**
     * 
     * @return
     */
    public Iterable<Key> keys() {
        return keys(min(), max());
    }

    /**
     * 
     * @param lo
     * @param hi
     * @return
     */
    public Iterable<Key> keys(Key lo, Key hi) {
        Queue<Key> queue = new Queue<>();
        keys(root, queue, lo, hi);
        return queue;
    }

    /**
     * !NOTE: first left subtree, then root node, last right subtree (InOrder)
     * @param n
     * @param queue
     * @param lo
     * @param hi
     */
    private void keys(Node n, Queue<Key> queue, Key lo, Key hi) {
        if (n == null) {
            return;
        }

        int cmpToLo = lo.compareTo(n.key);
        int cmpToHigh =hi.compareTo(n.key);
        /* Case 1: if lo is smaller than the current node's key, go to the left subtree */
        if (cmpToLo < 0) {
            keys(n.left, queue, lo, hi);
        }
        /* Case 2: if lo <= current node's key <= hi, add the current node's key to the queue */
        if (cmpToLo <= 0 && cmpToHigh >= 0) {
            queue.enqueue(n.key);
        }
        /* Case 3: if hi is larger than the current node's key, go to the right subtree */
        if (cmpToHigh > 0) {
            keys(n.right, queue, lo, hi);
        }
    }
}
```

## Topic: Priority Queue
### typical interface
```java
/** (Min) Priority Queue: Allowing tracking and removal of the
  * smallest item in a priority queue. */
public interface MinPQ<Item> {
	/** Adds the item to the priority queue. */
	public void add(Item x);
	/** Returns the smallest item in the priority queue. */
	public Item getSmallest();
	/** Removes the smallest item from the priority queue. */
	public Item removeSmallest();
	/** Returns the size of the priority queue. */
	public int size();
}
```
- application
    - keep track of the `smallest`, `largest` or `best`, etc

### example: find the M most unharmonious messsages
```java
public List<String> unharmoniousTexts(Sniffer sniffer, int M) {
    	Comparator<String> cmptr = new HarmoniousnessComparator();
    	MinPQ<String> unharmoniousTexts = new HeapMinPQ<Transaction>(cmptr);
    	for (Timer timer = new Timer(); timer.hours() < 24; ) {
        	unharmoniousTexts.add(sniffer.getNextMessage());
        if (unharmoniousTexts.size() > M) 
           { unharmoniousTexts.removeSmallest(); }
    }
    ArrayList<String> textlist = new ArrayList<String>();
    	while (unharmoniousTexts.size() > 0) {
            textlist.add(unharmoniousTexts.removeSmallest());
    	}
    	return textlist;
}
```
- the above is a better implementation, which only requires `O(M)` memory space
    - this is achieved by always keeping `M` items tracked at the same time at most

### comparison of time complexity with different underlying data structures
|             | Ordered Array | Bushy BST | Hash Table | Heap
|-------------|-----------|-----------|------------|----------|
| add | Θ(N) | Θ(log N) | Θ(1) | Θ(log N) |
| getSmallest | Θ(1) | Θ(log N) | Θ(N) | Θ(1) |
| removeSmallest | Θ(N) | Θ(log N) | Θ(N) | Θ(log N) |

## Topic: Heap
### binary min-heap
- definition
    - binary tree that is complete, and obeys min-heap property
- min-heap
    - every node is less than or equal to both of its children
- complete
    - missing items only at the bottom level if any, all nodes are as far left as possible
![Z - assets/images/Pasted image 20240410064257.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240410064257.png)
### array based implementation
![Z - assets/images/Pasted image 20240410064319.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240410064319.png)
- advantages
    - easy to implement (compared with tress)
    - memory efficient (compared with trees)

### lab10

```java
import org.junit.Test;
import static org.junit.Assert.*;

/**
 * A Generic heap class. Unlike Java's priority queue, this heap doesn't just
 * store Comparable objects. Instead, it can store any type of object
 * (represented by type T), along with a priority value. Why do it this way? It
 * will be useful later on in the class...
 */
public class ArrayHeap<T> implements ExtrinsicPQ<T> {
    private Node[] contents;
    private int size;

    private final static int ROOT = 1;

    public ArrayHeap() {
        contents = new ArrayHeap.Node[16];

        /* Add a dummy item at the front of the ArrayHeap so that the getLeft,
         * getRight, and parent methods are nicer. */
        contents[0] = null;

        /* Even though there is an empty spot at the front, we still consider
         * the size to be 0 since nothing has been inserted yet. */
        size = 0;
    }

    /**
     * Returns the index of the node to the left of the node at i.
     */
    private static int leftIndex(int i) {
        /* left node is always at the position of i * 2 */
        return i << 1;
    }

    /**
     * Returns the index of the node to the right of the node at i.
     */
    private static int rightIndex(int i) {
        /* right node is always at the position of i * 2 + 1 */
        return (i << 1) + 1;
    }

    /**
     * Returns the index of the node that is the parent of the node at i.
     */
    private static int parentIndex(int i) {
        /* parent node is always at the position of i / 2 */
        return i >> 1;
    }

    /**
     * Gets the node at the ith index, or returns null if the index is out of
     * bounds.
     */
    private Node getNode(int index) {
        if (!inBounds(index)) {
            return null;
        }
        return contents[index];
    }

    /**
     * Returns true if the index corresponds to a valid item. For example, if
     * we have 5 items, then the valid indices are 1, 2, 3, 4, 5. Index 0 is
     * invalid because we leave the 0th entry blank.
     */
    private boolean inBounds(int index) {
        if ((index > size) || (index < 1)) {
            return false;
        }
        return true;
    }

    /**
     * Swap the nodes at the two indices.
     */
    private void swap(int index1, int index2) {
        Node node1 = getNode(index1);
        Node node2 = getNode(index2);
        contents[index1] = node2;
        contents[index2] = node1;
    }


    /**
     * Returns the index of the node with smaller priority. Precondition: not
     * both nodes are null.
     */
    private int min(int index1, int index2) {
        Node node1 = getNode(index1);
        Node node2 = getNode(index2);
        if (node1 == null) {
            return index2;
        } else if (node2 == null) {
            return index1;
        } else if (node1.myPriority < node2.myPriority) {
            return index1;
        } else {
            return index2;
        }
    }


    /**
     * Bubbles up the node currently at the given index.
     */
    private void swim(int index) {
        // Throws an exception if index is invalid. DON'T CHANGE THIS LINE.
        validateSinkSwimArg(index);

        int parentIndex = parentIndex(index);
        /* Already at the root, no more swim */
        if (parentIndex == 0) {
            return;
        }

        /** Bubble up until it's smaller than its parent (priority)
         *  Stop when reaching out the root node
         */
        if (min(index, parentIndex) == index) {
            /* Now the Node at the given index is swapped to its parent's position */
            swap(index, parentIndex);

            /* No need to continue bubble up if reaching out to the root node already */
            swim(parentIndex);
        }
    }

    /**
     * Bubbles down the node currently at the given index.
     */
    private void sink(int index) {
        // Throws an exception if index is invalid. DON'T CHANGE THIS LINE.
        validateSinkSwimArg(index);

        int left = leftIndex(index);
        int right = rightIndex(index);
        /* Node with smaller priority bubble up */
        int minIndex = min(left, right);
        if (minIndex > size) {
            return;
        }

        if (min(index, minIndex) == minIndex) {
            /* Node at index is sinked to minIndex */
            swap(index, minIndex);

            /* Sink until reaching out to the bottom */
            sink(minIndex);
        }

    }

    /**
     * Inserts an item with the given priority value. This is enqueue, or offer.
     * To implement this method, add it to the end of the ArrayList, then swim it.
     */
    @Override
    public void insert(T item, double priority) {
        /* If the array is totally full, resize. */
        if (size + 1 == contents.length) {
            resize(contents.length * 2);
        }

        /** Append the new item to the array
         *  size += 1 because the initial value of size = 0
         */
        contents[++size] = new Node(item, priority);
        /* Bubble up the new item if essential */
        swim(size);
    }

    /**
     * Returns the Node with the smallest priority value, but does not remove it
     * from the heap. To implement this, return the item in the 1st position of the ArrayList.
     */
    @Override
    public T peek() {
        return size == 0 ? null : getNode(ROOT).item();
    }

    /**
     * Returns the Node with the smallest priority value, and removes it from
     * the heap. This is dequeue, or poll. To implement this, swap the last
     * item from the heap into the root position, then sink the root. This is
     * equivalent to firing the president of the company, taking the last
     * person on the list on payroll, making them president, and then demoting
     * them repeatedly. Make sure to avoid loitering by nulling out the dead
     * item.
     */
    @Override
    public T removeMin() {
        if (size == 0) {
            return null;
        }

        /* Swap the node at the root and the bottom */
        if (size > 1) {
            swap(ROOT, size);
        }

        /* Nulling out the minimum before sink */
        T val = getNode(size).item();
        contents[size] = null;

        if (size > 1) {
            /* Sink the new root if necessary */
            sink(ROOT);
        }

        size -= 1;

        return val;
    }

    /**
     * Returns the number of items in the PQ. This is one less than the size
     * of the backing ArrayList because we leave the 0th element empty. This
     * method has been implemented for you.
     */
    @Override
    public int size() {
        return size;
    }

    /**
     * Change the node in this heap with the given item to have the given
     * priority. You can assume the heap will not have two nodes with the same
     * item. Check item equality with .equals(), not ==. This is a challenging
     * bonus problem, but shouldn't be too hard if you really understand heaps
     * and think about the algorithm before you start to code.
     */
    @Override
    public void changePriority(T item, double priority) {
        if (item == null) {
            return;
        }

        Node match = null;
        int index = -1;
        /* O(N) */
        for (int i = 1; i <= size; ++i) {
            if (contents[i].item().equals(item)) {
                match = contents[i];
                index = i;
                break;
            }
        }

        /* Do nothing if the priority is the same */
        if (match.priority() == priority) {
            return;
        } else if (match.priority() > priority) {
            match.myPriority = priority;
            if (index > 1) {
                /* Swim the matched node due to its higher priority */
                swim(index);
            }
        } else {
            /* If changing to a larger priority */
            match.myPriority = priority;
            if (index < size) {
                /* Sink the matched node due to its lower priority */
                sink(index);
            }
        }
    }

    /**
     * Prints out the heap sideways. Provided for you.
     */
    @Override
    public String toString() {
        return toStringHelper(1, "");
    }

    /* Recursive helper method for toString. */
    private String toStringHelper(int index, String soFar) {
        if (getNode(index) == null) {
            return "";
        } else {
            String toReturn = "";
            int rightChild = rightIndex(index);
            toReturn += toStringHelper(rightChild, "        " + soFar);
            if (getNode(rightChild) != null) {
                toReturn += soFar + "    /";
            }
            toReturn += "\n" + soFar + getNode(index) + "\n";
            int leftChild = leftIndex(index);
            if (getNode(leftChild) != null) {
                toReturn += soFar + "    \\";
            }
            toReturn += toStringHelper(leftChild, "        " + soFar);
            return toReturn;
        }
    }


    /**
     * Throws an exception if the index is invalid for sinking or swimming.
     */
    private void validateSinkSwimArg(int index) {
        if (index < 1) {
            throw new IllegalArgumentException("Cannot sink or swim nodes with index 0 or less");
        }
        if (index > size) {
            throw new IllegalArgumentException("Cannot sink or swim nodes with index greater than current size.");
        }
        if (contents[index] == null) {
            throw new IllegalArgumentException("Cannot sink or swim a null node.");
        }
    }

    private class Node {
        private T myItem;
        private double myPriority;

        private Node(T item, double priority) {
            myItem = item;
            myPriority = priority;
        }

        public T item(){
            return myItem;
        }

        public double priority() {
            return myPriority;
        }

        @Override
        public String toString() {
            return myItem.toString() + ", " + myPriority;
        }
    }


    /** Helper function to resize the backing array when necessary. */
    private void resize(int capacity) {
        Node[] temp = new ArrayHeap.Node[capacity];
        for (int i = 1; i < this.contents.length; i++) {
            temp[i] = this.contents[i];
        }
        this.contents = temp;
    }

    @Test
    public void testIndexing() {
        assertEquals(6, leftIndex(3));
        assertEquals(10, leftIndex(5));
        assertEquals(7, rightIndex(3));
        assertEquals(11, rightIndex(5));

        assertEquals(3, parentIndex(6));
        assertEquals(5, parentIndex(10));
        assertEquals(3, parentIndex(7));
        assertEquals(5, parentIndex(11));
    }

    @Test
    public void testSwim() {
        ArrayHeap<String> pq = new ArrayHeap<>();
        pq.size = 7;
        for (int i = 1; i <= 7; i += 1) {
            pq.contents[i] = new ArrayHeap<String>.Node("x" + i, i);
        }
        // Change item x6's priority to a low value.

        pq.contents[6].myPriority = 0;
        System.out.println("PQ before swimming:");
        System.out.println(pq);

        // Swim x6 upwards. It should reach the root.

        pq.swim(6);
        System.out.println("PQ after swimming:");
        System.out.println(pq);
        assertEquals("x6", pq.contents[1].myItem);
        assertEquals("x2", pq.contents[2].myItem);
        assertEquals("x1", pq.contents[3].myItem);
        assertEquals("x4", pq.contents[4].myItem);
        assertEquals("x5", pq.contents[5].myItem);
        assertEquals("x3", pq.contents[6].myItem);
        assertEquals("x7", pq.contents[7].myItem);
    }

    @Test
    public void testSink() {
        ArrayHeap<String> pq = new ArrayHeap<>();
        pq.size = 7;
        for (int i = 1; i <= 7; i += 1) {
            pq.contents[i] = new ArrayHeap<String>.Node("x" + i, i);
        }
        // Change root's priority to a large value.
        pq.contents[1].myPriority = 10;
        System.out.println("PQ before sinking:");
        System.out.println(pq);

        // Sink the root.
        pq.sink(1);
        System.out.println("PQ after sinking:");
        System.out.println(pq);
        assertEquals("x2", pq.contents[1].myItem);
        assertEquals("x4", pq.contents[2].myItem);
        assertEquals("x3", pq.contents[3].myItem);
        assertEquals("x1", pq.contents[4].myItem);
        assertEquals("x5", pq.contents[5].myItem);
        assertEquals("x6", pq.contents[6].myItem);
        assertEquals("x7", pq.contents[7].myItem);
    }

    @Test
    public void testPeak() {
        ArrayHeap<String> pq = new ArrayHeap<>();
        assertNull(pq.peek());
        pq.insert("c", 3);
        assertEquals("c", pq.contents[1].myItem);
        assertEquals("c", pq.peek());
        pq.insert("d", 2);
        assertEquals("d", pq.peek());
        pq.contents[2].myPriority = 0;
        pq.swim(2);
        assertEquals("c", pq.peek());
    }


    @Test
    public void testInsert() {
        ArrayHeap<String> pq = new ArrayHeap<>();
        pq.insert("c", 3);
        assertEquals("c", pq.contents[1].myItem);

        pq.insert("i", 9);
        assertEquals("i", pq.contents[2].myItem);

        pq.insert("g", 7);
        pq.insert("d", 4);
        assertEquals("d", pq.contents[2].myItem);

        pq.insert("a", 1);
        assertEquals("a", pq.contents[1].myItem);

        pq.insert("h", 8);
        pq.insert("e", 5);
        pq.insert("b", 2);
        pq.insert("c", 3);
        pq.insert("d", 4);
        System.out.println("pq after inserting 10 items: ");
        System.out.println(pq);
        assertEquals(10, pq.size());
        assertEquals("a", pq.contents[1].myItem);
        assertEquals("b", pq.contents[2].myItem);
        assertEquals("e", pq.contents[3].myItem);
        assertEquals("c", pq.contents[4].myItem);
        assertEquals("d", pq.contents[5].myItem);
        assertEquals("h", pq.contents[6].myItem);
        assertEquals("g", pq.contents[7].myItem);
        assertEquals("i", pq.contents[8].myItem);
        assertEquals("c", pq.contents[9].myItem);
        assertEquals("d", pq.contents[10].myItem);
    }


    @Test
    public void testInsertAndRemoveOnce() {
        ArrayHeap<String> pq = new ArrayHeap<>();
        pq.insert("c", 3);
        pq.insert("i", 9);
        pq.insert("g", 7);
        pq.insert("d", 4);
        pq.insert("a", 1);
        pq.insert("h", 8);
        pq.insert("e", 5);
        pq.insert("b", 2);
        pq.insert("c", 3);
        pq.insert("d", 4);
        String removed = pq.removeMin();
        assertEquals("a", removed);
        assertEquals(9, pq.size());
        assertEquals("b", pq.contents[1].myItem);
        assertEquals("c", pq.contents[2].myItem);
        assertEquals("e", pq.contents[3].myItem);
        assertEquals("c", pq.contents[4].myItem);
        assertEquals("d", pq.contents[5].myItem);
        assertEquals("h", pq.contents[6].myItem);
        assertEquals("g", pq.contents[7].myItem);
        assertEquals("i", pq.contents[8].myItem);
        assertEquals("d", pq.contents[9].myItem);
    }

    @Test
    public void testInsertAndRemoveAllButLast() {
        ExtrinsicPQ<String> pq = new ArrayHeap<>();
        pq.insert("c", 3);
        pq.insert("i", 9);
        pq.insert("g", 7);
        pq.insert("d", 4);
        pq.insert("a", 1);
        pq.insert("h", 8);
        pq.insert("e", 5);
        pq.insert("b", 2);
        pq.insert("c", 3);
        pq.insert("d", 4);

        int i = 0;
        String[] expected = {"a", "b", "c", "c", "d", "d", "e", "g", "h", "i"};
        while (pq.size() > 1) {
            assertEquals(expected[i], pq.removeMin());
            i += 1;
        }
    }

    @Test
    public void testChangePriority() {
        ArrayHeap<String> pq = new ArrayHeap<>();
        pq.insert("c", 3);
        pq.insert("i", 9);
        pq.insert("g", 7);
        pq.insert("a", 1);
        pq.insert("h", 8);
        pq.insert("e", 5);
        pq.insert("b", 2);
        pq.insert("d", 4);

        pq.changePriority("a", 10);
        assertNotEquals("a", pq.peek());
        assertEquals("b", pq.peek());
        assertEquals("a", pq.contents[7].item());

        pq.changePriority("a", 1);
        assertEquals("a", pq.peek());
        assertNotEquals("b", pq.peek());
        assertEquals("a", pq.contents[1].item());
        assertEquals("b", pq.contents[3].item());
        assertEquals("e", pq.contents[7].item());

        pq.changePriority("c", 0);
        assertEquals("c", pq.peek());
        assertNotEquals("a", pq.peek());
        assertEquals("c", pq.contents[1].item());
        assertEquals("a", pq.contents[2].item());

        pq.changePriority("c", 10);
        assertNotEquals("c", pq.peek());
        assertEquals("c", pq.contents[8].item());

        pq.changePriority("i", 3.5);
        assertEquals("i", pq.contents[2].item());
        assertEquals("d", pq.contents[4].item());

    }

    /*
    public static void main(String[] args) {
        ArrayHeap<String> launcher = new ArrayHeap<>();
        launcher.testSwim();
        launcher.testIndexing();
        launcher.testSink();
        launcher.testInsert();
        launcher.testPeak();
        launcher.testInsertAndRemoveOnce();
        launcher.testInsertAndRemoveAllButLast();
        launcher.testChangePriority();
    } */
}
```

## Topic: Advanced Trees
### traversals
- depth first traversals
    - preorder
    - inorder
    - postorder

```java
public void PreOrder(BSTNode x) {
	if (x != null) {
		print(x.key);
		print(x.left);
		print(x.right);
	}
}
```

```java
public void InOrder(BSTNode x) {
	if (x != null) {
		print(x.left);
		print(x.key);
		print(x.right);
	}
}
```

```java
public void PostOrder(BSTNode x) {
  if (x != null) {
    PostOrder(x.left);
    PostOrder(x.right);
    print(x.key);
  }
}
```

- application
	- post-order traversal for calculating file sizes
	 ![Z - assets/images/Pasted image 20240410063618.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240410063618.png)	
	  > children then parent

- [ ] single-responsibility principle
  - `A module should be repsonsible to one, and only one actor`
  - example
    - a module that compiles and prints a report
      - compilation process can change
      - report format can change
      - report contents can change
  - reference: https://en.wikipedia.org/wiki/Single_responsibility_principle
- [ ] open-closed principle
  - OOP concept
  - `software entities should be open for extension, but closed for modification`
  - reference: https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle
### visitor pattern
- *A **visitor pattern** is a [software design pattern](https://en.wikipedia.org/wiki/Software_design_pattern) that separates the [algorithm](https://en.wikipedia.org/wiki/Algorithm) from the [object](https://en.wikipedia.org/wiki/Object_(computer_science)) structure. Because of this separation new operations can be added to existing object structures without modifying the structures.*
- reference:
```java
void preorderTraverse(Tree<Label> T, Action<Label> vistor) {
   if (T == null) { return; }
   vistor.visit(T); /* before we hard coded a print */
   preorderTraverse(T.left, vistor);
   preorderTraverse(T.right, vistor);
}

/* Visitor class */
interface Action<Label> {
    void visit(Tree<Label> T);
}

/* Abstract method in vistee */
class FindPig implements Action<String> {
    boolean found = false;
    @Override
    void visit(Tree<String> T) {
        if ("pig".equals(T.label)) {
            found = true;
        }
    }
}

preorderTraverse(tree, new FindPig());
```
### level order traversal
```java
void levelorderTraverse(BSTNode x) {
    for (int i = 0; i < x.height; i++) {
        visitLevel(x, i);
    }
}

void visitLevel(BSTNode x, int level) {
    if (T == null) return;
    
    if (level == 0) print(x.key);
    else {
        visitLevel(x.left, level - 1);
        visitLevel(x.right, level - 1);
    }
}
```
### tree height
- difference between bushy and spindly trees
```Python
``````Python
``````Python
``````Python
```
## range finding
#### geometric search
```java
public Set<Label> findInRange(Tree T, Label min, Label max) {
    Set<Label> results = new Set<>();
    rangeFindHelper(T, min, max, results);
    return results;
}

private void rangeFindHelper(Tree T, Label min, Label max, Set<Label> s) {
    if (T == null) return;
    int cmpMin = min.compareTo(T.label);
    int cmpMax = max.compareTo(T.label);
    if (cmpMin < 0) {
        rangeFindHelper(T.right, min, max, s);
    } else if (cmpMax > 0) {
        rangeFindHelper(T.left, min, max, s);
    } else {
        s.add(T.label);
    }
}
```
#### prune tree
	![Z - assets/images/Pasted image 20240410063643.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240410063643.png)
### spatial trees
![Z - assets/images/Pasted image 20240410063711.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240410063711.png)
### iterator

```java
private class preorderIterator implements Iterator<Label> {
    Stack<Tree<Label>> s = new Stack<>();
    public preorderIterator() {
        s.push(Tree.this);
    }
    public boolean hasNext() {
        return (!s.isEmpty());
    }
    
    public Label next() {
        Tree<Label> node = s.pop();
        for (int i = 0; i < node.numOfChild(); i++) {
            s.push(node.child(i));
        }
        return node.label;
    }
}
```