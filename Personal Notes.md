## Copying and Copy Constructors
- Issue with default copy constructor is that if there is a pointer, the address is copied over, not a copy of the underlying object.
```c++
String(const String& other) {
  ..................
}
```

## Lvalue and Rvalue in C++ 
- Every expression in C++ is either Lvalue or Rvalue expression. 
- **Lvalue:** If you can take address of expression then it is Lvalue, and they last extended period of time.
- **Rvalue:** Can't take address and they are temporary, they don't exist after one line 

```c++
int x = 10; // x is lvalue, 10 is r value

int a = 10, b = 20;
int x = (a + b); // x is lvalue, (a + b) is rvalue
int* p = &(a + b); // ERROR: (a + b) is not lvalue

class Cat {};
Cat c = Cat(); // Cat() is an rvalue 

int square(int x) { return x*x; }
int sq = square(10); // square(10) is an rvalue
```
- **Lvalue Reference:** an refer to rvalue if it is const (i.e. const int& ref = 10). When passing an rvalue to function and parameter is expecting an lvalue reference, make the parameter const reference to allow rvalues to be sent in (i.e. void setValueTo99(const int& param)).
- **Rvalue Reference:** Mainly used for moving semantics and perfect forwarding. Use && to create Rvalue reference (i.e. int&& ref = 10). Reason we might want to use is don't want to be continually copying, and instead want to just move something that is already created somewhere.
- **Move Constructor:** Creates an inexpensive shallow copy of rvalue argument. In constrast to copy constructor which is making expensive deep copy.
```c++
Vector(const Vector& v) { // Copy Constructor (called by foo(reusable))
  *perform deep copy* 
}

Vector(Vector&& v) { // Move Constructor (called by foo(createVector())
  *move contents from v to this*
}
```
- Assignment operator (copy, move) only called when doing assignment with prexisting variable. Doing something like String s2 = s1 will call copy constructor because new String object being created.
- **std::move:** It changes an expression from being an lvalue (such as a named variable) to being an xvalue. An xvalue tells the compiler: "You can plunder me, move anything I'm holding and use it elsewhere (since I'm going to be destroyed soon anyway)". Really just taking variable and need to turn it into a temporary, so you can use move constructor/operator to take the resources from that variable. 
- **std::forward:** According to the T template parameter, std::forward identifies whether an lvalue or an rvalue reference has been passed to it and returns a corresponding kind of reference.

## Inheritance + Virtual
- Ambiguity can arise when you have one derived class inheriting from two base classes that have the same function, and the derived class itself doesn't override that function. Need to specify the parent class to class function from (derived.BaseClass1::function1()). Ambiguity can also occur with variables. 
- Ambiguity can arise in diamond problem where most Base class has a variable, and the variable is found in multiple inherited parents. Solve this using the virtual keyword during the inherit portion for both base classes (class BaseClass1 : virtual public CommonBaseClass). 
- Call super constructor in member initialization (BaseClass() : CommonBaseClass(100) {});
- Use virtual functions to achieve dynamic polymorphism (dynamic dispatch or late method binding) which gives the ability to call derived class function using a base class pointer or reference.
- Virtual keyword for destructor is necessary when you want different destructors to follow proper order while objects is being deleted through base class pointer.
- For every class that contains virtual functions, the compiler constructs a virtual table, a.k.a vtable. The vtable contains an entry for each virtual function accessible by the class and stores a pointer to its definition. Only the most specific function definition callable by the class is stored in the vtable. Entries in the vtable can point to either functions declared in the class itself (e.g. C::bar()), or virtual functions inherited from a base class (e.g. C::qux()).
- You might be thinking: vtables are cool and all, but how exactly do they solve the problem? When the compiler sees b->bar() in the example above, it will lookup B’s vtable for bar’s entry and follow the corresponding function pointer, right? We would still be calling B::bar() and not C::bar()… Very true, I still need to tell the second part of the story: vpointers. Every time the compiler creates a vtable for a class, it adds an extra argument to it: a pointer to the corresponding virtual table, called the vpointer. Note that the vpointer is just another class member added by the compiler and increases the size of every object that has a vtable by sizeof(vpointer). Hopefully you have grasped how dynamic function dispatch can be implemented by using vtables: when a call to a virtual function on an object is performed, the vpointer of the object is used to find the corresponding vtable of the class. Next, the function name is used as index to the vtable to find the correct (most specific) routine to be executed. Done!
- By now it should also be clear why it is always a good idea to make destructors of base classes virtual. Since derived classes are often handled via base class references, declaring a non-virtual destructor will be dispatched statically, obfuscating the destructor of the derived class (it is never called).

## RAII 
- "RAII" stands for "Resource Acquisition is Initialization" and is actually quite a misnomer, since it isn't resource acquisition (and the initialization of an object) it is concerned with, but releasing the resource (by means of destruction of an object).
- At its very heart, the idiom features encapsulating resources (chunks of memory, open files, unlocked mutexes, you-name-it) in local, automatic objects, and having the destructor of that object releasing the resource when the object is destroyed at the end of the scope it belongs to:
- 

