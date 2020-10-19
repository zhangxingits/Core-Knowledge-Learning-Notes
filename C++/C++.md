

# C++语言

[TOC]

## 1. C++常用STL的使用

STL是一个标准模板库，是一个高效的C++程序库。

### 1.1 std::vector

vector是线性容器，它的元素严格按照线性序列排序，和动态数组很相似。和数组类似的是，它的元素存储在一块连续的存储空间中，这也意味着不仅可以使用迭代器（iterator）访问元素，还可以使用指针的偏移方式访问。和常规数组不一样的是，vector能够自动存储元素，可以自动增长或缩小存储空间。

vector的优点如下所述：

1. 可以使用下标访问个别的元素。
2. 迭代器可以按照不同的方式遍历容器。
3. 可以在容器的末尾增加或删除元素。

和数组相比，虽然容器在自动处理容量的大小时会消耗更多的内存，但是容器能提供和数组一样的性能，而且能很好地调整存储空间大小。和其他标准的顺序容器相比，vector能更有效访问容器内的元素和在末尾添加和删除元素；而在其他位置添加和删除元素，vector则不及其他顺序容器，在迭代器和引用也不比lists支持的好。

容器的大小和容器的容量是有区别的，大小是指元素的个数，容量是分配的内存大小，容量一般不小于容器的大小。vector::size()返回容器的大小，vector::apacity()返回容量值，容量多于容器大小的部分用于以防容器大小的增加使用。每次重新分配内存都会很影响程序的性能，所以一般分配的容量都大于容器的大小，若要自己指定分配的容量的大小，则可以使用vector::reserve()，但是规定的值要大于size()值。

#### 1.1.1 vector的内存管理与效率

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

#### 1.1.2 使用“交换技巧”来修整vector过剩空间/内存

有一种方法来把它从曾经最大的容量减少到它现在需要的容量，这样的方法常常被称为“收缩到合适”（shrink to fit）。该方法只需一条语句：vector<int>（ivec）.swap（ivec）。表达式vector<int>（ivec）表示建立一个临时vector，它是ivec的一份拷贝。但是，vector的拷贝构造函数只分配拷贝的元素需要的内存，所以这个临时vector没有多余的容量。然后临时vector和ivec交换数据完成，但ivec只有临时变量的修整过的容量，而这个临时变量则持有了曾经在ivec中的没用到的过剩容量。在这个语句结尾处，临时vector被销毁，以释放以前ivec使用的内存，收缩到合适的大小。

#### 1.1.3 用swap方法强行释放vector所占内存

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

#### 1.1.4 Vector类的简单实现

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

通过分析代码，可以发现vector的特点，如下所述：

1. 随即访问元素效率很高。
2. push_back的效率也会很高。
3. push_front的效率非常低，不建议使用。
4. insert需要把插入位置以后的元素全部后移，效率比较低，不建议使用。
5. erase需要把删除位置后面的元素全部前移，效率比较低，不建议使用。
6. 当内存不够时，需要重新申请内存，再把以前的元素复制过来，效率也比较低

### 1.2 std::map

map本质是一类关联式容器，属于模板类关联的本质在于元素的值与某个特定的键相关联，而并非通过元素在数组中的位置类获取。它的特点是增加和删除节点对迭代器的影响很小，除了操作节点，对其他的节点都没有什么影响。对于迭代器来说，不可以修改键值，只能修改其对应的实值。map内部数据的组织，map内部自建一棵红黑树（一种非严格意义上的平衡二叉树），这棵树具有对数据自动排序的功能，所以在map内部所有的数据都是有序的。

key和value可以是任意你需要的类型，但是需要注意的是对于key的类型，唯一的约束就是必须支持“<”操作符。

根据key值快速查找记录，查找的复杂度基本是Log（N），即如果有1000个记录，最多查找10次；1000000个记录，最多查找20次。除此之外，还有快速插入Key-Value记录、快速删除记录、根据Key修改value记录、遍历所有记录等功能。



#### 1.2.1. map的插入

