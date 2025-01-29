# STL (Standard Template Library)

[TOC]

# Core Components
* Containers
* Algorithms
* Iterators
* Functors
* Adapters
* Allocators

## Allocators
\<memory>

* alloc::allocate()  内存配置
* alloc::deallocate()  内存释放
* ::construct()  对象构造
* ::destroy()  对象析构

## Iterators
* Containers & Algorithms 之间的粘合剂
* Iterators是一种 smart pointer
	* dereference (*)
	* member access (->)

## Containers
* sequence Containers  
  Define: 元素都ordered，但未必sorted
	* vector
	* list
	* deque
	* stack
	* queue
	* priority-queue
	
* associative Containers
	* set
	* map

sdsfsdfsdfsdf