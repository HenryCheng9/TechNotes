## Static
static通常分为两类，一类是在类外部，另一类是在类内部。

类外部的static只对定义了它的单元可见，只在该编译单元的内部进行链接。类似于在类中使用private

类内部的static将和该类的所有实例共享内存，静态方法则没有实例会传给该静态方法

但注意静态方法不能访问非静态变量，因为静态方法没有类的实例，它不知道该调用那个实例的非静态变量。

通过static可以延长变量的生存期。

## 枚举
枚举是整数，如果需要限定一个变量的取值在几个整数中，可以考虑用枚举

## 构造函数
特殊类型的函数，在每一次实例化对象的时候运行。

如果不想让人们将类实例化，只想让其他人使用类中的静态方法，可以将构造函数设置为private

```cpp
class Log() {
	private:
		Log() {}
	public:
		static void hello() {
		//do something
	}
}
```

或者将构造函数删除

```cpp
class Log() {
	public:
		Log() = delete;
		static void hello() {
		//do something
	}
}
```

每个非静态方法总是获得当前类的一个实例作为参数

## 析构函数
构造函数是在实例化一个对象时运行，而析构函数是在销毁一个对象时运行。

适用于栈和堆分配的对象。

对于在stack上的对象，在对象超出作用域的时候，将会被自动销毁，此时将运行析构函数。

在堆上的对象中的内存需要手动清理。

## 虚函数和纯虚函数（接口）
CPP-P501

通过在基类中定义虚函数，可以在子类中实现override

```cpp
class Entity {
public:
	virtual std::string GetName() { //定义虚函数
		return "Entiry";
	}
};

class Player : public Entity {
private:
	std::string m_Name;
public:
	Player(const std::string& name) :m_Name(name) {}

	std::string GetName() override { //override的实现
		return m_Name;
	}
};
```

纯虚函数在基类中定义一个没有实现的函数，**强制子类去实现该函数**。对于子类来说必须实现所有的纯虚函数之后才能实例化。实现如下：

```cpp
class Entity {
public:
	virtual std::string GetName() = 0；
};
```

C++并没有interface关键字，实际上interface也是类的一种

### 虚函数的机制
example

```cpp
Derived derived;
Base* base;
base = &derived;
base->func();
```

func函数的执行将根据虚函数是否定义来执行：

-   如果在基类中没有标记func为虚函数，那么将根据指针类型调用Base::func()。指针类型在编译的时候已经确定，属于静态链接。
-   如果基类中标记了func是虚函数，那么调用的是Derived::func()。必须在执行的时候才能确定，属于动态链接。

**如何确定是否应该使用虚函数？**

如果基类不存在派生类那么就不需要虚函数

派生类不重新定义任何基类的方法，那么也不需要在基类中定义虚函数

**虚函数表**

CPP P504
## 可见性
默认的可见性是private

private的变量只有基类和友元可以访问

protected表示基类和子类可以访问

public可以让所有的类和对象访问
## 访问数组的方法

```cpp
	int arr[5]; 
	arr[3] = 2; 
	
	int* ptr = arr; 
	*(ptr + 3) = 2; 
	
	*(int*)((char*)ptr + 12) = 2;
```


上面创建数组是在stack上创建的，也可以通过new关键字在heap上创建

区别是生存期不同，stack上数组的生存期是在一个作用域中，而heap上数组的生存期是整个程序

```cpp
int* arr = new int[5];
```

实际上，用new开辟的内存生存期都是整个程序，除非自动删除销毁。

如果有一个函数返回一个数组，这个数组是在函数中创建的，那么就必须用new来分配空间。

如果在class中区别：

```cpp
class example {
	int arr[5];
	example() {
		for (int i = 0; i < 5; i++)
		{
			arr[i] = 2;
		}
	}
};

class example {
	int* arr = new int[5];
	example() {
		for (int i = 0; i < 5; i++)
		{
			arr[i] = 2;
		}
	}
};
```

前者直接在example类的位置保存了arr的数值，但后者只在example的位置保存了arr的地址（间接寻址）

数组元素的计数

```cpp
int arr[5];
int count = sizeof(arr) / sizeof(int);
```

但一般不用这个方法，需要自己维护数组的数量

```cpp
const int size  = 5;
int arr[size];
```

另一种创建数组的方法是通过

```cpp
std::array<int , 5> arr;
```

std::array 通常带有边界检查，并且自动维护size。因此会比原始数组更安全，但是速度不够

## 字符串
如何创建一个字符串

```cpp
const char* str = "Hello!";
std::string str = "Hello!";

```

