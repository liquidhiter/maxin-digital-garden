---
{"dg-publish":true,"permalink":"/12-c/cs-106-l/course-notes/","noteIcon":"","created":"2024-12-22T23:52:58.483+01:00","updated":"2024-12-24T01:49:04.313+01:00"}
---

- [ ] future courses might follow
![Z - assets/images/Pasted image 20241222235319.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241222235319.png)
- [ ] art of asking questions...
![Z - assets/images/Pasted image 20241222235508.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241222235508.png)

### understanding
- overload the **operator<<**
```c++
struct Student {
public:
    std::string name;
    int age;
    int id;

public:
    /**
     * Overload the << operator
     * friend because the function is a non-member function
     * first parameter of the member function is the object that called the function, i.e. Student (this) object
     *
     */
    friend std::ostream& operator<<(std::ostream& os, const Student& student) {
        os << "Name: " << student.name << ", Age: " << student.age << ", ID: " << student.id;
        return os;
    }

    /**
     * Overload the + operator
     */
    Student operator+(const Student& other) const{
        return Student {name + other.name, age + other.age, id + other.id};
    }
};
```
> for binary operator like <<, the left operand determines which class's member function to call
> non-member function because the left operand is the std::ostream& which is defined in the standard library
> the first parameter of a member function must be an instance of the class
> underlying calls `std::ostream& operator<<(std::ostream&, const char*);`

**More explanation**:
[[ChatGPT/operator overloading and non-member function\|operator overloading and non-member function]]

- `std::pair`
	- template
```c++
template <typename T1, typename T2>
struct pair {
	T1 first;
	T2 seconds;
};
```

- `[[nodiscard]]`
	- new feature from C++17
	- compilation time warnings of ignoring the return value of certain functions

- `clang-tidy`
	- definition of function in aa header file should have an `inline` specifier
	- WHY?
		- function definition in a header file can be included in multiple compilation units
			- functions are copied across different units, which results in multiple definition errors during linking stage
	- HOW?
		- `inline` permits multiple identical definitions across translation units by allowing compiler to merge identical definitions during linking
	- MORE
		- *template functions are already implicitly `inline*`
**More explanation**
[[ChatGPT/function definition in header file should have inline\|function definition in header file should have inline]]

