# $Cpp$

[TOC]

# Object Oriented Programming

## Characteristics

- **Encapsulation**
  - Access permission for classes
    - private
    - protected
    - public

- **Inheritance**

- **Polymorphism**
  - Static polymorphism: **Overload**  (Function overload & Operator overload)
  
  - Dynamic polymorphism: **Overriding**
    - Implement: VTables (*virtual tables*)

### *Q: non virtualfunction vs. virtual function*

```cpp
#include <iostream>

class Base {
public:
    void nonVirtualFunction() {
        std::cout << "This is the Base class non-virtual function." << std::endl;
    }

    virtual void virtualFunction() {
        std::cout << "This is the Base class virtual function." << std::endl;
    }
};

class Derived : public Base {
public:
    void nonVirtualFunction() { // This is a non-virtual function, not using 'virtual' keyword
        std::cout << "This is the Derived class non-virtual function." << std::endl;
    }

    void virtualFunction() override { // This is a virtual function, using 'override' keyword
        std::cout << "This is the Derived class virtual function." << std::endl;
    }
};

int main() {
    Base* basePtr = new Derived();
    basePtr->nonVirtualFunction(); // Calls the Base class non-virtual function at compile time
    basePtr->virtualFunction();    // Calls the Derived class virtual function at runtime

    delete basePtr;
    return 0;
}

```

Output result:

```cpp
This is the Base class non-virtual function.
This is the Derived class virtual function.
```

### *Q: virtual function vs. pure virtual function*

A pure virtual function is a virtual function in the base class that has no implementation and is meant to be overridden in the derived classes. 

- override 
  - virtual function: It is not necessary for all derived classes to override the virtual function of the base class
  - pure virtual function: Derived classes should override the pure virtual function of the base class
- abstract class
  - virtual function: There is no abstract class concept
  - pure virtual function: If there is at least one pure virtual function, that classis called an abstract class

### *Q: When inheriting, is the destructor of the parent class a virtual function? Can constructors be virtual functions? Why?*




## Class

### Default Functions

| Default Functions |      |
| ------------------- | ---- |
| Default Constructor |   ```MyClass()```   |
| Destructors |  ```~MyClass();```    |
| Copy constructors |  ```MyClass(const MyClass& other);```    |
| Copy assignment operators | ```MyClass& operator=(const MyClass& other)``` |
| Move constructors |  ```MyClass(MyClass&& other);``` |
|Move assignment operators|```MyClass& operator=(MyClass&& other);```|


#### *Q: Move constructors vs. Copy constructors?*

Move Constructor: is a special constructor used to efficiently transfer the resources (like memory ownership) of an existing object to a newly created object. And the source object is no longer needed afterward. 

### Class memory structure

- **virtual pointer (vptr)**
- **Data members**
- **Padding**

