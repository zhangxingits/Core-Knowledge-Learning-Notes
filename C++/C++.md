

# C++语言

[TOC]

## 1. C++常用STL的使用

STL是一个标准模板库，是一个高效的C++程序库。

### 1.1 vector

vector是线性容器，它的元素严格按照线性序列排序，和动态数组很相似。和数组类似的是，它的元素存储在一块连续的存储空间中，这也意味着不仅可以使用迭代器（iterator）访问元素，还可以使用指针的偏移方式访问。和常规数组不一样的是，vector能够自动存储元素，可以自动增长或缩小存储空间。

vector的优点如下所述：

1. 可以使用下标访问个别的元素。
2. 迭代器可以按照不同的方式遍历容器。
3. 可以在容器的末尾增加或删除元素。

和数组相比，虽然容器在自动处理容量的大小时会消耗更多的内存，但是容器能提供和数组一样的性能，而且能很好地调整存储空间大小。和其他标准的顺序容器相比，vector能更有效访问容器内的元素和在末尾添加和删除元素；而在其他位置添加和删除元素，vector则不及其他顺序容器，在迭代器和引用也不比lists支持的好。

容器的大小和容器的容量是有区别的，大小是指元素的个数，容量是分配的内存大小，容量一般不小于容器的大小。vector::size()返回容器的大小，vector::apacity()返回容量值，容量多于容器大小的部分用于以防容器大小的增加使用。每次重新分配内存都会很影响程序的性能，所以一般分配的容量都大于容器的大小，若要自己指定分配的容量的大小，则可以使用vector::reserve()，但是规定的值要大于size()值。

### 1.2 vector的内存管理与效率

关于STL容器，最令人称赞的特性之一就是只要不超过它们的最大值，就可以自动增长到足以容纳用户放进去的数据的大小。（这个最大容量值，只要调用名叫max_size的成员函数就可以获得）对于vector和string，如果需要更多空间，就会以类似realloc的思想来增长大小。vector容器支持随机访问，因此为了提高效率，它内部是使用动态数组的方式实现的。在通过reserve()函数来申请特定大小的内存空间时候总是按指数边界来增大其内部缓冲区。当进行insert或push_back等增加元素的操作时，如果此时动态数组的内存不够用，就要动态的重新分配当前大小的1.5~2倍的新内存区，再把原数组的内容复制过去。所以，在一般情况下，其访问速度同一般数组相比，只有在重新分配发生时，其性能才会下降。正如例3.15中的代码一样，进行pop_back操作时，capacity并不会因为vector容器里的元素减少而有所下降，还会维持操作之前的大小。对于vector容器来说，如果有大量的数据需要进行push_back，应当使用reserve()函数提前设定其容量大小，否则会出现许多次容量扩充操作，导致效率低下。

reserve成员函数允许开发者最小化必须进行的重新分配的次数，因而可以避免真分配的开销和迭代器、指针、引用失效。但在解释reserve为什么可以那么做之前，需要先简要介绍有时候令人困惑的4个相关成员函数，如下所述。在标准容器中，只有vector和string提供了所有这些函数。

1. size()可以获得容器中有多少元素，但不能获得容器为它容纳的元素分配的内存大小。
2. capacity()可以获得容器在它已经分配的内存中可以容纳多少元素。那是容器在那块内存中总共可以容纳多少元素，而不是还可以容纳多少元素。如果想知道一个vector或string中有多少没有被占用的内存，则必须从capacity()中减去size()。如果size和capacity返回同样的值，容器中就没有剩余空间了，而下一次插入（通过insert或push_back等）会引发上面的重新分配步骤。
3. resize（Container::size_type n）用来强制把容器改为容纳n个元素。调用resize函数之后，size函数将会返回n。如果n小于当前大小，容器尾部的元素会被销毁。如果n大于当前大小，新默认构造的元素会添加到容器尾部。如果n大于当前容量，在元素加入之前会进行重新分配。
4. reserve（Container::size_type n）强制容器把它的容量改为不小于n，提供的n不小于当前所需的大小。因为容量需要增加，这一般会强迫进行一次重新分配。如果n小于当前容量，vector会忽略它，则这个调用什么都不做，string可能把它的容量减少为size()和n中大的数，但string的大小没有改变。

综上所述，只要有元素需要插入而且容器的容量不足时就会发生重新分配（包括它们维护的原始内存分配和回收，对象的拷贝和析构和迭代器、指针和引用的失效）。所以，避免重新分配的关键是使用reserve尽快把容器的容量设置为足够大，最好在容器被构造之后立刻进行。

例如，假定想建立一个容纳1~1000值的vector<int>，若不使用reserve，则可以像这样来做：

```c
vector<int> v;
for(int i = 0; i <= 1000; i++) v.push_bach(i);
```

 在大多数STL实现中，这段代码在循环过程中将会导致2~10次重新分配。（10这个数没什么奇怪的。记住vector在重新分配发生时一般把容量翻倍，而1000约等于2^{10}。）

把代码改为使用reserve，如下所示：

```c
vector<int> v;
v.reserve(1000);
for(int i = 0; i <= 1000; i++) v.push_bach(i);
```

则这在循环中不会发生重新分配。

通常有两种情况使用reserve来避免不必要的重新分配。第一种可用的情况是当知道有多少元素将最后出现在容器中时，就像上面的vector代码，就可以提前reserve适当数量的空间；第二种情况是保留可能需要的最大的空间，然后添加完全部数据后，再修整掉任何多余的容量。

### 1.3 使用“交换技巧”来修整vector过剩空间/内存