- solution of quadratic expression solver
```c++
/* ************************************************************************
> File Name:     quadratic_equation_solver.cpp
> Author:        maxin
> mail:          lyingflatlearner@gmail.com
> Created Time:  2024/12/23 10:22:57
> Description:
 ************************************************************************/
#pragma once

/* Standard C++ headers */
#include <utility> /* std::pair */
#include <cmath>

class QuadraticEquationSolver {
private:
    /* Coefficients of the quadratic equation */
    double a, b, c;
public:
    /**
     * Constructor
     */
    QuadraticEquationSolver(const double a, const double b, const double c) : a(a), b(b), c(c) {}
    std::pair<bool, std::pair<double, double>> solveQuadratic() const; // NOLINT(*-use-nodiscard)
private:
    /**
     * Calculate the discriminant of the quadratic equation
     */
    [[nodiscard]]double calculateDiscriminant() const {
        return b * b - 4 * a * c;
    }

    /**
     * Check if the quadratic equation has real roots
     */
    [[nodiscard]]bool hasRealRoots() const {
        return calculateDiscriminant() >= 0;
    }

    /**
     * Calculate the roots of the quadratic equation
     */
    [[nodiscard]] std::pair<double, double> calculateRoots() const {
        const double d = calculateDiscriminant();
        double x1 = (-b + std::sqrt(d)) / (2 * a);
        double x2 = (-b - std::sqrt(d)) / (2 * a);
        return {x1, x2};
    }
};

/**
 * Implement the solveQuadratic function
 * inline keyword is used to avoid multiple definitions of the function
 * when the header file is included in multiple source files
 */
inline std::pair<bool, std::pair<double, double>> QuadraticEquationSolver::solveQuadratic() const {
    if (!hasRealRoots()) {
        return {false, {0, 0}};
    }
    return {true, calculateRoots()};
}
```
> example use
```c++
#include "quadratic_equation_solver.h"

static inline void printQuadraticEquation(double a, double b, double c) {
    std::cout << "Quadratic Equation: " << a << "x^2 + " << b << "x + " << c << std::endl;
}

static inline void printRoots(const std::pair<double, double>& roots) {
    std::cout << "Roots: " << roots.first << ", " << roots.second << std::endl;
}

static inline void printNoRoots() {
    std::cout << "No real roots" << std::endl;
}

static inline void printResults(const std::pair<bool, std::pair<double, double>>& results) {
    if (results.first) {
        printRoots(results.second);
    } else {
        printNoRoots();
    }
}

/*TODO: use of random library */
/* Mersenne Twister random number generator */
std::random_device rd;
std::mt19937 gen(rd());
/* Define a uniform distribution */
std::uniform_real_distribution<double> dist(0.0, 65536.0);

for (int i = 0; i < 1000; ++i) {
	double coeff_a = dist(gen);
	double coeff_b = dist(gen);
	double coeff_c = dist(gen);
	printQuadraticEquation(coeff_a, coeff_b, coeff_c);
	QuadraticEquationSolver solver(coeff_a, coeff_b, coeff_c);
	/**  
	 * `[hasRoots, roots]`: This is a structured binding declaration that unpacks the `std::pair<bool,
	 * std::pair<double, double>>` returned by `solveQuadratic()` into two separate variables: 
	 * - `hasRoots` will be of type `bool`.
	 * - `roots` will be of type `std::pair<double, double>`.
	 */
    auto [hasRoots, roots] = solver.solveQuadratic();  
	printResults({hasRoots, roots});

#if 0
	auto results = solver.solveQuadratic();  
	auto hasRoot = results.first;  
	auto solution = results.second;  
	printResults({hasRoot, solution});
#endif
}
```
- improvements
	- `auto`
		- `auto results = solver.solveQuadratic();`
		- `[hasRoots, roots]` structured bindings to unpack the pair
	- `using`
		- `using Zeros = std::pair<double, double>;  
		- `using Solution = std::pair<bool, Zeros>;`

- how to read a file efficiently using modern C++?
	- [ ] TODO (stream)

- initialization
	- preferred uniform initialization to avoid narrowing conversion
- structured binding
	- initialize some variables from data structures with fixed sizes at compile time
	- **can use on objects where the size is known at compile-time**
- references
	- passed by value
	- passed by reference
```c++
inline void shiftvector(std::vector<std::pair<int, int>> &nums) {
    /* auto [num1, num2], passed values */
#if 0
    for (auto [num1, num2] : nums) {
        num1++;
        num2++;
    }
#else
    for (auto & [num1, num2] : nums) {
        num1++;
        num2++;
    }
#endif
}
```
- structured bindings
	- passed by values by default

- l-value and r-value
- an r-value can be to the left or the right of an equal sign
	- `int y = x; x = 200;`
	- both `y` and `x` are l-values
- an r-value can only to the right of an equal sign
	- KEY: `r-values` are temporary
	- `cannot bind non-const lvalue reference of type 'int&' to an rvalue of type 'int'`
```c++
static int squareN(int& num) {
    return static_cast<int>(std::pow(num, 2));
}

inline void callSquareN() {
    int num = 10;
    int result = squareN(num);
    std::cout << "num: " << num << ", result: " << result << std::endl;

#if 1
    result = squareN(20);
    std::cout << "result: " << result << std::endl;
#endif
}
```
> const l-value <=> r-value
> semantically, they should be compatible
- `const`: a qualifier for objects that declares they cannot be modified
```c++
#include <vector>