双引号其实就是一个char*，字符串先初始化为一个char*之后再拓展为string

string的拓展

```cpp
std::string str = "Hello!";
str += " World";

std::string str = std::string("Hello ") + "World!";
```

## 关于 Const //todo
## 成员列表初始化
类里面一切的成员都应该用成员列表初始化，否则会浪费一些空间

```cpp
class Example {
public:
	Example() {
		std::cout << "Created Example" << std::endl;
	}

	Example(int x) {
		std::cout << "Created Example with " << x << std::endl;
	}
};

class Entity {
private:
	Example m_example;
	int m_number;
public:
	Entity() : m_example(Example(8)), m_number(6)
	{
		//do somethin
	}
};
```

区别（核心问题）：

-   如果是在构造函数内部初始化，那么是在所有的数据成员被分配内存空间后才进行的；
-   如果是利用列表初始化，则是在分配内存的时候就进行了初始化，之后才会进入构造函数执行函数体内的代码

必须使用的四种情况：

-   初始化一个引用成员时（why？因为引用必须要在分配内存的时候就要确定）
-   初始化一个常量成员时（和引用类似，常量成员在分配内存的时候就必须确定）
-   调用一个基类的构造函数，并且有一组参数时（分配内存的时候会默认调用没有参数的构造函数，上面的例子）
-   调用一个成员类的构造函数，并且有一组参数时

编译器会一一操作初始化列表，以适当的顺序在构造函数之内安插初始化操作，并且在任何显示用户代码之前；

list中的项目顺序是由类中的成员声明顺序决定的，不是由初始化列表的顺序决定的。（**初始化列表的顺序必须和类中的成员声明顺序一致**）

## 三元操作符
三元操作符的嵌套
```cpp
speed = level > 5 ? level > 10 ? 15 : 10 : 5;
```

## 创建并初始化对象
C++中需要选择对象放在什么地方，是在栈上还是在堆上

-   栈上的对象有一个自动的生存期，生存期取决于声明它的作用域
-   在堆上创建的对象需要由用户决定什么时候删除

```cpp
Entity e("cheng"); //created object on the stack
Entity* e1 = new Entity("zhl"); //created object on the heap
std::cout << e.GetStr() << std::endl;
std::cout << e1->GetStr() << std::endl;
delete e1; //free the memory
```

大多数情况下是在stack上创建对象的，但是如果需要显式的控制对象的生存期或者对象特别大的情况下，那么就需要在heap上创建对象

## new 关键字
new的作用就是在heap上分配内存，完成之后会返回指向这个内存的指针。同时在调用new的时候也会调用构造函数，但是malloc不会。

```cpp
Entity* e1 = new Entity("zhl");
Entity* e2 = (Entity*)malloc(sizeof(Entity));
int* b = new int[50];

delete[] b; //数组的new也需要用数组的delete
delete(e1);
free(e2);

//new的另一种用法:placement new
int* c = new int[50]; //200bytes
Entity* e3 = new(c) Entity(); //指在c的内存上创建对象e3
```

new的实现过程：

1.  首先调用名为operator new的标准库函数，分配足够大的原始为类型化的内存，以保存指定类型的一个对象
2.  运行该类型的一个**构造函数**，用指定的初始化构造对象
3.  最后返回指向新分配并构造后的对象指针

delete的实现过程：

1.  对指针指向的对象运行适当的**析构函数**
2.  调用名为operator delete的标准库函数释放该对象所用内存

**new/malloc和delete/free的区别**

-   new自动计算要分配的空间大小，malloc需要手工计算
-   new是类型安全的，malloc不是
    ```cpp
    int *p = new float[2]; //编译错误
    int *p = (int*)malloc(2 * sizeof(double));//编译无错误
    ```

-   malloc没有构造函数和operator new标准库函数的调用
-   malloc free是标准库函数，支持覆盖；而new delete是运算符，支持重载
-   malloc free仅仅分配和回收内存空间，不会调用构造函数和析构函数；而new和delete会
-   malloc free返回的是void*（**必须进行类型转换**）；而new delete返回的是具体类型指针

new和new[]的区别

-   new只会调用一次构造函数
-   new[]会调用每一个元素的构造函数

## 隐式转换和 explicit
C++中可以支持一次隐式转换，如下

```cpp
class Entity {
private:
	std::string m_name;
	int m_age;
public:
	Entity(const char* name) :m_name(name), m_age(-1) {};
	Entity(int age) :m_name("Unknown"), m_age(age) {};
};

int main() {
	Entity e0 = "zhl"; //隐式转换，将const char*  转换为Entity
	Entity e = 23; //将int转换为Entity
	std::cin.get();
	return 0;
}
```

