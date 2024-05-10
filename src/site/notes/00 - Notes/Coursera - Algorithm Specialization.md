---
{"dg-publish":true,"permalink":"/00-notes/coursera-algorithm-specialization/","noteIcon":"","created":"2024-01-27T07:56:48.891+01:00","updated":"2024-01-27T07:57:50.130+01:00"}
---

## Week 1

### sum of two digits
skipped...
### maximum pairwise product
fast solution shown in the video is actually over-complicated
- demo solution

    ```c++
    long long MaxPairwiseProduct(const std::vector<int&> numbers) {
        auto n = numbers.size();

        /*Find the index of the largest number*/
        int index_of_largest = -1;
        for (auto i = 0; i < n; ++i) {
            if ((index_of_largest == -1) || (numbers[i] > numbers[index_of_largest])) {
                index_of_largest = i;
            }
        }

        /*Find the index of the second largest number*/
        int index_of_second_largest = -1;
        for (auto j = 0; j < n; ++j) {
            /** Bug in the example solution: multiple largest numbers
            * Instead of comparing the value, only need to skip the index_of_largest
            */
            if ((j != index_of_largest) && ((index_of_second_largest == -1) || (numbers[j] > numbers[index_of_second_largest]))) {
                index_of_second_largest = j;
            }
        }

        return static_cast<long long>(numbers[index_of_largest]) * numbers[index_of_second_largest];
    }
    ```

- simpler solution:

    ```c++
    /** Fast Solution
      * Time Complexity: O(n)
      */
    long long MaxPairwiseProductFast(const std::vector<int>& numbers) {
        auto n = numbers.size();
        int largest = 0, secondLargest = 0;
        for (auto i = 0; i < n; ++i) {
            if (numbers[i] > largest) {
                secondLargest = largest;
                largest = numbers[i];
            }
            else if (numbers[i] > secondLargest) {
                secondLargest = numbers[i];
            }
        }

        return static_cast<long long>(largest) * secondLargest;
    }
    ```

- `g++` compiler flag `-pipe`
    - reference: [g++(1) - Linux manual page (man7.org)](https://www.man7.org/linux/man-pages/man1/g++.
    html)
    -  use pipes rather than temporary files for communication between the various stages of compilation  
    (some assembler is unable to read from a pipe).
- `g++` compiler flag `-O2`
    - optimization level: Optimize even more than `-O1`.
- stress test
    - definition: a program that generates random tests in an `infinite` loop. For each test case, it   launches the solution on this test and an alternative solution on the same test and compares the    results.
    - Question: naïve solution can't give answer to some large inputs...
    - test cases should be covered
        - big numbers
        - small numbers
        - large amount of input data
        - small amount of input data
        - different distribution in input data
        - .etc
    - boundary values (e.x. length)
        - minimum value
        - maximum value
    - degenerated cases
---

## Week 2

### Takeaway
- how to abstract and formulate a practical problem into an algorithmic question?

### Fibonacci Numbers
- Naïve Solution
```c++
int fibonacci_naive(int n) {
    if (n <= 1)
        return n;

    return fibonacci_naive(n - 1) + fibonacci_naive(n - 2);
}
```
- duplicate calculation of certain `fibonacci` numbers
    - this is a common issue in recursion algorithms
    - the sub-problems should be independent

- Fast Solution: iteration

```c++
/** Fast solution to calculate fibonacci numbers
  * Time Complexity: O(logn)
  */
long long fibonacci_fast(int n) {
    long long first = 0, second = 1;
    for (int i = 0; i < n; ++i) {
        // Python: first, second = second, first + second
        long long tmp = second;
        second += first;
        first = tmp;
    }

    // Defintion of fibonacci number starts from 0
    // 0, 1, 1, 2, 3, ...
    return first;
}
```

- Fast Solution: memorization

```c++
#include <unordered_map>
long long fibonacci_memorized(int n, std::unordered_map<int, long long>& records) {
    if (records.find(n) != records.end()) {
        return records[n];
    }

    long long fibNum = fibonacci_memorized(n - 1, records) + fibonacci_memorized(n - 2, records);
    return (records[n] = fibNum);
}

long long fibonacci_fast(int n) {

    std::unordered_map<int, long long> memorizedFibonacciNums; 
    memorizedFibonacciNums[0] = 0;
    memorizedFibonacciNums[1] = 1;

    return fibonacci_memorized(n, memorizedFibonacciNums);
}
```
- In `Python`, there is a specific `decorator` which can be used to `automatically` memorize the previous result without manually creating this has map.