inline void const_use() {
    std::vector<int> vec {1, 2, 3, 4, 5};
    const std::vector<int> const_vec {1, 2, 3, 4, 5};
    std::vector<int>& ref_vec {vec};
    const std::vector<int>& const_ref_vec {vec};

    vec.push_back(6);
    // const_vec.push_back(6);
    ref_vec.push_back(6);
    // const_ref_vec.push_back(6);

    /* non-const reference to a const object */
    // std::vector<int>& ref_const_vec {const_vec};
}
```
> never declare a non-const reference to a const one
> WHY? should not be able to modify (non-const) any variable which is not supposed to be modifiable (const)

- streams
	- a general input/output abstraction for C++
- `cerr` and `clog`
- `stringstream`
![Z - assets/images/Pasted image 20241223180400.png](/img/user/Z%20-%20assets/images/Pasted%20image%2020241223180400.png)
- `>>` operator only reads until the next whitespace
- `std::cout` is line buffered until explicitly flush
	- `std::flush`
	- `std::endly` also tells to flush
		- BUT, flushing is expensive
- `C++` is smart enough knowing when to auto flush
	- use `\n` instead of `std::endl`
		- `\n` doesn't flush by default unless the stream is **line-buffered** (like *std::cout* is tied to the terminal)
	- `std::ios::sync_with_stdio(false)`
		- massive performance boost
		- [reference](https://stackoverflow.com/questions/31162367/significance-of-ios-basesync-with-stdiofalse-cin-tienull)
```c++
/* ************************************************************************
> File Name:     stringstream.h
> Author:        maxin
> mail:          lyingflatlearner@gmail.com
> Created Time:  2024/12/23 17:55:14
> Description:
 ************************************************************************/
#ifndef STRINGSTREAM_H
#define STRINGSTREAM_H

#include <string>
#include <sstream>
#include <chrono>
#include <utility>

static void flush_performance(std::pair<bool, double>& test) {
    /* takes effect for the entire program */
    std::ios::sync_with_stdio(test.first);

    /* Performance measurement */
    auto total_elapsed = 0.0;
    constexpr size_t num_iterations = 100;
    for (size_t i = 0; i < num_iterations; ++i) {
        auto start = std::chrono::high_resolution_clock::now();
        for (int i = 1; i <= 5; ++i) {
            std::cout << i << "\n";
        }
        auto end = std::chrono::high_resolution_clock::now();
        std::chrono::duration<double> elapsed = end - start;
        total_elapsed += elapsed.count(); /* count() returns the duration in seconds */
    }
    auto average_elapsed = total_elapsed / num_iterations;
    // std::cout << "Average time per iteration: " << average_elapsed * 1E6 << "us" << std::endl;
    test.second = average_elapsed * 1E6;
}

static void print_test_results(const std::pair<bool, double>& test) {
    std::cout << "Synchronization: " << std::boolalpha << test.first << std::endl;
    std::cout << "Average time per iteration: " << test.second << "us" << std::endl;
}

inline void string_stream() {
    std::string initial_quote = "Bjarne Stroustrup C makes it easy to shoot yourself in the foot";

    /// Create a string stream object
    std::stringstream ss{initial_quote};

    /// Data destinations
    std::string first;
    std::string last;
    std::string language, extracted_quote;
    ss >> first >> last >> language;
    std::cout << first << " " << last << " " << language << std::endl;
    std::getline(ss, extracted_quote);
    std::cout << extracted_quote << std::endl;

    /* std::flush: flushes the output buffer of the stream */
    std::cout << "Hello, CPP";

    /* std::endl is equivalent to '\n' + std::flush */
    std::cout << std::flush << " [flushed]";
    std::cout << "Hello, CPP" << std::endl << " [flushed]";
    std::cout << std::endl;

    for (int i = 1; i <= 5; ++i) {
        std::cout << i << std::endl;
    }

    /* Flush the output buffer before running the performance comparison */
    std::cout << std::flush;

    /* Test the performance of std::cout with and without synchronization */
    std::vector<std::pair<bool, double>> test_sets {{true, 0.0}, {false, 0.0}};
    for (auto& test : test_sets) {
        flush_performance(test);
    }

    /* Print the test results */
    for (const auto& test : test_sets) {
        print_test_results(test);
    }
}