但是不能实现多次的隐式转换，比如

```cpp
class Entity {
private:
	std::string m_name;
	int m_age;
public:
	Entity(const std::string& name) :m_name(name), m_age(-1) {};
	Entity(int age) :m_name("Unknown"), m_age(age) {};
};

int main() {
	Entity e0 = "zhl"; //报错，不存在const char* -> const std::string& -> Entity的隐式转换
	Entity e = 23;
	std::cin.get();
	return 0;
}
```

explicit可以禁用这种隐式转换，强调某个构造函数必须显式地被调用

```cpp
class Entity {
private:
	std::string m_name;
	int m_age;
public:
	explicit Entity(const char* name) :m_name(name), m_age(-1) {};
	explicit Entity(int age) :m_name("Unknown"), m_age(age) {};
};

int main() {
	Entity e0("zhl"); //只能显式调用
	Entity e(23);
	std::cin.get();
	return 0;
}
```

explicit关键字用来修饰类的构造函数，被修饰的构造函数的类，不能发生相应的隐式类型转换，只能以显示的方式进行类型转换，注意以下几点：

-   explicit 关键字**只能用于类内部的构造函数声明**上
-   explicit 关键字作用于**单个参数的构造函数**
-   被explicit修饰的构造函数的类，不能发生相应的隐式类型转换

explicit可以用来保护一些类型之间地转换，可以保护数据的安全

**关于隐式转换**

-   C++的基本类型中并非完全的对立，部分数据类型之间是可以进行隐式转换的。所谓隐式转换，是指不需要用户干预，编译器私下进行的类型转换行为。很多时候用户可能都不知道进行了哪些转换
-   C++面向对象的多态特性，就是通过父类的类型实现对子类的封装。通过隐式转换，你可以直接将一个子类的对象使用父类的类型进行返回。在比如，数值和布尔类型的转换，整数和浮点数的转换等。某些方面来说，隐式转换给C++程序开发者带来了不小的便捷。C++是一门强类型语言，类型的检查是非常严格的。
-   基本数据类型 基本数据类型的转换以取值范围的作为转换基础（保证精度不丢失）。隐式转换发生在从小->大的转换中。比如从char转换为int。从int->long。自定义对象 子类对象可以隐式的转换为父类对象
-   C++中提供了explicit关键字，在构造函数声明的时候加上explicit关键字，能够禁止隐式转换。
-   如果构造函数只接受一个参数，则它实际上定义了转换为此类类型的隐式转换机制。可以通过将构造函数声明为explicit加以制止隐式类型转换，**关键字explicit只对一个实参的构造函数有效**，需要多个实参的构造函数不能用于执行隐式转换，所以无需将这些构造函数指定为explicit。
## 运算符及重载

C++中可以将运算符进行重载，使得代码的可读性更高

```cpp
struct Vector2
{
	int x, y;

	Vector2(int x, int y) :x(x), y(y){};

	Vector2 add(const Vector2& other) const {
		return Vector2(x + other.x, y + other.y);
	}

	Vector2 operator+(const Vector2& other) const { //定义了+的重载
		return add(other);
	}
};

std::ostream& operator<<(std::ostream& stream, const Vector2& other) { //定义了<<的重载
	stream << other.x << " , " << other.y << std::endl;
	return stream;
}

std::ostream& operator<<(std::ostream& stream, const std::vector<int>& vec) { //<<的另一个重载

	for (const auto& it : vec) {
		stream << it << " ";
	}
	return stream;
}

int main() {
	Vector2 pos(2, 2);
	Vector2 speed(1, 1);

	Vector2 new_pos = pos + speed;
	std::cout << new_pos;

	std::vector<int> vec = { 1,2,3,4,5,6 };
	std::cout << vec << std::endl;
	std::cin.get();
	return 0;
}
```

## this 关键词
指向本类实例的一个指针

## 作用域指针

通过new分配的内存在heap上，不会自动释放，需要用户手动管理。

但是可以通过作用域指针来管理堆上的内存，一个实现的demo如下：

```cpp
class Entity {
public:
	Entity() {
		std::cout << "Created Entity!" << std::endl;
	}

	~Entity() {
		std::cout << "Destoryed Entity!" << std::endl;
	}
	 
};

class ScopedPtr {
private:
	Entity* m_ptr;
public:
	ScopedPtr(Entity* e) : m_ptr(e) {};
	~ScopedPtr() {
		delete m_ptr;
	}
};

int main() {
	{
		ScopedPtr e = new Entity();
	}
	std::cin.get();
	return 0;
}
```