先讲下map的插入，map的插入有3种方式：用insert函数插入pair数据、用insert函数插入value_type数据和用数组方式插入数据。

- 用insert函数插入pair数据

  ```c++
  map<int, string> mapStudent;
  mapStudent.insert(pair<int, string>(1, "student_one"));
  mapStudent.insert(pair<int, string>(2, "student_two"));
  mapStudent.insert(pair<int, string>(3, "student_three"));
  map<int, string>::iterator iter;
  for(iter = mapStudent.begin(); iter != mapStudent.end(); iter++){
      cout<<iter->first<<" "<<iter->second<<endl;
  }
  ```

- 用insert函数插入value_type数据

  ```c++
  map<int, string> mapStudent;
  mapStudent.insert(map<int, string>::value_type (1,"student_one"));
  mapStudent.insert(map<int, string>::value_type (2,"student_two"));
  mapStudent.insert(map<int, string>::value_type (3,"student_three"));
  
  map<int, string>::iterator  iter;
  for(iter = mapStudent.begin(); iter != mapStudent.end(); iter++){
      cout<<iter->first<<" "<<iter->second<<endl;
  }
  ```

- map中用数组方式插入数据

  ```c++
  map<int, string> mapStudent;
  mapStudent[1] =  "student_one";
  mapStudent[2] =  "student_two";
  mapStudent[3] =  "student_three";
  for(iter = mapStudent.begin(); iter != mapStudent.end(); iter++){
      cout<<iter->first<<" "<<iter->second<<endl;
  }
  ```

以上3种用法，虽然都可以实现数据的插入，但是它们是有区别的，当然了第一种和第二种在效果上是完全一样的，用insert函数插入数据，在数据的插入上涉及集合的唯一性这个概念，即当map中有这个关键字时，insert操作是插入不了数据的，但是用数组方式就不同了，它可以覆盖以前该关键字对应的值。那么这就涉及如何知道insert语句是否插入成功的问题了，可以用pair来获得是否插入成功。

可以通过insert返回的pair的第二个变量来知道是否插入成功，它的第一个变量返回的是一个map的迭代器，如果插入成功的话insert_Pair.second应该是true的，否则为false。

```c++
map<int, string> mapStudent;
pair<map<int, string>::iterator, bool> insert_pair;
insert_pair = mapStudent.insert(pair<int,string>(1,"student_one"));
if(insert_pair.second == true){
    cout<<"Insert Successfully"<<endl;
}
else{
    cout<<"Insert Failure"<<endl;
}

insert_pair = mapStudent.insert(pair<int, string>(1, "student_two"));
if(insert_pair.second == true){
    cout<<"Insert Successfully"<<endl;
}else{
    cout<<"Insert Failure"<<endl;
}
map<int, string>::iterator iter;
for(iter = mapStudent.begin(); iter != mapStudent.end(); iter++){
    cout<<iter->first<<" "<<iter->second<<endl;
}
```

用pair判断insert到map的数据是否插入成功。pair变量insert_pair中的第一个元素的类型是map<int,string>::iterator，是和即将要判断的map中的key、value类型一致的一个map的迭代器。如果insert成功了，则insert_pair.second的结果为true，否则则为false。同一个key已经有数据之后，再insert就会失败。而数组插入的方式，则是直接覆盖。

#### 1.2.2 map的遍历

map数据的遍历，这里也提供3种方法，来对map进行遍历：应用前向迭代器方式、应用反向迭代器方式和数组方式。应用前向迭代器，上面举例程序中已经讲解过了，这里着重讲解应用反向迭代器的方式，下面举例说明。

- map反向迭代器的使用举例

  ```c++
  map<int,string> mapStudent;
  mapStudent[1] =  "student_one"; 
  mapStudent[2] =  "student_two"; 
  mapStudent[3] =  "student_three";
  
  map<int, string>::reverse_iterator   iter;
  for(iter = mapStudent.rbegin(); iter != mapStudent.rend(); iter++){
      cout<<iter->first<<" "<<iter->second<<endl;
  }
  ```

