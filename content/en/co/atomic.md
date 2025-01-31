---
weight: 2
title: "Atomic Operations"
---

include: [co/atomic.h](https://github.com/idealvin/co/tree/master/include/co/atomic.h).


## Arithmetic operations


### atomic_inc


```cpp
template<typename T>
inline T atomic_inc(T* p);
```


- Atomic increment, T is any integer type with a length of 1, 2, 4, 8 bytes, and the parameter p is a pointer of type T.
- This function performs an increment operation on the integer pointed to by p and returns the result after increment.



- Example



```cpp
int i = 0;
uint64 u = 0;
int r = atomic_inc(&i);    // i -> 1, r = 1
uint64 x = atomic_inc(&u); // u -> 1, x = 1
```


### atomic_fetch_inc


```cpp
template<typename T>
inline T atomic_fetch_inc(T* p);
```


- The same as [atomic_inc](#atomic_inc), but returns the value before increment.



- Example



```cpp
int i = 0;
uint64 u = 0;
int r = atomic_fetch_inc(&i);    // i -> 1, r = 0
uint64 x = atomic_fetch_inc(&u); // u -> 1, x = 0
```


### atomic_dec


```cpp
template<typename T>
inline T atomic_dec(T* p);
```


- Atomic decrement, T is any integer type with a length of 1, 2, 4, 8 bytes, and the parameter p is a pointer of type T.
- This function decrements the integer pointed to by p and returns the decremented result.



- Example



```cpp
int i = 1;
uint64 u = 1;
int r = atomic_dec(&i);    // i -> 0, r = 0
uint64 x = atomic_dec(&u); // u -> 0, x = 0
```


### atomic_fetch_dec


```cpp
template<typename T>
inline T atomic_fetch_dec(T* p);
```


- The same as [atomic_dec](#atomic_dec), but returns the value before decrement.



- Example



```cpp
int i = 1;
uint64 u = 1;
int r = atomic_fetch_dec(&i);    // i -> 0, r = 1
uint64 x = atomic_fetch_dec(&u); // u -> 0, x = 1
```


### atomic_add


```cpp
template<typename T, typename V>
inline T atomic_add(T* p, V v);
```


- Atomic addition, T is any integer type with a length of 1, 2, 4, 8 bytes, V is any integer type, and the parameter p is a pointer of type T.
- This function adds the value v to the integer pointed to by p, and returns the result after adding v.



- Example



```cpp
int i = 0;
uint64 u = 0;
int r = atomic_add(&i, 1);    // i -> 1, r = 1
uint64 x = atomic_add(&u, 1); // u -> 1, x = 1
```


### atomic_fetch_add


```cpp
template<typename T, typename V>
inline T atomic_fetch_add(T* p, V v);
```


- The same as [atomic_add](#atomic_add), but returns the value before adding v.



- Example



```cpp
int i = 0;
uint64 u = 0;
int r = atomic_fetch_add(&i, 1);    // i -> 1, r = 0
uint64 x = atomic_fetch_add(&u, 1); // u -> 1, x = 0
```


### atomic_sub


```cpp
template<typename T, typename V>
inline T atomic_sub(T* p, V v);
```


- Atomic subtraction, T is any integer type with a length of 1, 2, 4, 8 bytes, V is any integer type, and the parameter p is a pointer of type T.
- This function subtracts value v from the integer pointed to by p, and returns the result of subtracting v.



- Example



```cpp
int i = 1;
uint64 u = 1;
int r = atomic_sub(&i, 1);    // i -> 0, r = 0
uint64 x = atomic_sub(&u, 1); // u -> 0, x = 0
```


### atomic_fetch_sub


```cpp
template<typename T, typename V>
inline T atomic_fetch_sub(T* p, V v);
```


- The same as [atomic_sub](#atomic_sub), but returns the value before subtracting v.



- Example



```cpp
int i = 1;
uint64 u = 1;
int r = atomic_fetch_sub(&i, 1);    // i -> 0, r = 1
uint64 x = atomic_fetch_sub(&u, 1); // u -> 0, x = 1
```




## Bit operation


### atomic_or


```cpp
template<typename T, typename V>
inline T atomic_or(T* p, V v);
```


- Atomic bitwise OR operation, T is any integer type with a length of 1, 2, 4, 8 bytes, V is any integer type, and the parameter p is a pointer of type T.
- This function performs bitwise OR operation on the integer pointed to by p and v, and returns the result of the operation.



- Example



```cpp
int i = 5;
uint64 u = 5;
int r = atomic_or(&i, 3);    // i |= 3, i -> 7, r = 7
uint64 x = atomic_or(&u, 3); // u |= 3, u -> 7, x = 7
```


### atomic_fetch_or


```cpp
template<typename T, typename V>
inline T atomic_fetch_or(T* p, V v);
```


- The same as [atomic_or](#atomic_or), but returns the value before the bitwise OR operation.



- Example



```cpp
int i = 5;
uint64 u = 5;
int r = atomic_fetch_or(&i, 3);    // i |= 3, i -> 7, r = 5
uint64 x = atomic_fetch_or(&u, 3); // u |= 3, u -> 7, x = 5
```


### atomic_and


```cpp
template<typename T, typename V>
inline T atomic_and(T* p, V v);
```


- Atomic bitwise AND operation, T is any integer type with a length of 1, 2, 4, 8 bytes, V is any integer type, and the parameter p is a pointer of type T.
- This function performs bitwise AND operation on the integer pointed to by p and v, and returns the result after the operation.



- Example



```cpp
int i = 5;
uint64 u = 5;
int r = atomic_and(&i, 3);    // i &= 3, i -> 1, r = 1
uint64 x = atomic_and(&u, 3); // u &= 3, u -> 1, x = 1
```


### atomic_fetch_and


```cpp
template<typename T, typename V>
inline T atomic_fetch_and(T* p, V v);
```


- The same as [atomic_and](#atomic_and), but returns the value before the bitwise AND operation.



- Example



```cpp
int i = 5;
uint64 u = 5;
int r = atomic_fetch_and(&i, 3);    // i &= 3, i -> 1, r = 5
uint64 x = atomic_fetch_and(&u, 3); // u &= 3, u -> 1, x = 5
```


### atomic_xor


```cpp
template<typename T, typename V>
inline T atomic_xor(T* p, V v);
```


- Atomic bitwise XOR operation, T is any integer type with a length of 1, 2, 4, 8 bytes, V is any integer type, and the parameter p is a pointer of type T.
- This function performs a bitwise XOR operation on the integer pointed to by p and v, and returns the result of the operation.



- Example



```cpp
int i = 5;
uint64 u = 5;
int r = atomic_xor(&i, 3);    // i ^= 3, i -> 6, r = 6
uint64 x = atomic_xor(&u, 3); // u ^= 3, u -> 6, x = 6
```


### atomic_fetch_xor


```cpp
template<typename T, typename V>
inline T atomic_fetch_xor(T* p, V v);
```


- The same as [atomic_xor](#atomic_xor), but returns the value before the bitwise XOR operation.



- Example



```cpp
int i = 5;
uint64 u = 5;
int r = atomic_fetch_xor(&i, 3);    // i ^= 3, i -> 6, r = 5
uint64 x = atomic_fetch_xor(&u, 3); // u ^= 3, u -> 6, x = 5
```




## Exchange


### atomic_swap


```cpp
template<typename T, typename V>
inline T atomic_swap(T* p, V v);
```


- Atomic swap operation, T is any built-in type (including pointer type) with a length of 1, 2, 4, 8 bytes, and V is any type that can be converted to type of T.
- This function exchanges the value pointed to by p and v, and returns the value before the exchange.



- Example



```cpp
bool b = false;
int i = 0;
void* p = 0;
bool x = atomic_swap(&b, true);      // b -> true, x = false
int r = atomic_swap(&i, 1);          // i -> 1, r = 0
void* q = atomic_swap(&p, (void*)8); // p -> 8, q = 0
```


### atomic_compare_swap


```cpp
template<typename T, typename O, typename V>
inline T atomic_compare_swap(T* p, O o, V v);
```


- Atomic swap operation, T is any built-in type (including pointer type) with a length of 1, 2, 4, 8 bytes, O and V are any types that can be converted to type of T.
- This function exchanges with v only when the value pointed to by p is equal to o.
- This function returns the value before the exchange operation. The user can judge whether the exchange operation is performed according to whether the return value is equal to o.



- Example



```cpp
bool b = false;
int i = 0;
void* p = 0;
bool x = atomic_compare_swap(&b, false, true);  // b -> true, x = false
int r = atomic_compare_swap(&i, 1, 2);          // No swap, i remains unchanged, r = 0
void* q = atomic_compare_swap(&p, 0, (void*)8); // p -> 8, q = 0
```




## get & set


### atomic_get


```cpp
template<typename T>
inline T atomic_get(T* p);
```


- This function gets the value of the variable pointed to by p, T is any built-in type (including pointer type) with a length of 1, 2, 4, 8 bytes.



- Example



```cpp
int i = 7;
int r = atomic_get(&i); // r = 7
```


### atomic_set


```cpp
template<typename T, typename V>
inline void atomic_set(T* p, V v);
```


- This function sets the value pointed to by p to v, T is any built-in type (including pointer type) with a length of 1, 2, 4, 8 bytes, and V is any type that can be converted to type of T.



- Example



```cpp
int i = 7;
atomic_set(&i, 3); // i -> 3
```


### atomic_reset


```cpp
template<typename T>
inline void atomic_reset(T* p);
```


- This function sets the value pointed to by p to 0, and T is any built-in type (including pointer types) with a length of 1, 2, 4, or 8 bytes.



- Example



```cpp
int i = 7;
void* p = &i;
atomic_reset(&i); // i -> 0
atomic_reset(&p); // p -> 0
```