## Vector Allocation
- Stack allocated vector will allocate the vector, i.e. the header info, on the stack, but the elements on the free store ("heap"). Created a vector on the heap will allocate everything on the free store.

## Rule of 3/5/0
### Rule of 3
- If a class requires a user-defined destructor, a user-defined copy constructor, or a user-defined copy assignment operator, it almost certainly requires all three. Because C++ copies and copy-assigns objects of user-defined types in various situations (passing/returning by value, manipulating a container, etc), these special member functions will be called, if accessible, and if they are not user-defined, they are implicitly-defined by the compiler.
```c++
#include <iostream>
#include <cstddef>
#include <cstring>
 
class rule_of_three
{
    char* cstring; // raw pointer used as a handle to a dynamically-allocated memory block
 
    rule_of_three(const char* s, std::size_t n) // to avoid counting twice
    : cstring(new char[n]) // allocate
    {
        std::memcpy(cstring, s, n); // populate
    }
public:
    rule_of_three(const char* s = "")
    : rule_of_three(s, std::strlen(s) + 1) {}
 
    ~rule_of_three() // I. destructor
    {
        delete[] cstring; // deallocate
    }
 
    rule_of_three(const rule_of_three& other) // II. copy constructor
    : rule_of_three(other.cstring) {}
 
    rule_of_three& operator=(const rule_of_three& other) // III. copy assignment
    {
        if (this == &other)
            return *this;
 
        std::size_t n{std::strlen(other.cstring) + 1};
        char* new_cstring = new char[n];            // allocate
        std::memcpy(new_cstring, other.cstring, n); // populate
        delete[] cstring;                           // deallocate
 
        cstring = new_cstring;
        return *this;
    }
 
    operator const char *() const // accessor
    {
        return cstring;
    }
};
 
int main()
{
    rule_of_three o1{"abc"};
    std::cout << o1 << ' ';
    auto o2{ o1 }; // I. uses copy constructor
    std::cout << o2 << ' ';
    rule_of_three o3("def");
    std::cout << o3 << ' ';
    o3 = o2; // III. uses copy assignment
    std::cout << o3 << ' ';
}   // <- II. all destructors are called 'here'
```
- Classes that manage non-copyable resources through copyable handles may have to declare copy assignment and copy constructor private and not provide their definitions or define them as deleted. This is another application of the rule of three: deleting one and leaving the other to be implicitly-defined will most likely result in errors.
### Rule of 5
- Because the presence of a user-defined (or = default or = delete declared) destructor, copy-constructor, or copy-assignment operator prevents implicit definition of the move constructor and the move assignment operator, any class for which move semantics are desirable, has to declare all five special member functions:
```c++
class rule_of_five
{
    char* cstring; // raw pointer used as a handle to a dynamically-allocated memory block
public:
    rule_of_five(const char* s = "") : cstring(nullptr)
    { 
        if (s)
        {
            std::size_t n = std::strlen(s) + 1;
            cstring = new char[n];      // allocate
            std::memcpy(cstring, s, n); // populate 
        } 
    }
 
    ~rule_of_five()
    {
        delete[] cstring; // deallocate
    }
 
    rule_of_five(const rule_of_five& other) // copy constructor
    : rule_of_five(other.cstring) {}
 
    rule_of_five(rule_of_five&& other) noexcept // move constructor
    : cstring(std::exchange(other.cstring, nullptr)) {}
 
    rule_of_five& operator=(const rule_of_five& other) // copy assignment
    {
        return *this = rule_of_five(other);
    }
 
    rule_of_five& operator=(rule_of_five&& other) noexcept // move assignment
    {
        std::swap(cstring, other.cstring);
        return *this;
    }
 
// alternatively, replace both assignment operators with 
//  rule_of_five& operator=(rule_of_five other) noexcept
//  {
//      std::swap(cstring, other.cstring);
//      return *this;
//  }
};
```
- Unlike Rule of Three, failing to provide move constructor and move assignment is usually not an error, but a missed optimization opportunity.
### Rule of 0
- Classes that have custom destructors, copy/move constructors or copy/move assignment operators should deal exclusively with ownership (which follows from the Single Responsibility Principle). Other classes should not have custom destructors, copy/move constructors or copy/move assignment operators.
```c++
class rule_of_zero
{
    std::string cppstring;
public:
    rule_of_zero(const std::string& arg) : cppstring(arg) {}
};
```
- When a base class is intended for polymorphic use, its destructor may have to be declared public and virtual. This blocks implicit moves (and deprecates implicit copies), and so the special member functions have to be declared as defaulted.
```c++
class base_of_five_defaults
{
public:
    base_of_five_defaults(const base_of_five_defaults&) = default;
    base_of_five_defaults(base_of_five_defaults&&) = default;
    base_of_five_defaults& operator=(const base_of_five_defaults&) = default;
    base_of_five_defaults& operator=(base_of_five_defaults&&) = default;
    virtual ~base_of_five_defaults() = default;
};
```
- However, this makes the class prone to slicing, which is why polymorphic classes often define copy as deleted (see C.67: A polymorphic class should suppress copying in C++ Core Guidelines), which leads to the following generic wording for the Rule of Five: C.21: If you define or =delete any default operation, define or =delete them all