#### 1.2.3 map的查找

要判定一个数据（关键字）是否在map中出现的方法比较多，这里给出2种常用的数据查找方法。

第一种：用count函数来判定关键字是否出现，其缺点是无法定位数据出现位置，由于map的一对一的映射特性，就决定了count函数的返回值只有两个，要么是0，要么是1，当要判定的关键字出现时返回1。

第二种：用find函数来定位数据出现位置，它返回的一个迭代器，当数据出现时，它返回数据所在位置的迭代器；如果map中没有要查找的数据，它返回的迭代器等于end函数返回的迭代器。

- 用find方法查找map中的数据，find函数返回的是一个迭代器；找不到对应数据的时候，则会返回mapStudent.end()。

```c++
map<int,string> mapStudent;
mapStudent[1] = "student_one"; 
mapStudent[2] = "student_two"; 
mapStudent[3] = "student_three";
map<int, string>::iterator iter=mapStudent.find(1); 
if(iter != mapStudent.end()){
	cout<<"Found, the value is "<<iter->second<<endl;
}else{
	cout<<"Do not found"<<endl; 
}
```

#### 1.2.4 map的删除

用erase方法可删除map中的元素。

```c++
map.erase(k);	 // 删除map中键为k的元素，并返回size_type类型的值以表示删除的元素个数
map.erase(p);	 // 从map中删除迭代器p所指向的元素。p必须指向map中确实存在的元素，而且不能等于map.end()，返回void类型。
map.erase(b, e); /* 从map中删除一段范围内的元素，该范围由迭代器对b和e标记。b和e必须标记map中的一段有效范围：即b和e都必须指向map中的元素或最后一个元素的下一个位置。而且，b和e要么相等（此时删除的范围为空），要么b所指向的元素必须出现在e所指向的元素之前，返回void类型。常用的是第二种，并且是在遍历的过程中删除元素。 */
```

```c++
 map<int, string> mapStudent;
 mapStudent[1]="student_one";
 mapStudent[2]="student_two";
 mapStudent[3]="student_three";
 mapStudent[4]="student_four";	

 map<int, string>::iterator iter=mapStudent.begin();
 for(;iter!=mapStudent.end();){
 	if((*iter).second=="student_one"){
 		mapStudent.erase(iter++);
 	}
 	else{
 		++iter;
 	}
 }	

 for(iter=mapStudent.begin();iter!=mapStudent.end();iter++){
 	cout<<iter->first<<" "<<iter->second<<endl;
 }
```

> 注意：mapStudent.erase(iter++)；中的iter++，不是erase(iter)，然后iter++。因为iter指针被erase之后就失效了，不能再用iter++；也不是erase(++iter)，这样就不是删iter原来指向的元素了。

#### 1.2.5 map的排序

map的排序默认按照key从小到大排序，但有以下几点需要注意：①按照key从大到小排序；②key（第一个元素）是一个结构体；③想按value（第二个元素）排序。

```c++
template < class Key, class T, class Compare = less<Key>, class Allocator = allocator<pair<const Key,T> > > 
class map;
```

它有4个参数，其中比较熟悉的有两个：Key和Value。第4个是Allocator，用来定义存储分配模型的，此处不作介绍。现在重点看下第3个参数：

```c++
class Compare = less<Key>
```

这也是一个class类型的，而且提供了默认值less<Key>。less是STL里面的一个函数对象，那么什么是函数对象呢？

所谓的函数对象，即调用操作符的类，其对象常称为函数对象（function object），它们是行为类似函数的对象。表现出一个函数的特征，就是通过“对象名+（参数列表）”的方式使用一个类，其实质是对operator（）操作符的重载。

现在来看一下less的实现：

```c++
template <class T> struct less : binary_function <T, T, bool> {
    bool operator() (const T& x, const T& y) const
    {return x < y;}
}
```

它是一个带模板的struct，里面仅仅对（）运算符进行了重载，实现很简单，但用起来很方便，这就是函数对象的优点所在。STL中还为四则运算等常见运算定义了这样的函数对象，与less相对的还有greater：