作用域指针是在栈上创建的，在离开作用域的时候，作用域指针的析构函数被调用，在这个函数内部调用了delete删除了heap上的Entity。就实现了离开作用域时free掉Entity的内存。

## 智能指针

-   unique_ptr：作用域指针，不能被复制
-   shared_ptr：共享指针，引用计数
-   weak_ptr：弱指针

**作用域指针**

不能复制，可以在作用域结束的时候自动释放内存。

任何一个作用域指针die之后，就会销毁内存，因此其他的作用域指针不起作用，因此不能被复制。

```cpp
std::unique_ptr<Entity> entity0(new Entity());
std::unique_ptr<Entity> entity1 = std::make_unique<Entity>(); //不直接调用new是因为异常安全

//智能指针的构造函数是explicit的，不能做隐式转换，必须显式调用构造函数
std::unique_ptr<Entity> entity = new Entity(); //不允许
```

**共享指针**

可以被复制，通过引用计数来管理。当计数变为0的时候，指针就失效了。

```cpp
std::shared_ptr<Entity> sharedEntity0(new Entity());
std::shared_ptr<Entity> sharedEntity1 = std::make_shared<Entity>(); //不直接调用new是为了更有效率
```

shared_ptr需要分配另一块内存，用来存储引用计数的控制块，

-   如果采用第一种初始化方式，那么会先给new Entity分配内存，再控制块分配内存
-   第二种是一起分配的，更有效率

**弱指针**

可以给弱指针赋值为一个shared_ptr，但不会增加引用计数

![[Untitled.png]]
![[12.2.png]]![[12.4.png]]
![[12.5 1.png]]
## 复制与拷贝构造函数
当使用赋值操作符的时候，总是在复制一些值。除了引用。

拷贝构造函数就是用另一个实例化的对象来初始化一个新的对象。

```cpp
class Entity {
private:
	int m_age;
public:
	Entity(int x): m_age(x){
		std::cout << "Created Entity!" << std::endl;
	}

	Entity(const Entity& other) { //拷贝构造函数
		std::cout << "Copied Entity" << std::endl;
		this->m_age = other.m_age;
	}

	~Entity() {
		std::cout << "Destoryed Entity!" << std::endl;
	}
	 
};

int main() {
	Entity e1(3); //1.直接初始化
	Entity e2 = 4; //2.拷贝初始化
	Entity e3 = e2; //3.拷贝初始化

	std::cin.get();
	return 0;
}
```

1,2,3的区别：

1.  直接初始化，调用的是构造函数
2.  拷贝初始化，先为4创建一个临时对象，再将临时对象作为参数传递给**拷贝构造函数**。这种方法对于explicit修饰的构造函数不能使用。
3.  隐式调用了拷贝构造函数，类似于赋值操作符的重载

**什么情况下会用到拷贝构造函数**

-   用类的一个实例化对象去初始化另一个对象的时候
-   函数的参数是类的对象时（非引用传递）
-   函数的返回值是函数体内局部对象的类的对象时 ,此时虽然发生（Named return Value优化）NRV优化，但是由于返回方式是值传递，所以会在返回值的地方调用拷贝构造函数

默认拷贝构造函数的实现 example

```cpp
Rect::Rect(const Rect& r)
{
    width=r.width;
    height=r.height;
}
```

如果对于一个static int count的计数静态变量，在调用拷贝函数的时候会重新初始化，计数作用就失效了。也就是说，**编译器默认生成的拷贝函数并不会维护静态变量**。需要重写拷贝函数。

**防止默认拷贝发生**

通过对对象复制的分析，我们发现对象的复制大多在进行“值传递”时发生，这里有一个小技巧可以防止按值传递——**声明一个私有拷贝构造函数**。甚至不必去定义这个拷贝构造函数，这样因为拷贝构造函数是私有的，如果用户试图按值传递或函数返回该类对象，将得到一个编译错误，从而可以避免按值传递或返回对象。

当出现类的等号赋值时，会调用拷贝函数，在未定义显示拷贝构造函数的情况下，系统会调用默认的拷贝函数——即浅拷贝，它能够完成成员的一一复制。当数据成员中没有指针时，浅拷贝是可行的。但**当数据成员中有指针时**，如果采用简单的浅拷贝，则两类中的两个指针将指向同一个地址，当对象快结束时，会调用两次析构函数，而导致**指针悬挂**现象。所以，这时，必须采用深拷贝。

