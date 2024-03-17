---
{"dg-publish":true,"permalink":"/3-online-courses/0-cs-61-b/lectures/cs-61-b-2018-spring-learning-notes-chapter-9/","noteIcon":"","created":"2024-03-16T22:13:52.560+01:00","updated":"2024-03-17T11:26:21.883+01:00"}
---

## Slides
<iframe src="https://docs.google.com/presentation/d/1J7q2RImSbg26vrWMaYQwYo6_zPDrrdGRmwm_U2oY20s/edit#slide=id.g11d1bfb9b0_11_110" width="700" height="1000" ></iframe>

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

### Weighted Quick Union

![Z - assets/images/Pasted image 20240317071349.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240317071349.png)
- smaller height
	- faster to climb the trees
- no direct way to track the tree height
	- **tracking the tree's size** works asymptotically
- RULE
	- *link* root of smaller tree to larger tree
```java
public class WeightedQuickUnion implements DisjointSets {
    int parents[];
    int sizes[];

    public WeightedQuickUnion(int N) {
        for (int i = 0; i < N; i++) {
            parents[i] = -1;
            sizes[i] = 1;
        }
    }

    public int findRoot(int p) {
        while(parents[p] >= 0) {
            p = parents[p];
        }
        return p;
    }

    public void connect(int p, int q) {
        int rootP = findRoot(p);
        int rootQ = findRoot(q);
        if (rootP == rootQ) return;
        
        if (sizes[rootP] < sizes[rootQ]) {
            parents[rootP] = rootQ;
            sizes[rootQ] += sizes[rootP];
        } else {
            /* Including tie */
            parents[rootQ] = rootP;
            sizes[rootP] += sizes[rootQ];
        }
    }

    public boolean isConnected(int p, int q) {
        return findRoot(p) == findRoot(q);
    }
}

```

### Path Compression

![Z - assets/images/Pasted image 20240317101050.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240317101050.png)
- When doing `isConnected(15, 10)`
	- climb tree
		- `15->11->5->1->0`
		- `10->3`
	- flatten the tree branches
		- `15->0`
```java
public class WeightedQuickUnionWithPathCompression implements DisjointSets{
    int parents[];
    int sizes[];

    public WeightedQuickUnionWithPathCompression(int N) {
        parents = new int[N];
        sizes = new int[N];
        for (int i = 0; i < N; ++i) {
            parents[i] = -1;
            sizes[i] = 1;
        }
    }

    public int findRoot(int p) {
        /* Find the root */
        if (parents[p] < 0)
            return p;
        
        int root = findRoot(parents[p]);
        parents[p] = root;

        return root;
    }

    public void connect(int p, int q) {
        int rootP = findRoot(p);
        int rootQ = findRoot(q);

        /* already connected */
        if (rootP == rootQ) return;

        if (sizes[rootP] < sizes[rootQ]) {
            parents[rootP] = rootQ;
            sizes[rootQ] += sizes[rootP];
        } else {
            parents[rootQ] = rootP;
            sizes[rootP] += sizes[rootQ];
        }
    }

    public boolean isConnected(int p, int q) {
        return findRoot(p) == findRoot(q);
    }

    public static void main(String[] args) {
        WeightedQuickUnionWithPathCompression union = new WeightedQuickUnionWithPathCompression(16);
        union.connect(3, 10);
        union.connect(0, 3);
        union.connect(0, 4);
        union.connect(8, 14);
        union.connect(2, 8);
        union.connect(2, 9);
        union.connect(0, 2);
        union.connect(11, 15);
        union.connect(5, 11);
        union.connect(5, 12);
        union.connect(6, 13);
        union.connect(1, 5);
        union.connect(1, 6);
        union.connect(1, 7);
        union.connect(0, 1);
    }
}
```
![Z - assets/images/Pasted image 20240317112106.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020240317112106.png)
- solved by the path compression