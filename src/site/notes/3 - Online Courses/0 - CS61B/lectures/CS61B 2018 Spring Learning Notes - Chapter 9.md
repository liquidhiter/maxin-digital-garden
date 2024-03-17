---
{"dg-publish":true,"permalink":"/3-online-courses/0-cs-61-b/lectures/cs-61-b-2018-spring-learning-notes-chapter-9/","noteIcon":"","created":"2024-03-16T22:13:52.560+01:00","updated":"2024-03-17T07:23:42.717+01:00"}
---

## Topic: Disjoint Sets
- definition of a disjoint set
	- Two sets are named _disjoint sets_ if they have no elements in common. A Disjoint-Sets (or Union-Find) data structure keeps track of a fixed number of elements partitioned into a number of _disjoint sets_.
- data structure
	- disjoint set
- APIs
	- `connext(x, y)`
	- `isConnected(x, y)`
![Z - assets/images/Pasted image 20240316222307.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240316222307.png)
- example: 
	- how disjoint set on integers work
- how to implement?
	- all connected elements are stored in the same container ?
	- two containers are merged once there is any common element?
		- or consider the single element as one meta container?
			- then all connect is merge
- NOTE:
	- order is not important here, only connectivity matters!

### array `id`

![Z - assets/images/Pasted image 20240316225121.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240316225121.png)
- implementation
```java
public class QuickFindDs implements DisjointSets {
    
    private int[] id;
    
    public QuickFindDs(int N) {
        id = new int[N];
        for (int i = 0; i < N; ++i) {
            id[i] = -1;
        }
    }

    /**
     * Connect p with q
     */
    public void connect(int p, int q) {
        // int prevId = id[p];
        int pid = id[p];
        int qid = id[q];
        for (int i = 0; i < id.length; ++i) {
            if (id[i] == pid) {
                id[i] = qid;
            }
        }
    }

    public boolean isConnected(int p, int q) {
        return id[p] == id[q];
    }

    public static void main(String[] args) {
        QuickFindDs q = new QuickFindDs(10);
        q.connect(4, 3);
        q.connect(3, 8);
        assert(q.isConnected(4, 8));
        assert(q.isConnected(3, 8));
        assert(q.isConnected(3, 4));
        System.out.println(q.isConnected(4, 8));
    }
}
```
- complexity
	- `connect`
		- O(N)
	- `isConnected`
		- O(1)

### quick union
![Z - assets/images/Pasted image 20240317065002.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240317065002.png)
- because `parent[3] = -1` which means it's not connected to {0, 1, 2, 4, 5}
	- it actually becomes a single element set
```java
public class QuickUnionDS implements DisjointSets {
    int[] parents;

    public QuickUnionDS(int N) {
        parents = new int[N];
        for (int i = 0; i < N; ++i) {
            parents[i] = -1;
        }
    }
    
    private int findRoot(int p) {
        while(parents[p] >= 0) {
            p = parents[p];
        }
        return p;
    }

    public void connect(int p, int q) {
        int rootP = findRoot(p);
        int rootQ = findRoot(q);
        /* q's root becomes p's */
        parents[rootQ] = rootP;
    }

    public boolean isConnected(int p, int q) {
        return findRoot(p) == findRoot(q);
    }

    public static void main(String[] args) {
        QuickUnionDS union = new QuickUnionDS(10);
        union.connect(1, 2);
        assert(union.isConnected(1, 2));
        union.connect(0, 1);
        assert(union.isConnected(0, 1));
        assert(union.isConnected(1, 2));

        union.connect(3, 5);
        assert(union.isConnected(3, 5));
        assert(!union.isConnected(0, 3));
        assert(!union.isConnected(2, 5));
        union.connect(2, 5);
        assert(union.isConnected(2, 5));
        assert(union.isConnected(4, 3));
    }
}

```
- complexity
	- `connect`
		- O(N) in the worst case
			- imbalanced tree
	- `isConnected`
		- O(N) in the worst case
			- imbalanced tree

###


![Z - assets/images/Pasted image 20240317071349.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240317071349.png)
- smaller height
	- faster to climb the trees
- no direct way to track the tree height
	- **tracking the tree's size** works asymptotically
- RULE
	- *link* root of smaller tree to larger tree