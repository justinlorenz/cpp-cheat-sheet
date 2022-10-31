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

