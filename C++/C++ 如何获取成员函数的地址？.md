# C++ 如何获取成员函数的地址？

```c++
#include <iostream>
using namespace std;

//typedef void(*Fun)(void);

class Base {
public:
    void i() { cout << "Base::i" << endl; };
};
int main()
{
    union {
      void* pv;
      void(Base::* pfn)();
    } u;
    u.pfn = &Base::i;
    Base b;
    (b.*u.pfn)();
}
```



```c++
#include <iostream>

class D
{
	public:
		D() {};
		~D() {};
		virtual void fun() { std::cout << "fun D" << "\n"; }
		virtual void wife() {std::cout << "my name is " << this->name << ",I'm [bored]~" <<"\n"; }
	private:
		char name[5] = "D";
};

class E
{
	public:
		E() {};
		~E() {};
		virtual void fun() { std::cout << "fun E" << std::endl; }
		virtual void wife() { std::cout << "<wife fun>:my name is " << this->name << ",I'm [excited]~" << std::endl; }

		void daughter() { std::cout << "<daughter fun>:my name is " << this->name << ",I'm very [excited]~" << std::endl; }
	private:
		char name[5] = "E";
};

void foo() { std::cout << "hello world" << std::endl; }

int main()
{
	D* d = new D();
	E* e = new E();
	
	void* efun = *((void**)*((void**)e));
	void* ewife = *((void**)*((void**)e) + 1); // 只好找出E的wife地址，自己跑去调用，嘻嘻，

	__asm {
		//call foo


		mov ecx, dword ptr[e]
		call efun

		mov ecx, dword ptr[e]
		call ewife

		mov eax, E::daughter
		mov ecx, dword ptr[d]
		call eax
	}

	return 0;
}
```