#endif //STRINGSTREAM_H
```

- `ofstream`
	- different mode, e.x. `std::ios::app`
```c++
static std::string out_file_path() {
    const std::string file_name = "hello.txt";
    std::filesystem::path curr_folder = std::filesystem::current_path();
    std::string file_path = curr_folder.string() + "/../" + file_name;
    return file_path;
}

/**
 *
 */
inline void outstream() {
    std::string file_path = out_file_path();
    /* Delete the file if it exists */
    if (std::filesystem::exists(file_path)) {
        std::filesystem::remove(file_path);
    }
    std::ofstream ofs{file_path};
    if (ofs.is_open()) {
        ofs << "Hello CS106L" << '\n';
    }
    ofs.close();
    ofs << "This will not be written to the file" << '\n';
    ofs.open(file_path);
    ofs << "This will overwrite the previous content" << '\n';
    ofs.close();

    /* Append to the file */
    ofs.open(file_path, std::ios::app);
    ofs << "This will be appended to the file" << '\n';
    ofs.close();
}
```

- `fstream`
	- interface for both reading and writing
		- combination of `ifstream` and `ostream`
```c++
/**
 * std::fstream: A stream class to both read and write from/to files
 * ifstream & ostream, usage is similar to the above functions
 */
inline void fstream() {
    /* Reuse the same out_file_path() function */
    std::string file_path = out_file_path();
    std::string poem = R"(When icicles hang by the wall
And Dick the shepherd blows his nail
And Tom bears logs into the hall,
And milk comes frozen home in pail,
When Blood is nipped and ways be foul,
Then nightly sings the staring owl,
Tu-who;
Tu-whit, tu-who: a merry note,
While greasy Joan doth keel the pot.

When all aloud the wind doth blow,
And coughing drowns the parson's saw,
And birds sit brooding in the snow,
And Marian's nose looks red and raw
When roasted crabs hiss in the bowl,
Then nightly sings the staring owl,
Tu-who;
Tu-whit, tu-who: a merry note,
While greasy Joan doth keel the pot.)";
    /**
     * Read and write to a file
     * std::ios::in: Open for input operations
     * std::ios::out: Open for output operations
     * std::ios::app: Append mode
     *
     */
    std::fstream file {file_path, std::ios::in | std::ios::out};
    if (file.is_open()) {
        /* Write to the file */
        file << poem << std::endl;

        /* Move the file pointer to the top */
        file.seekg(0, std::ios::beg);
        /* Check if the file pointer is at the beginning */
        if (file.fail()) {
            std::clog << "Failed to move the file pointer to the beginning" << std::endl;
        }

        std::string line;
        while (getline(file, line)) {
            std::cout << "Read from file: " << line << std::endl;
        }

        /* Overwrite the first line */
        file.clear();
        file.seekp(0, std::ios::beg);
        if (file.fail()) {
            std::clog << "Failed to move the file pointer to the beginning" << std::endl;
        }
        file << "New content for the first line" << std::endl;

        /* Close the file stream */
        file.close();
    }
}
```
> std::ios::app forces all writes to occur at the end of the file, **ignoring the seek position set by `seekp()`**
> remember to clear the EOF and other error flags via `file.clear()` method
- `std::cin` is buffered which stops at a whitespace
```c++
/**
 * std::cin: Standard input stream
 */
inline void cin_stream() {
    double pi;
    double tao;
    std::string name;
    std::cin >> pi;
    /* Ignore the newline character */
    std::getline(std::cin, name);
    /* Read the name after consuming the newline character */
    std::getline(std::cin, name);
    std::cin >> tao;
    std::cout << "Pi: " << pi << ", Name: " << name << ", TAO: " << tao << std::endl;
}
```
> std::cin leaves the newline character in the buffer
> following std::getline() stops at the first whitespace which ONLY reads out the new line character left-over
> so, would better not use std::cin and std::getline together
> [WORKAROUND]
> -[x] consumes the left-over newline character in the buffer by first calling a dummy getline
> -[x] reads out the `real` input from the buffer

**More explanation**
[[ChatGPT/fstream\|fstream]]

### questions
- [ ] how function overloading works under hood? 
	- i.e. how does C++ support this?
- [ ] stream