[What does C++ Object Layout Look Like? | Nimrod's Coding Lab](https://nimrod.blog/posts/what-does-cpp-object-layout-look-like/)

#### Single Inheritance

```cpp
class Base {
public:
    virtual ~Base();
    virtual void Foo();
    virtual void Bar();
    double b;
};

class Derived : public Base {
public:
    void Foo() override; // override
    virtual void Baz(); // new virtual function
    double d;
  	
};
```



<img src="./assets/base.png" alt="img" style="zoom:50%;" /><img src="./assets/derived.png" alt="img" style="zoom:50%;" />

#### Multiple Inheritance

```cpp
struct Base1 {
    virtual ~Base1();
    virtual void Foo();
  	double b1;
};

struct Base2 {
    virtual ~Base2();
    virtual void Bar();
  	double b2;
};

struct Derived: public Base1, public Base2 {
    ~Derived();
    void Foo() override;
    double d;
};
```

<img src="./assets/multiple_inheritance.png" alt="img" style="zoom:50%;" /><img src="./assets/vi_derived_complete.png" alt="img" style="zoom:40%;" />



#### Virtual Inheritance

```cpp
struct VBase {
    virtual void Foo();
    double v;
};

struct Base1 : virtual public VBase {
    void Foo() override;
    virtual void Bar();
    double b1;
};

struct Base2 : virtual public VBase {
    virtual void Baz();
    double b2;
};

struct Derived: public Base1, public Base2 {
    double d;
};
```

<img src="./assets/vbase_1.png" alt="img" style="zoom:50%;" />





#### *Q: Size of an empty class?*

the size of an object should be at least 1 byte. This is to ensure that each object has a unique address, even for empty classes. So, the size of an empty class is typically 1 byte.

# Memory Management

## Memory architecture

- global area
- heap area
- stack area
- constant area
- code area

## Memory creation & deletion

### *Q: new/delete vs. malloc/free*
- Language support
  - `new/delete` are C++ operators.
  - `malloc/free` can be used in both C and C++.
- Return Type:
  - `new` return a point to the dynamically allocated object.
  - `malloc` return `void*` pointer, which needs to be explicitly cast to the appropriate type.
- Initialization of Objects:
  - `new`: It not only allocates memory but also calls the constructor of the object being created, ensuring proper initialization.
  - `malloc`: It only allocates memory but does not call the constructor, leaving the memory uninitialized. It's up to the programmer to explicitly call the constructor for initialization.

### Resource Acquisition Is Initialization

```cpp
#include <memory>

class MyResource {
public:
    MyResource() {
        // Get the resource in the constructor
        data = new int[100];
        // You can get other resources here, such as opening files, establishing network connections, etc.
    }

    ~MyResource() {
        // Release resources in destructor
        delete[] data;
        // You can release other resources here, such as closing files, closing network connections, etc.
    }
private:
    int* data;
};

int main() {
    // Manage resources with RAII
    std::unique_ptr<Resource> resourcePtr = std::make_unique<Resource>();

    // Use resources here, do not need to release resources manually, and will automatically release resources when leaving the scope

    return 0;
}
```



#### *Q: What is RAII based on (life cycle, scope, construction and destruction)*

- life cycle:
  In C++, when an object is created, its constructor is called and the object is initialized. When an object goes out of scope or is explicitly destroyed, its destructor is called and the object is destroyed. RAII uses this object life cycle mechanism to manage resource allocation and release. Resources typically refer to memory, file handles, network connections, or other system resources.
- Scope:
  Variables in C++ have a specific scope, and when variables go out of their scope, they will be destroyed. RAII uses the scope of variables to ensure that resources are released in a timely manner when they are no longer needed. Resources are encapsulated in objects, and when the object goes out of scope, the destructor is automatically called and the resources are released.
- Construct destructor:
  The core of RAII is to use the object's constructor to acquire resources, and use the destructor to release resources. Resource application or initialization is performed in the object's constructor, and resource release is performed in the destructor. This ensures that the resource remains valid throughout the life of the object, and also ensures that the resource is released when it is no longer needed, avoiding resource leaks.

## *Q: The situation of Memory leak?*

- Dynamic Memory Allocation: If you allocate memory using `new` and forget to deallocate it with `delete`, the memory will not be released, leading to a memory leak.
- Not properly managing pointers: If pointers are reassigned or lose track of their original memory location, it can become challenging to free the allocated memory correctly.
- Exceptions: If an exception is thrown before memory deallocation, the memory will not be freed.

## Pointers
### Smart Pointers

- `unique_ptr`
- `shared_ptr`
- `weak_ptr`

#### underlying implementation

- `unique_ptr` represents exclusive ownership of a dynamically allocated object and ensures that there is only one `unique_ptr` pointing to the object at any given time. When a `unique_ptr` is copied or assigned, ownership of the managed object is transferred, which helps prevent double deletion or memory leaks. The `unique_ptr` uses move semantics to transfer ownership efficiently.
- `shared_ptr` allows multiple `shared_ptr` instances to share ownership of the same dynamically allocated object. It keeps track of the number of `shared_ptr` instances pointing to the object using a reference counting mechanism. When the last `shared_ptr` pointing to the object is destroyed or reset, the object is deleted automatically. This allows you to have multiple references to the same object without worrying about manual memory management.
- `weak_ptr` is designed to work in conjunction with `shared_ptr`. It is a non-owning smart pointer that allows you to observe the object managed by a `shared_ptr` without affecting its reference count. The `weak_ptr` is useful to break cycles in reference counting scenarios (cyclic dependencies) that might cause memory leaks when using only `shared_ptr`. It provides a way to check if the object still exists before accessing it through `lock()` method.

#### *Q: How to solve the circular reference in shared_ptr?*

Answer is `weak_ptr`.

Here's an example of a circular reference:

```cpp
#include <iostream>
#include <memory>

class Node {
public:
    std::shared_ptr<Node> next;
};

int main() {
    std::shared_ptr<Node> node1 = std::make_shared<Node>();
    std::shared_ptr<Node> node2 = std::make_shared<Node>();

    node1->next = node2;
    node2->next = node1;

    // The shared_ptr reference counts will not reach zero because of the circular reference.
    // The Node objects will not be deallocated properly, causing a memory leak.
}
```

**Using std::weak_ptr**: `std::weak_ptr` is a smart pointer that does not affect the reference count of the managed object. It allows you to break the strong reference cycle. You can use it in combination with `std::shared_ptr` when you need to create a weak link that does not keep the objects alive.

```cpp
#include <iostream>
#include <memory>

class Node {
public:
    std::shared_ptr<Node> next;
};

int main() {
    std::shared_ptr<Node> node1 = std::make_shared<Node>();
    std::shared_ptr<Node> node2 = std::make_shared<Node>();

    node1->next = node2;
    node2->next = std::weak_ptr<Node>(node1); // Use weak_ptr to break the cycle.

    // When the shared_ptr node1 and node2 go out of scope, the Node objects will be deallocated properly.
}

```

Or Redesign your data structure, Breaking the reference manually
## References

- lvalue reference
- rvalue reference

### std::move



## *Q: references`&` vs. pointers`*`*

- Definition:
  - Pointers: A pointer is a variable that holds the memory address of another variable. 
  - References: A reference is an alias or an alternative name for an existing variable. 
- Memory usage:
  - Pointers: Pointers consume memory to store the address they point to. 
  - References: References do not consume additional memory. 
- Nullability:
  - Pointers: Pointers can be assigned a special value called `nullptr` (or `NULL` in older C++ versions) to indicate that they do not point to any valid memory address.
  - References: References must always be initialized when declared, and they cannot be null. 
- Reassignment
  - Pointers: Pointers can be reassigned to point to different memory addresses.
  - References: References cannot be reassigned after initialization; they remain bound to the same variable throughout their lifetime.

# [STL (Standard Template Library)](./Standard_Template_Library.md)

# Other

### Type Conversion

- `static_cast`
- `dynamic_cast`
- `const_cast`
- `reinterpret_cast`

Difference:

- `static_cast`: is used for general type conversions that can be determined at compile time. It can be used for implicit and explicit conversions between related types.

  ```cpp
  float f = 3.14;
  int num = static_cast<int>(f); // num will be 3
  ```
  
- `dynamic_cast`: The `dynamic_cast` is primarily used in polymorphic class hierarchies, where inheritance and virtual functions are involved. It allows for safe downcasting of pointers or references to derived classes. If the cast is not valid (e.g., if the object is not of the target type), it returns a null pointer for pointers or throws a `std::bad_cast` exception for references.

  ```cpp
  class Base {
    virtual void foo() {}
  };
  class Derived : public Base {};
  
  Base* basePtr = new Derived;
  Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);
  
  if (derivedPtr) {
      // The dynamic_cast was successful, use derivedPtr here
  } else {
      // The dynamic_cast failed
      // Handle the case where the object is not of the target type
  }
  ```

- `const_cast`: The `const_cast` is used to add or remove the `const` qualifier from a variable. It allows for modifying a `const` object by casting away its constness. However, it should be used with caution, as modifying a `const` object can lead to undefined behavior if the object was originally declared as `const`.

  ```cpp
  const int value = 42;
  int* ptr = const_cast<int*>(&value);
  *ptr = 100; // Modifying a const object, use with caution
  ```

- `reinterpret_cast`: The `reinterpret_cast` is the most dangerous type casting operator in C++ and should be used sparingly. It allows for converting one pointer type to another unrelated pointer type, and also can reinterpret an object's binary representation. This casting operator does not perform any type checking and is mostly used for low-level type conversions and platform-specific optimizations.

  ```cpp
  int num = 42;
  char* charPtr = reinterpret_cast<char*>(&num);
  // Use caution when using reinterpret_cast, as it can lead to undefined behavior
  ```

  

## 