深拷贝与浅拷贝的区别就在于深拷贝会在堆内存中另外申请空间来储存数据，从而也就解决了指针悬挂的问题。**简而言之，当数据成员中有指针时，必须要用深拷贝**。

https://www.cnblogs.com/alantu2018/p/8459250.html

## 箭头操作符的重载
可以通过重载箭头操作符来直接操作类中成员对象的成员

```cpp
class Entity {
public:
	Entity(){
		std::cout << "Created Entity!" << std::endl;
	}

	~Entity() {
		std::cout << "Destoryed Entity!" << std::endl;
	}

	void Print() {
		std::cout << "Hello World!" << std::endl;
	}

	void Print() const {
		std::cout << "Const Hello World!" << std::endl;
	}
	 
};

class ScopedPtr
{
public:
	ScopedPtr(Entity* e) :m_entity(e) {};
	~ScopedPtr() {
		delete m_entity;
	};

	Entity* operator->() {
		return m_entity;
	}

	const Entity* operator->() const {
		return m_entity;
	}
private:
	Entity* m_entity;
};

int main() {
	ScopedPtr e(new Entity()); //非const
	e->Print();

	const ScopedPtr e1(new Entity()); //const指针
	e1->Print();

	std::cin.get();
	return 0;
}
```

**如何获得一个类中成员的偏移？**

```cpp
int offset = (int)&((Vector3*)nullptr)->x;
```


## 动态数组 Vector

push_back的时候为什么会出现对象的复制？

```cpp
std::vector<Entity> vec;
vec.push_back(Entity());
```

1.  首先Entity会创建一个对象，在main函数的栈帧中，然后需要将这个实例复制到vector的内存空间中，因此就会出现copy
2.  其次在vector调整容量的时候会将原来的对象复制到一块新的内存中，再添加新的对象

```cpp
class Entity {
public:
	int number;
public:
	Entity(int x) : number(x)  {
		std::cout << "Created Entity: " << this->number <<  std::endl;
	}

	~Entity() {
		std::cout << "Destoryed Entity: " << this->number << std::endl;
	}

	Entity(const Entity& other)
	 : number(other.number){
		std::cout << "Copied Entity: " << this->number <<  std::endl;
	}
};

int main() {
	std::vector<Entity> vec;
	vec.push_back(Entity(1));
	vec.push_back(Entity(2));
	vec.push_back(Entity(3));
	std::cin.get();
	return 0;
}
```

```cpp
Created Entity: 1
Copied Entity: 1
Destoryed Entity: 1
Created Entity: 2
Copied Entity: 2
Copied Entity: 1
Destoryed Entity: 1
Destoryed Entity: 2
Created Entity: 3
Copied Entity: 3
Copied Entity: 1
Copied Entity: 2
Destoryed Entity: 1
Destoryed Entity: 2
Destoryed Entity: 3
```


![[vector.jpeg]]
所以说vector的复制主要有两个原因导致：

1.  我们在main stack frame中创建了实例，然后需要复制到vector的内存空间，能不能考虑直接在vector的空间创建这个实例？
2.  vector的空间不断增加，能不能提前告知vector的容量大小

解决方案如下：

```cpp
int main() {
	std::vector<Entity> vec;
	vec.reserve(3); //解决问题2
	vec.emplace_back(1); //emplace_back解决问题1
	vec.emplace_back(2);
	vec.emplace_back(3);
	std::cin.get();
	return 0;
}
```

注意emplace_back是通过**传递构造函数的参数**来在vector的内存空间中直接创建对象实例

## 静态链接
include 一堆头文件，提供**函数的声明，告诉我们哪些函数可以使用**

libraries 预先构建的二进制文件，动态库（.dll）和静态库(.lib)。库文件提供函数的定义，保证正确调用和运行。

include时用引号和尖括号的区别：

引号先检查相对路径，如果没有就会找编译器的include文件

https://www.bilibili.com/video/BV1vp4y1W7ze/?spm_id_from=333.788.recommend_more_video.0

**vs里面的操作：**

添加include目录：
![[include.png]]

添加lib：
![[include2.png]]

![[include3.png]]

## 动态链接
动态链接是指链接发生在运行时，静态链接发生在编译时。

静态链接允许更多的优化。

头文件同时支持静态和动态链接，因此不需要动include。

同时需要把上面图片中的附加依赖项删除，添加xxxdll.lib类的文件（该文件保存了指向动态库的指针）。

然后编译生成之后，将dll文件放在exe文件同一目录下即可。