```c++
template <class T> struct greater : binary_function <T, T, bool> {
    bool operator() (const T& x, const T& y) const
    {return x > y;}
}
```

因此，要想让map中的元素按照key从大到小排序，可以如下代码所示：

```c++
map<string, int, greater<string> > mapStudent;
mapStudent["LiMin"]=90;
mapStudent["ZiLinMi"]=72;
mapStudent["BoB"]=79;
map<string, int>::iterator iter=mapStudent.begin();
for(iter=mapStudent.begin();iter!=mapStudent.end();iter++){
    cout<<iter->first<<" "<<iter->second<<endl;
}
```

只需指定它的第3个参数Compare，把默认的less指定为greater，即可达到按照key从大到小排序。现在知道如何为map指定Compare类了，如果想自己写一个Compare的类，让map按照想要的顺序来存储，比如按照学生姓名的长短排序进行存储，那么只要自己写一个函数对象，实现想要的逻辑，并在定义map的时候把Compare指定为自己编写的这个就可以实现了，代码如下：

```c++
struct CmpByKeyLength {  
  bool operator()(const string& k1, const string& k2) {  
    return k1.length() < k2.length();  
  }  
};

map<string, int, CmpByKeyLength > mapStudent;
mapStudent["LiMin"]=90;
mapStudent["ZiLinMi"]=72;
mapStudent["BoB"]=79;
map<string, int>::iterator iter=mapStudent.begin();
for(iter=mapStudent.begin();iter!=mapStudent.end();iter++){
    cout<<iter->first<<" "<<iter->second<<endl;
}
```

因此，想改变map的key排序方法，可以通过修改Compare函数实现。

key是结构体的，如果没有重载小于号（<）操作，就会导致insert函数在编译时就无法编译成功。其实，为了实现快速查找，map内部本身就是按序存储的（比如红黑树）。在插入<key，value>键值对时，就会按照key的大小顺序进行存储。这也是作为key的类型必须能够进行<运算比较的原因。

如下代码，mapStudent的key是StudentInfo类型的，要重载StudentInfo类型的<号，这样才能正常地插入到mapStudent中：

```c++
typedef struct tagStudentInfo  
{  
    int iID;  
    string  strName;  
	bool operator < (tagStudentInfo const& r) const {  
        //这个函数指定排序策略，按iID排序，如果iID相等的话，按strName排序  
        if(iID < r.iID)  return true;  
        if(iID == r.iID) return strName.compare(r.strName) < 0;  
        return false;
    }  
}StudentInfo;//学生信息

/*用学生信息映射分数*/  
map<StudentInfo, int>mapStudent;  
StudentInfo studentInfo;  
studentInfo.iID = 1;  
studentInfo.strName = "student_one";  
mapStudent[studentInfo]=90;
studentInfo.iID = 2;  
studentInfo.strName = "student_two"; 
mapStudent[studentInfo]=80;
map<StudentInfo, int>::iterator iter=mapStudent.begin();
for(;iter!=mapStudent.end();iter++){
    cout<<iter->first.iID<<" "<<iter->first.strName<<" "<<iter->second<<endl;
}
```

如果想按照map的value（第二个元素）排序，第一反应是利用stl中提供的sort算法实现，这个想法是好的，不幸的是，sort算法有个限制，利用sort算法只能对序列容器进行排序，只能是线性的（如vector、list、deque）。map也是一个集合容器，它里面存储的元素是pair，但是它不是线性存储的（如红黑树），所以利用sort不能直接和map结合进行排序。虽然不能直接用sort对map进行排序，那么可以间接进行，把map中的元素放到序列容器（如vector）中，然后再对这些元素进行排序呢？这个想法看似是可行的。要对序列容器中的元素进行排序，也有个必要条件：就是容器中的元素必须是可比较的，也就是实现了<操作的。那么现在就来看下map中的元素是否满足这个条件。

