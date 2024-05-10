---
{"dg-publish":true,"permalink":"/05-online-courses/c00-cs-61-b/lectures/cs-61-b-2018-spring-learning-notes-chapter-09/","noteIcon":"","created":"2024-03-16T22:13:52.560+01:00","updated":"2024-04-10T06:52:11.208+02:00"}
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

### lab6
```java
public class UnionFind {
    /**
     * DO NOT DELETE OR MODIFY THIS, OTHERWISE THE TESTS WILL NOT PASS.
     * You can assume that we are only working with non-negative integers as the items
     * in our disjoint sets.
     */
    private int[] data;

    /* Creates a UnionFind data structure holding N items. Initially, all
       items are in disjoint sets. */
    public UnionFind(int N) {
        data = new int[N];
        for (int i = 0; i < N; ++i) {
            data[i] = -1;
        }
    }

    private void validate(int idx) {
        if (idx < 0 || idx >= data.length)
            throw new IllegalArgumentException("Given index is out of bounds!");
    }

    /* Returns the size of the set V belongs to. */
    public int sizeOf(int v) {
        return data[find(v)];
    }

    /* Returns the parent of V. If V is the root of a tree, returns the
       negative size of the tree for which V is the root. */
    public int parent(int v) {
        validate(v);
        return data[v];
    }

    /* Returns true if nodes/vertices V1 and V2 are connected. */
    public boolean connected(int v1, int v2) {
        return find(v1) == find(v2);
    }

    /**
     * Helper function
     * @param v
     * @return
     */
    private int findRecursive(int v) {
        if (data[v] < 0) {
            return v;
        } else {
            data[v] = find(data[v]);
            return data[v];
        }
    }

    /* Returns the root of the set V belongs to. Path-compression is employed
       allowing for fast search-time. If invalid items are passed into this
       function, throw an IllegalArgumentException. */
    public int find(int v) {
        validate(v);
        return findRecursive(v);
    }

    /* Connects two items V1 and V2 together by connecting their respective
       sets. V1 and V2 can be any element, and a union-by-size heuristic is
       used. If the sizes of the sets are equal, tie break by connecting V1's
       root to V2's root. Union-ing a item with itself or items that are
       already connected should not change the structure. */
    public void union(int v1, int v2) {
        int root1 = find(v1);
        int root2 = find(v2);
        if (root1 == root2) return;
        
        /* ALWAYS connect a smaller tree to the larger one
         * tie breaks by connecting the root of the smaller tree to the root of the larger tree also
         * !NOTE: size is negative
         */
        if (data[root1] >= data[root2]) {
            data[root2] += data[root1];
            data[root1] = root2;
        } else {
            data[root1] += data[root2];
            data[root2] = root1;
        }
    }

    /**
     * DO NOT DELETE OR MODIFY THIS, OTHERWISE THE TESTS WILL NOT PASS.
     */
    public int[] returnData() {
        return data;
    }
}

```

## hw3 hashing
- typical implementation of `equal` method
```java
    /**
     * Compares this date to the specified date.
     *
     * @param  other the other date
     * @return {@code true} if this date equals {@code other}; {@code false} otherwise
     */
    @Override
    public boolean equals(Object other) {
        if (other == this) return true;
        if (other == null) return false;
        if (other.getClass() != this.getClass()) return false;
        Date that = (Date) other;
        return (this.month == that.month) && (this.day == that.day) && (this.year == that.year);
    }
```
- according to java specification
```markdown
Reflexive: x.equals(x) must be true for any non-null x.
Symmetric: x.equals(y) must be the same as y.equals(x) for any non-null x and y.
Transitive: if x.equals(y) and y.equals(z), then x.equals(z) for any non-null x, y, and z.
Consistent: x.equals(y) must return the same result if called multiple times, so long as the object referenced by x and y do not change.
Not-equal-to-null: x.equals(null) should be false for any non-null x.
```
- overwrite `hashCode` when `equal` is overwritten
```markdown
“Note that it is generally necessary to override the hashCodemethod whenever the equals method is overridden, so as to maintain the general contract for the hashCode method, which states that equal objects must have equal hash codes.”
```
    - reasoning
        - objects are stored in one `hashSet`
        - `contains` is used to check the existence of certain objects
        - which actually compares the equality of the objects
        - that's what described in the Java specification, "equal objects must have equal has codes"
    - NOTE:
        - `hashCode` returns the address of the objects by default

```markdown
If you’re getting a figure similar the first where everything is in two buckets, here’s why: Even though your hashCode is perfect, it’s always returning a multiple of 5. Try changing M to 9 in HashTableVisualizer, and you should see a nice spread of values. Understanding why this is happening is a great idea.
```
- 5k mod 10 always return 0 or 5, while 5k mod 9 returns number evenly

```java
    /* Create a list of Complex Oomages called deadlyList
     * that shows the flaw in the hashCode function.
     */
    @Test
    public void testWithDeadlyParams() {
        List<Oomage> deadlyList = new ArrayList<>();
        for (int i = 0; i < 255; ++i) {
            List<Integer> deadParam = new ArrayList<>();
            deadParam.add(i);
            for (int j = 0; j < 10; ++j) {
                deadParam.add(0);
            }
            ComplexOomage coomage = new ComplexOomage(deadParam);
            deadlyList.add(coomage);
        }

        assertTrue(OomageTestUtility.haveNiceHashCodeSpread(deadlyList, 10));
    }
```
- root cause:
    - 256 = 0b100000000, Integer in Java has limited range (signed)
    - hashCode is implemented by multiplying with 256 repeatedly, which always generate 0 when all non-zero bits are shifted out.
    - this happens to all the number with the same pattern
    - common trick to use prime numbers when calculating hashCode