## VS 多项目
演示内容，多练习

https://www.bilibili.com/video/BV1Vp4y1s7uD/?spm_id_from=333.788.recommend_more_video.-1

## 多返回值
通常的方法可以使用输入参数来处理多种类型的返回值

元组tuple：一个类，可以包含x个变量，但是不关心类型

对pair：一个类，包含两个变量，可以不关心类型

但是这两种多返回值的处理方式并不理想，代码比较难读，通常可以自定义一个结构体来作为多返回值的类。

## 模版
模板并不实际存在，直到我们调用它的时候才会使用我们给定的模板参数来创建。（有点类似于动态链接）

编译模板时会发生什么？

根据用户写的函数进行填空，然后在编译的时候编译器自动完成剩余部分的代码。

模板就是编译器根据用户的规则，基于函数或类的使用来进行自动代码的编写。

```cpp
template<typename T, int N>
class Array {
private:
	T m_Array[N];
public:
	int getSize() const { return N; }
};

int main() {
	Array<int, 5> array1;
	Array<std::string, 4> array2;
	std::cout << array1.getSize() << std::endl;
	std::cout << array2.getSize() << std::endl;
	std::cin.get();
	return 0;
}
```

## 堆和栈

-   栈是从高到低分配的，所做的事情就是移动栈指针。不同变量之间的内存是依次分配的，它们之间会有一些内存守护防止溢出。而在离开作用域的时候，栈指针会依次往回走，非常简单
-   而在堆上分配内存，会做一系列复杂的工作，包括malloc、free list等工作

所以应该尽可能地使用栈分配

更详细的内容：
https://chenqx.github.io/2014/09/25/Cpp-Memory-Management/

## 宏
预处理阶段基本上是一个文本编辑阶段，可以控制什么代码会实际交给编译器

```cpp
#ifdef ZHL_DEBUG
#define LOG(X) std::cout << X << std::endl
#else
#define LOG(X)
#endif // ZHL_DEBUG

int main() {
	LOG("Hello World!");
	std::cin.get();
	return 0;
}

//version2
#if ZHL_DEBUG == 1
#define LOG(X) std::cout << X << std::endl
#elif defined(ZHL_RELEASE)
#define LOG(X)
#endif // ZHL_DEBUG
```

每一行后面转义可以在多行里面进行宏定义

## auto 关键字
在比较长的类型时可以使用，或者在for循环中可以使用，或者在函数返回值类型比较复杂的时候使用。

但是不要过度使用。

## 静态数组 std::array
-   静态数组和普通数组都是保存在栈中的
-   std::array 有边界检查
-   std::array 可以使用array.size(), 迭代器等STL操作

std::array中的size函数直接返回5，并不返回一个变量，并不占用多余的内存。

所以std::array相比普通的数组并不会影响太多的性能，反而增加了许多其他的东西。

```cpp
template<int N>
void PrintArray(const std::array<int, N>& arr) {
	for (int i = 0; i < arr.size(); ++i) {
		std::cout << arr[i] << " ";
	}
	std::cout << std::endl;
}
int main() {
	std::array<int, 5> arr;
	arr[0] = 1;
	arr[1] = 2;
	arr[2] = 3;
	arr[3] = 4;
	arr[4] = 5;
	PrintArray(arr);
    std::cin.get();
	return 0;
}
```

## 函数指针
基本使用方法：

```cpp
void HelloWorld() {
	std::cout << "Hello World!" << std::endl;
}

void Hello(int x) {
	std::cout << "Hello! Value: " << x << std::endl;
}

int main() {
	auto zheng = HelloWorld;
	zheng();

	void(*zhl)() = HelloWorld;
	zhl();

	typedef void(*HelloWorldFunction)();
	HelloWorldFunction func = HelloWorld;
	func();

	auto cheng = Hello;
	cheng(1);

	void(*harold)(int) = Hello;
	harold(2);

	typedef void(*HelloFunction)(int);
	HelloFunction hello = Hello;
	hello(3);

    std::cin.get();
	return 0;
}
```

实际用法：可以将函数作为一个参数传递给另外一个函数

```cpp
void PrintValue(int x) {
	std::cout << "Value: " << x << std::endl;
}

void ForEach(const std::vector<int>& vec, void(*func)(int)) {
	for (int value : vec)
		func(value);
}

int main() {
	std::vector<int> vec = { 1,2,3,4,5 };
	ForEach(vec, PrintValue);
    std::cin.get();
	return 0;
}

//other version
int main() {
	std::vector<int> vec = { 1,2,3,4,5 };
	ForEach(vec, [](int value){std::cout << "Value: " << value << std::endl;});
    std::cin.get();
	return 0;
}
```

