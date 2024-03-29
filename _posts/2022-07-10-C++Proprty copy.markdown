---
title:  "C++ Auto-Implemented Properties"
date:   2022-07-10 15:54:00
categories: [C++ Study]
tags: [C++ Property]
---
The most interesting thing while studying C# is the property function!

After implementing this property as a header file in c++, if you can change it at will, you will use it well for a long time, and I implemented it because I thought the maintenance of the function prototype would be good!

But the problem was a read-only property.

Automatic implementation of properties in C# Read-only properties can simply be made like this.

```csharp
public int Property {get; private set;}
Property = 10;
```
<b>I made this function with C++.</b>

## Property.h
```cpp
#pragma once
//read-write
template<class T> class PropTemp {
private:
	T variable;
public:
	PropTemp(const T variable) { this->variable = variable; }
	void set(T variable) { this->variable = variable; }
	T get() const { return variable; }
};

template<class T> class PropReadOnly {
private:
	T variable;
public:
	PropReadOnly(const T variable) { this->variable = variable; }
	T get() const { return variable; }
};

template<class T> class PropWriteOnly {
private:
	T variable;
public:
	PropWriteOnly(const T variable) { this->variable = variable; }
	void set(T variable) { this->variable = variable; }
};
```

## main.cpp

```cpp
#include<iostream>
#include "Property.h"
using namespace std;
int main() {

	//Read and Write Property
	PropTemp<int> protemp(9);
	cout << "protemp1.get() = " << protemp.get() << "\n";
	protemp.set(10);
	cout << "protemp1.get() = " << protemp.get() << "\n";

	//Read-Only Property
	PropReadOnly<float> proReadOnly(15);
	cout << "proReadOnly.get() = " << proReadOnly.get() << "\n";
	//proReadOnly.set(10);  -> Error!!

	//Write-Only Property
	PropWriteOnly<string> prowriteOnly("Hi");
	//cout << "prowriteOnly.get() = " << prowriteOnly.get() << "\n";  -> Error!!
	prowriteOnly.set("hello");
}
```

<b>Thank you :)</b>