有一种方法来把它从曾经最大的容量减少到它现在需要的容量，这样的方法常常被称为“收缩到合适”（shrink to fit）。该方法只需一条语句：vector<int>（ivec）.swap（ivec）。表达式vector<int>（ivec）表示建立一个临时vector，它是ivec的一份拷贝。但是，vector的拷贝构造函数只分配拷贝的元素需要的内存，所以这个临时vector没有多余的容量。然后临时vector和ivec交换数据完成，但ivec只有临时变量的修整过的容量，而这个临时变量则持有了曾经在ivec中的没用到的过剩容量。在这个语句结尾处，临时vector被销毁，以释放以前ivec使用的内存，收缩到合适的大小。

### 1.4 用swap方法强行释放vector所占内存

```c
template < class T > void ClearVector( vector<T>& v )
{
    vector<T> vtTemp;
    vtTemp.swap( v );
}
```

```c
vector<int> v;
v.push_bach(1);
v.push_bach(2);
v.push_bach(3);
// 1. 方法1
vector<int>().swap(v);
// 2. 方法2
v.swap(vector<int>());
// 3. 方法3，加大括号{}是让tmp退出{}时自动析构
{ 
    std::vector<int> tmp = v;
    v.swap(tmp);
}
```

### 1.5 Vector类的简单实现

```c++
#include<algorithm>
#include<iostream>
#include <assert.h>
using namespace std;
template<typename T>
class myVector
{
private:
	/*walk length*/ 
	/*myVector each time increase space length*/ 
	#define WALK_LENGTH 64;

public:
	/*default constructor*/ 
	myVector():array(0),theSize(0),theCapacity(0){	}
	myVector(const T& t,unsigned int n):array(0),theSize(0),theCapacity(0){
		while(n--){
			push_back(t);
		}
	}

	/*copy constructor*/ 
	myVector(const myVector<T>& other):array(0),theSize(0),theCapacity(0){
		*this = other;
	}

	/*= operator*/ 
	myVector<T>& operator =(myVector<T>& other){
		if(this == &other)
			return *this;
		clear();
		theSize = other.size();
		theCapacity = other.capacity();
		array = new T[theCapacity];
		for(unsigned int i = 0 ;i<theSize;++i)
		{
			array[i] = other[i];
		}
		return *this;
	}

	/*destructor*/ 
	~myVector(){
		clear();
	}

	/*the pos must be less than myVector.size();*/ 
	T& operator[](unsigned int pos){
		assert(pos<theSize);
		return array[pos];
	}

	/*element theSize*/ 
	unsigned int size(){
		return theSize;
	}

	/*alloc theSize*/ 
	unsigned int capacity(){
		return theCapacity;
	}
	
	/*is  empty*/ 
	bool empty(){
		return theSize == 0;
	}

	/*clear myVector*/ 
	void clear(){
		deallocator(array);
		array = 0;
		theSize = 0;
		theCapacity = 0;
	}

	/*adds an element in the back of myVector*/  
	void push_back(const T& t){
		insert_after(theSize-1,t);
	}

	/*adds an element int the front of myVector*/ 
	void push_front(const T& t){
		insert_before(0,t);
	}

	/*inserts an element after the pos*/ 
	/*the pos must be in [0,theSize);*/ 
	void insert_after(int pos,const T& t){
		insert_before(pos+1,t);
	}

	/*inserts an element before the pos*/ 
	/*the pos must be less than the myVector.size()*/ 
	void insert_before(int pos,const T& t){
		if(theSize==theCapacity){
			T* oldArray = array;
			theCapacity += WALK_LENGTH; 
			array = allocator(theCapacity);
			/*memcpy(array,oldArray,theSize*sizeof(T)):*/ 
			for(unsigned int i = 0 ;i<theSize;++i){
				array[i] = oldArray[i];
			}
			deallocator(oldArray);
		}

		for(int i = (int)theSize++;i>pos;--i){
			array[i] = array[i-1];
		}
		array[pos] = t;
	}

	/*erases an element in the pos;*/ 
	/*pos must be in [0,theSize);*/ 
	void erase(unsigned int pos){
		if(pos<theSize){
			--theSize;
			for(unsigned int i = pos;i<theSize;++i){
				array[i] = array[i+1];
			}
		}
	}

private:
	T*  allocator(unsigned int size){
		return new T[size];
	}

	void deallocator(T* arr){
		if(arr)
			delete[] arr;
	}
private:
	T*				array;
	unsigned int	theSize;
	unsigned int	theCapacity;
};

void printfVector(myVector<int>& vector1){
	for(unsigned int i = 0 ; i < vector1.size();++i){
		cout<<vector1[i]<<",";
	}
	cout<<"alloc size = "<<vector1.capacity()<<",size = "<<vector1.size()<<endl;
}
```

测试验证：

```c++
int main(){
	myVector<int> myVector1;
	myVector<int> myVector2(0,10);
	myVector2.push_front(1);
	myVector2.erase(11);
	printfVector(myVector2);
	myVector1.push_back(2);
	myVector1.push_front(1);
	printfVector(myVector1);
	myVector1.insert_after(1,3);
	printfVector(myVector1);

	myVector2 = myVector1;
	myVector2.insert_before(0,0);
	myVector2.insert_before(1,-1);
	printfVector(myVector2);
    return 0;
}
```

STL库中vector是一个自动管理的动态数组，只要明白vector的类型是一个数组，至于怎么去实现它其实不难。例3.17中选择了一种简单的方式去实现它：定义一个步长WALK_LENGTH，在数组空间不够时，再重新申请长度为theCapacity+WALK_LENGTH的内存，这样就避免了每次当myVector元素增加的时候，需要去重新申请空间的问题，当然不好的地方就是会浪费一定的空间，但是时间效率上会提高很多。因为vector可以支持下标访问，所以就不用单独构造一个iterator，从而提高效率。