## lambda

只要有一个函数指针，都可以在C++中使用lambda
https://www.cnblogs.com/DswCnblog/p/5629165.html

## 命名空间
namespace是为了避免命名冲突

```cpp
namespace apple {
    void print(const char* str) {
        std::cout << str << std::endl;
    }

    void print_again() {}
}

namespace orange {
    void print(const char* str) {
        std::string temp = str;
        std::reverse(temp.begin(), temp.end());
        std::cout << temp << std::endl;
    }
    namespace orange_in {
        void print_again() {}
    }
}

int main() {
    apple::print("Hello");
    orange::print("Hello");

    using apple::print_again; //导入某个特定的函数

    print_again();

    namespace o = orange; //命名空间重命名
    o::print("Hello");

    namespace oin = orange::orange_in; //嵌套
    oin::print_again();

    std::cin.get();
}
```

## 排序
`std::sort`默认从小到大排序，但可以通过修改compare函数来改变排序的逻辑。

```cpp
int main() {
    std::vector<int> vec = { 1,2,3,4,5 };
    std::sort(vec.begin(), vec.end(), [](int a, int b) {
        return a > b; //从大到小
        });
    std::cout << vec << std::endl; //5 4 3 2 1
    std::cin.get();
}
```

**compare函数是的逻辑是：**

-   如果需要把第一个参数放在第二个参数的前面，那么返回`true`
-   否则返回`false`

基于此我们可以构建如下的排序函数，确保1永远在最后一位

```cpp
int main() {
    std::vector<int> vec = { 1,2,3,4,5 };
    std::sort(vec.begin(), vec.end(), [](int a, int b) {
        if (a == 1)
            return false;
        if (b == 1)
            return true;
        return a < b;
        });
    std::cout << vec << std::endl; //2 3 4 5 1
    std::cin.get();
}
```

## 类型双关

C++运行我们通过访问内存的方式来改变数据的类型。具体的做法是将该类型作为一个指针，然后将其变成另一个指针，再将其解引用。就可以将A类型的内存空间变为B类型的内存空间。

```cpp
struct Entity
{
	int x, y;
};

int main() {
	int a = 5;
	double var = a; //00 00 00 00 00 00 14 40
	double var1 = (double)a; //00 00 00 00 00 00 14 40
	double var3 = *(double*)&a; //05 00 00 00 cc cc cc cc
	double& var4 = *(double*)&a; //05 00 00 00 cc cc cc cc

	Entity e = { 5,8 };
	int* position = (int*)&e;
	std::cout << position[0] << ", " << position[1] << std::endl;

	int y = *(int*)((char*)&e + 4);
	std::cin.get();
}
```

C++虽然有类型系统，但是可以通过直接操作内存的方式来绕过类型系统改变数据的类型。但是需要主要如果是不同大小的类型，那么就不能这样操作，否则会出现错误，比如var3, var4

## 联合体

共用体（union）是一种数据格式，它能够存储不同的数据类型，但只能同时存储其中的一种类型。联合体的用途之一是：当数据项使用两种或更多种格式（但不会同时使用）时，可节省空间。

公用元素采用联合体内内存最大的那一种内存大小。

```cpp
struct Vector2 {
	float a, b;
};

struct Vector4 {
	union {
		struct {
			float x, y, z, w;
		};

		struct {
			Vector2 a, b;
		};
	};
};

void PrintVector2(const Vector2& vec) {
	std::cout << vec.a << ", " << vec.b << std::endl;
}

int main() {
	Vector4 vector = { 1.0f, 2.0f, 3.0f, 4.0f };
	PrintVector2(vector.a);
	PrintVector2(vector.b);
	vector.z = 700.0f;
	PrintVector2(vector.a);
	PrintVector2(vector.b);
	std::cin.get();
}
```

在Vector4里面，采用联合体将其四个成员巧妙的分配到了两个Vector2中。他们的数据完全是一样的，只是可以用不同的方式来进行引用。

## 虚析构函数

如果用基类指针来引用派生类对象，那么基类的析构函数必须是 virtual 的，否则 C++ 只会调用基类的析构函数，不会调用派生类的析构函数。

除非类不用来做基类，否则析构函数必须是虚函数。

example1:

```cpp
class Base {
public:
	Base() {
		std::cout << "Base: Conscructor" << std::endl;
	}

	~Base() {
		std::cout << "Base: Descructor" << std::endl;
	}
};

class Derived : public Base {
public:
	Derived() {
		std::cout << "Derived: Conscructor" << std::endl;
	}

	~Derived() {
		std::cout << "Derived: Descructor" << std::endl;
	}
};

int main() {
	Base* base = new Base();
	delete base;
	std::cout << "--------------------------------" << std::endl;
	Derived* derived = new Derived();
	delete derived;

	std::cin.get();
}
```

输出结果：

![[output.png]]

但如果在派生类中申请了内存，并且在析构函数中释放了这些内存。而且在使用多态时，无法调用派生类的析构函数，会导致内存无法释放，出现内存泄漏。

example2

```cpp
class Derived : public Base {
public:
	int* arr;
	Derived() {
		arr = new int[5];
		std::cout << "Derived: Conscructor" << std::endl;
	}

	~Derived() {
		delete[] arr;
		std::cout << "Derived: Descructor" << std::endl;
	}
};

int main() {
	Base* base = new Base();
	delete base;
	std::cout << "--------------------------------" << std::endl;
	Base* derived = new Derived(); //notice
	delete derived;

	std::cin.get();
}
```

输出结果：
![[output0.png]]


解决方案：

必须将基类的析构函数标记为虚函数，这样编译器才能显式地知道需要调用派生类的析构函数，正确的释放内存空间。
```cpp
class Base {
public:
	Base() {
		std::cout << "Base: Conscructor" << std::endl;
	}

	virtual ~Base() { //notice
		std::cout << "Base: Descructor" << std::endl;
	}
};

class Derived : public Base {
public:
	int* arr;
	Derived() {
		arr = new int[5];
		std::cout << "Derived: Conscructor" << std::endl;
	}

	 ~Derived() {
		delete[] arr;
		std::cout << "Derived: Descructor" << std::endl;
	}
};

int main() {
	Base* base = new Base();
	delete base;
	std::cout << "--------------------------------" << std::endl;
	Base* derived = new Derived();
	delete derived;

	std::cin.get();
}
```

![[output1.png]]

## 类型转换

CPP-P642

常用的类型转换大体分为隐式类型转换和显式类型转换

C++风格的类型转换有四种

-   static_cast：C普通的类型转换
    
-   reinterpret_cast：操作内存，将内存从一种类型解释为另一种类型（即类型双关，不想转换任何东西，只是想把那种类型的指针解释成为别的东西） That's why it is **reinterpret**
    
-   dynamic_cast
    
    ```cpp
    class Grand {
    private:
    	int hold;
    public:
    	Grand(int h = 0) : hold(h) {}
    	virtual void Speak() const {
    		std::cout << "Grand Class" << std::endl;
    	}
    
    	virtual int Value() const {
    		return hold;
    	}
    };
    
    class Superb : public Grand {
    public:
    	Superb (int h = 0) : Grand(h){}
    	void Speak() const {
    		std::cout << "Superb Class" << std::endl;
    	}
    
    	virtual void Say() const {
    		std::cout << "Superb Value: " << Value() << std::endl;
    	}
    };
    
    class Magnificent :public Superb {
    private:
    	char ch;
    public:
    	Magnificent(int h = 0, char c = 'A') : Superb(h), ch(c) {}
    
    	void Speak() const {
    		std::cout << "Magnificent Class" << std::endl;
    	}
    
    	void Say() const {
    		std::cout << "Char: " << ch << "Value: " << Value() << std::endl;
    	}
    };
    
    Grand* GetOne();
    
    int main() {
    	std::srand(std::time(0));
    	Grand* pg;
    	Superb* ps;
    	for (int i = 0; i < 5; ++i) {
    		pg = GetOne();
    		pg->Speak();
    		if (ps = dynamic_cast<Superb*>(pg))
    			ps->Say();
    	}
    	std::cin.get();
    }
    
    Grand* GetOne() {
    	Grand* p = nullptr;
    	switch (std::rand() % 3) {
    	case 0:
    		p = new Grand(std::rand() % 100);
    		break;
    	case 1:
    		p = new Superb(std::rand() % 100);
    		break;
    	case 2:
    		p = new Magnificent(std::rand() % 100, 'A' + std::rand() % 26);
    		break;
    	}
    	return p;
    }
    ```
    
-   const_cast：添加或移除const修饰符
    

这种新特性更像是一种语法糖，方便阅读。它们的结果和C风格的类型转换一样，但是在编译的过程中做了一些其他的工作。

## 单例
https://www.cnblogs.com/sunchaothu/p/10389842.html