```c++
typedef pair<string, int> PAIR;   
bool cmp_by_value(const PAIR& lhs, const PAIR& rhs) {  
  return lhs.second < rhs.second;  
}  
struct CmpByValue {  
  bool operator()(const PAIR& lhs, const PAIR& rhs) {  
    return lhs.second < rhs.second;  
  }  
};

map<string, int> name_score_map;  
name_score_map["LiMin"] = 90;  
name_score_map["ZiLinMi"] = 79;  
name_score_map["BoB"] = 92;  
name_score_map.insert(make_pair("Bing",99));  
name_score_map.insert(make_pair("Albert",86));  
/*把map中元素转存到vector中*/   
vector<PAIR> name_score_vec(name_score_map.begin(), name_score_map.end());  
sort(name_score_vec.begin(), name_score_vec.end(), CmpByValue());  
/*sort(name_score_vec.begin(), name_score_vec.end(), cmp_by_value);也是可以的*/ 
for (int i = 0; i != name_score_vec.size(); ++i) {  
    cout<<name_score_vec[i].first<<" "<<name_score_vec[i].second<<endl;  
}
```

#### 1.2.6 map的原理

map内部自建一棵红黑树（一种非严格意义上的平衡二叉树），这棵树具有对数据自动排序的功能，所以在map内部所有的数据都是有序的。这里简单讲下红黑树的含义。

先来看下算法导论对R-B Tree的介绍：红黑树，一种二叉查找树，但在每个结点上增加一个存储位表示结点的颜色，可以是Red或Black。通过对任何一条从根到叶子的路径上各个结点着色方式的限制，红黑树确保没有一条路径会比其他路径长出两倍，因而是接近平衡的。红黑树，作为一棵二叉查找树，满足二叉查找树的一般性质。下面，来了解下二叉查找树的一般性质。

先来看下算法导论对R-B Tree的介绍：红黑树，一种二叉查找树，但在每个结点上增加一个存储位表示结点的颜色，可以是Red或Black。通过对任何一条从根到叶子的路径上各个结点着色方式的限制，红黑树确保没有一条路径会比其他路径长出两倍，因而是接近平衡的。红黑树，作为一棵二叉查找树，满足二叉查找树的一般性质。下面，来了解下二叉查找树的一般性质。

二叉查找树，也称有序二叉树（ordered binary tree），或已排序二叉树（sortedbinary tree），是指一棵空树或者具有下列性质的二叉树：①若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；②若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；③任意节点的左、右子树也分别为二叉查找树；④没有键值相等的节点（no duplicate node）。

因为一棵由n个结点随机构造的二叉查找树的高度为lgn，所以二叉查找树的一般操作的执行时间为O（lgn）。但二叉查找树若退化成了一棵具有n个结点的线性链后，则这些操作最坏情况运行时间为O（n）。红黑树虽然本质上是一棵二叉查找树，但它在二叉查找树的基础上增加了着色和相关的性质使得红黑树相对平衡，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为O（logn）。

它是如何保证一棵n个结点的红黑树的高度始终保持在logn的呢？这就引出了红黑树的5个性质：①每个结点要么是红的要么是黑的；②根结点是黑的；③每个叶结点（叶结点即指树尾端NIL指针或NULL结点）都是黑的；④如果一个结点是红的，那么它的两个儿子都是黑的；⑤对于任意结点而言，其到叶结点树尾端NIL指针的每条路径都包含相同数目的黑结点。

正是红黑树的这5条性质，使一棵n个结点的红黑树始终保持了logn的高度，从而也就解释了上面所说的“红黑树的查找、插入、删除的时间复杂度最坏为O（logn）”这一结论成立的原因。

当在对红黑树进行插入和删除等操作时，对树做了修改可能会破坏红黑树的性质。为了继续保持红黑树的性质，可以通过对结点进行重新着色，以及对树进行相关的旋转操作，即通过修改树中某些结点的颜色及指针结构，来达到对红黑树进行插入或删除结点等操作后继续保持它的性质或平衡的目的。