- Fast Solution: array based

```c++
long long fibonacci_fast_storage(int n, long long* records) {
    for (int i = 2; i <= n; ++i) {
        records[i] = records[i - 1] + records[i - 2];
    }

    return records[n];
}

long long fibonacci_fast(int n) {
    // fibonacci number starts from 0 and there are [0, ..., n], n + 1 in total
    long long* records = new long long[n + 1];
    records[0] = 0;
    records[1] = 1;
    long long result = fibonacci_fast_storage(n, records);
    delete[] records;

    return result;
}
```

- Visualization of different implementation: [Dynamic Programming (Fibonacci) (usfca.edu)](https://www.cs.usfca.edu/~galles/visualization/DPFib.html)

### Greatest Common Divisor
- Naïve Solution
```c++
int gcd_naive(int a, int b) {
    int current_gcd = 1;
    for (int d = 2; d <= a && d <= b; d++) {
      if (a % d == 0 && b % d == 0) {
        if (d > current_gcd) {
          current_gcd = d;
        }
      }
    }
    return current_gcd;
}
```

- Euclidean Algorithm
```c++
/** Proof: gcd(a, b) = gcd(a%b, b) = gcd(b, a%b)
  * Euclidean Algorithm
  */
int gcd_fast(int a, int b) {
    if (b == 0)
      return a;

    return gcd_fast(b, a % b);
}
```
-TODO: Binary GCD Algorithm (modulus is inefficient)

### Asymptotic Analysis 
```c++
for i from 2 to n:
	F[i] = F[i - 1] + F[i - 2]
```
- code above is part of the fast Fibonacci algorithm
- when adding two big numbers, it can be time-consuming which turns into the bottleneck of the algorithm. 
- running time is proportional to the number of digits, which is further proportional to the value itself: `O(n)`

### Last Digit of Fibonacci Number
```c++
#include <iostream>
#include <vector>
#include <cassert>

#define SUBMIT
// #undef SUBMIT

int get_fibonacci_last_digit_naive(int n) {
    if (n <= 1)
        return n;

    int previous = 0;
    int current  = 1;

    for (int i = 0; i < n - 1; ++i) {
        int tmp_previous = previous;
        previous = current;
        current = tmp_previous + current;
    }

    return current % 10;
}

// Time Complexity: O(n)
int get_fibonacci_last_digit_fast(int n) {
    std::vector<int> fib(n + 1);
    fib[0] = 0;
    fib[1] = 1;
    for (auto i = 2; i <= n; ++i) {
        fib[i] = (fib[i - 1] % 10 + fib[i - 2] % 10) % 10;
    }

    return fib[n];
}


void test_solution() {
    // cover given test points
    assert(get_fibonacci_last_digit_fast(3) == 2);
    assert(get_fibonacci_last_digit_fast(139) == 1);
    assert(get_fibonacci_last_digit_fast(91239) == 6);

    // for correctness concern
    for (auto i = 0; i <= 30; ++i)
        assert(get_fibonacci_last_digit_fast(i) == get_fibonacci_last_digit_naive(i));
    
    std::cout << "Test: Pass" << "\n";
} 


int main() {
#ifdef SUBMIT
    int n;
    std::cin >> n;
    // int c = get_fibonacci_last_digit_naive(n);
    int c = get_fibonacci_last_digit_fast(n);
    std::cout << c << '\n';
#else
    test_solution();
#endif // SUBMIT
}
```
- understand the basic of modulus arithmetic

### Huge Fibonacci Number Modulus
```c++
#include <iostream>
#include <cassert>
#include <vector>

using ll = long long;

#define SUBMIT
// #undef SUBMIT
// #define FAST_IMPLEMENTATION_ONE
#define FAST_IMPLEMENTATION_TWO

long long get_fibonacci_huge_naive(long long n, long long m) {
    if (n <= 1)
        return n;

    long long previous = 0;
    long long current  = 1;

    for (long long i = 0; i < n - 1; ++i) {
        long long tmp_previous = previous;
        previous = current;
        current = tmp_previous + current;
    }

    return current % m;
}


#ifdef FAST_IMPLEMENTATION_ONE
long long get_fibonacci_huge_fast(long long n, long long m) {
    assert(n >= 1 && n <= 10E14);
    assert(m >= 2 && m <= 10E3);

    if (n <= 1)
        return n;
    
    std::vector<long long> fib(3);
    fib[0] = 0;
    fib[1] = 1;
    for (long long i = 2; i <= n; ++i) {
        fib[2] = (fib[1] % m + fib[0] % m) % m;
        fib[0] = fib[1];
        fib[1] = fib[2];
    }

    return fib[2];
}
#endif // FAST_IMPLEMENTATION_ONE


#ifdef FAST_IMPLEMENTATION_TWO
ll get_fibonacci_huge_fast(ll n, ll m) {
    assert(n >= 1 && n <= 10E14);
    assert(m >= 2 && m <= 10E3);

    // Brute force: store the pattern sequence
    const int patternCount = 10000;
    std::vector<ll> pattern(patternCount);
    pattern[0] = 0;
    pattern[1] = 1;
    for (auto i = 2; i < patternCount; ++i) 
        pattern[i] = (pattern[i - 1] % m + pattern[i - 2] % m) % m;

    // Brute force: find the pattern
    auto first = pattern[0];
    auto period = 0;
    for (auto i = 1; i < patternCount; ++i) {
        if (pattern[i] == first) {
            bool isFind = true;

            // Check same patterns occur in [1, i) and [i + 1, 2i)
            for (auto j = 1; j < i; ++j) {
                if (pattern[j] != pattern[i + j]) {
                    isFind = false;
                    break;
                }
            }
            if (isFind) {
                period = i;
                break;
            }
        }
    }

    return pattern[n % period];
}
#endif // FAST_IMPLEMENTATION_TWO



void test_solution() {
    // cover all given test points
    assert(get_fibonacci_huge_fast(1, 239) == 1);
    assert(get_fibonacci_huge_fast(115, 1000) == 885);
    assert(get_fibonacci_huge_fast(2816213588, 239) == 151);

    // trust naive implementation :)
    for (auto i = 1; i <= 20; ++i)
        for (auto j = 2; j <= 100; ++j)
            assert(get_fibonacci_huge_fast(i, j) == get_fibonacci_huge_naive(i, j));

    std::cout << "Test: Pass" << '\n';
}


int main() {
#ifdef SUBMIT
    long long n, m;
    std::cin >> n >> m;
    // std::cout << get_fibonacci_huge_naive(n, m) << '\n';
    std::cout << get_fibonacci_huge_fast(n, m) << '\n';
#else
    test_solution();
#endif // SUBMIT
    return 0;
}

```

### Last Digit of the Sum of Fibonacci Numbers

```c++
#include <iostream>
#include <vector>
#include <cassert>

#define SUBMIT
// #undef SUBMIT

using ll = long long;


int fibonacci_sum_naive(long long n) {
    if (n <= 1)
        return n;

    long long previous = 0;
    long long current  = 1;
    long long sum      = 1;

    for (long long i = 0; i < n - 1; ++i) {
        long long tmp_previous = previous;
        previous = current;
        current = tmp_previous + current;
        sum += current;
    }

    return sum % 10;
}


int fibonacci_sum_fast(ll n) {
    assert(n >= 0 && n <= 10E14);

    // Last digit of fibonacci numbers is periodic: T = 60
    std::vector<int> fibLastDigits(60);
    fibLastDigits[0] = 0;
    fibLastDigits[1] = 1;
    for (auto i = 2; i < 60; ++i)
        fibLastDigits[i] = (fibLastDigits[i - 1] % 10 + fibLastDigits[i - 2] % 10) % 10;

    auto sumOfPeriods = 0, sum = 0;
    auto numOfPeriods = n / 60;
    if (numOfPeriods) {
        // Calculate the sum of all last digits
        for (auto i : fibLastDigits)
            sumOfPeriods += i;
    }
    auto remainder = n % 60;
    for (auto i = 0; i <= remainder; ++i)
        sum += fibLastDigits[i];

    return (numOfPeriods * sumOfPeriods  +sum) % 10;
}


void test_solution() {
    // Cover all given test points
    assert(fibonacci_sum_fast(3) == 4);
    assert(fibonacci_sum_fast(100) == 5);

    // Trust naive algorithm
    for (auto i = 0; i < 30; ++i)
        assert(fibonacci_sum_fast(i) == fibonacci_sum_naive(i));
    
    std::cout << "Test: Pass" << '\n';
}


int main() {
#ifdef SUBMIT
    long long n = 0;
    std::cin >> n;
    // std::cout << fibonacci_sum_naive(n);
    std::cout << fibonacci_sum_fast(n) << '\n';
#else
    test_solution();
#endif // SUBMIT
}

```

### Last Digit of the Partial Sum of Fibonacci Numbers

```c++
#include <iostream>
#include <vector>
#include <cassert>

#define SUBMIT
// #undef SUBMIT

using ll = long long;

long long get_fibonacci_partial_sum_naive(long long from, long long to) {
    long long sum = 0;

    long long current = 0;
    long long next  = 1;

    for (long long i = 0; i <= to; ++i) {
        if (i >= from) {
            sum += current;
        }

        long long new_current = next;
        next = next + current;
        current = new_current;
    }

    return sum % 10;
}


int fibonacci_sum_fast(ll n) {

    // Last digit of fibonacci numbers is periodic: T = 60
    std::vector<int> fibLastDigits(60);
    fibLastDigits[0] = 0;
    fibLastDigits[1] = 1;
    for (auto i = 2; i < 60; ++i)
        fibLastDigits[i] = (fibLastDigits[i - 1] % 10 + fibLastDigits[i - 2] % 10) % 10;

    auto sumOfPeriods = 0, sum = 0;
    auto numOfPeriods = n / 60;
    if (numOfPeriods) {
        // Calculate the sum of all last digits
        for (auto i : fibLastDigits)
            sumOfPeriods += i;
    }
    auto remainder = n % 60;
    for (auto i = 0; i <= remainder; ++i)
        sum += fibLastDigits[i];

    return (numOfPeriods * sumOfPeriods  +sum) % 10;
}



ll get_fibonacci_partial_sum_fast(ll from, ll to) {
    // Best practice: split complicated conditions
    assert(from >= 0);
    assert(to <= 10E14);
    assert(from <= to);

    // avoid including libraries since we only need one abs function...
    // return abs(fibonacci_sum_fast(to) - fibonacci_sum_fast(from - 1));
    ll result = fibonacci_sum_fast(to) - fibonacci_sum_fast(from - 1);

    return result > 0 ? result : -result;
}


void test_solution() {
    // Cover given test points
    assert(get_fibonacci_partial_sum_fast(3, 7) == 1);
    assert(get_fibonacci_partial_sum_fast(10, 10) == 5);

    // Trust naive algorithm
    for (auto i = 0; i < 0; ++i)
        for (auto j = 1; j < 30; ++j)
            assert(get_fibonacci_partial_sum_fast(i, j) == get_fibonacci_partial_sum_naive(i, j));
    
    std::cout << "Test: Pass" << '\n';
}


int main() {
#ifdef SUBMIT
    long long from, to;
    std::cin >> from >> to;
    // std::cout << get_fibonacci_partial_sum_naive(from, to) << '\n';
    std::cout << get_fibonacci_partial_sum_fast(from, to) << '\n';
#else
    test_solution();
#endif // SUBMIT
}

```

### Last Digit of the Sum of Squares of Fibonacci Numbers

```c++
#include <iostream>
#include <vector>
#include <cassert>

#define SUBMIT
// #undef SUBMIT

using ll = long long;


int fibonacci_sum_squares_naive(long long n) {
    if (n <= 1)
        return n;

    long long previous = 0;
    long long current  = 1;
    long long sum      = 1;

    for (long long i = 0; i < n - 1; ++i) {
        long long tmp_previous = previous;
        previous = current;
        current = tmp_previous + current;
        sum += current * current;
    }

    return sum % 10;
}


int fibonacci_sum_squares_fast(ll n) {
    assert(n >= 0);
    assert(n <= 10E14);
	// last digit of fibonacci numbers is periodic: T = 60
    std::vector<int> fibLastDigits(60);
    fibLastDigits[0] = 0;
    fibLastDigits[1] = 1;
    for (auto i = 2; i < 60; ++i)
        fibLastDigits[i] = (fibLastDigits[i - 1] % 10 + fibLastDigits[i - 2] % 10) % 10;

    return (fibLastDigits[n % 60] * fibLastDigits[(n + 1) % 60]) % 10;
}

void test_solution() {
    // Cover all given test points
    assert(fibonacci_sum_squares_fast(7) == 3);
    assert(fibonacci_sum_squares_fast(73) == 1);
    assert(fibonacci_sum_squares_fast(1234567890) == 0);

    // Trust naive algorithm
    for (auto i = 0; i < 30; ++i)
        assert(fibonacci_sum_squares_fast(i) == fibonacci_sum_squares_naive(i));

    std::cout << "Test: Pass" << '\n';
}


int main() {
#ifdef SUBMIT
    long long n = 0;
    std::cin >> n;
    // std::cout << fibonacci_sum_squares_naive(n);
    std::cout << fibonacci_sum_squares_fast(n) << '\n';
#else
    test_solution();
#endif // SUBMIT
}

```

### Great Common Divisor
```c++
#include <iostream>
#include <cassert>

#define SUBMIT
// #undef SUBMIT

int gcd_naive(int a, int b) {
    int current_gcd = 1;
    for (int d = 2; d <= a && d <= b; d++) {
      if (a % d == 0 && b % d == 0) {
        if (d > current_gcd) {
          current_gcd = d;
        }
      }
    }
    return current_gcd;
}


/** Mathmatical Foundation: gcd(a, b) = gcd(a%b, b) = gcd(b, a%b)
  * Euclidean Algorithm
  */
int gcd_fast(int a, int b) {
    if (b == 0)
      return a;

    return gcd_fast(b, a % b);
}


void test_solution() {
    assert(gcd_fast(1, 1) == 1);
    assert(gcd_fast(2, 256) == 2);
    assert(gcd_fast(17, 19) == 1);
    assert(gcd_fast(3918848, 1653264) == 61232);

    for (int i = 1; i <= 20; i += 2) {
        for (int j = 2; j <= 30; j += 3) {
            assert(gcd_fast(i, j) == gcd_naive(i, j));
        }
    }

    std::cout << "Test: Pass" << "\n";
}


int main() {
#ifdef SUBMIT
    int a, b;
    std::cin >> a >> b;
    // std::cout << gcd_naive(a, b) << std::endl;
    std::cout << gcd_fast(a, b) << std::endl;
#else
    test_solution();
#endif //SUBMIT
    return 0;
}

```

### Least Common Multiple
```c++
#include <iostream>
#include <cassert>
#include <ctime>
#include <cstdlib>

#define SUBMIT
// #undef SUBMIT

using ll = long long;

ll lcm_naive(int a, int b) {
    for (long l = 1; l <= (ll) a * b; ++l)
        if (l % a == 0 && l % b == 0)
            return l;

    return (ll) a * b;
}


// Euclidean Algorithm
int gcd(int a, int b) {
    if (b == 0) {
      return a;
    }

    return gcd(b, a %= b);
}

ll lcm(int a, int b) {
    // Corner cases
    if (a == 0 || b == 0) {
        return 0;
    }

    if (a % b == 0) {
        return a;
    }

    if (b % a == 0) {
        return b;
    }

    // Calculate the gcd of a and b
    int gcdOfAB = gcd(a, b);
    // mulOfA and mulOfB must be Coprime
    int mulOfA = a / gcdOfAB, mulOfB = b / gcdOfAB;

    return (ll) gcdOfAB * mulOfA * mulOfB;
}


void test_solution() {
    // Randomizing...
    srand(time(nullptr));

    assert(lcm_naive(10, 15) == 30);
    assert(lcm_naive(144, 3) == 144);
    assert(lcm_naive(1, 1) == lcm(1, 1));
    assert(lcm_naive(2, 8) == lcm(2, 8));
    for (int i = 0; i <= 20; ++i) {
        int a = rand() % 1000 + 1;
        int b = rand() % 1000 + 1;
        assert(lcm_naive(a, b) == lcm(a, b));
    }
}


int main() {
#ifdef SUBMIT
    int a, b;
    std::cin >> a >> b;
    // std::cout << lcm_naive(a, b) << std::endl;
    std::cout << lcm(a, b) << std::endl;
#else 
    test_solution();
#endif // SUBMIT
    return 0;
}
```