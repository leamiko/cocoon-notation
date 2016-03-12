# 蚕茧表示法

Cocoon Notation

一种易理解易书写的数据结构表示方法，像抽丝剥茧一样层层剥开一个多层次的像蚕茧一样的复杂对象，常用于数据建模，接口描述，代码注释等。

下面将以实例的方式描述表示规则，并且给出了Javascript和C++两种实现供参考。

## 对象(object)

也称为结构体(structure)或类(class).

一个对象Customer，有两个字段（或称属性）id和name，则可以表示为

	Customer: {id, name}

左侧是类型名(type name)，一般用大写字母开头。右侧是类型描述，使用花括号描述对象类型，形式为 `{field1, field2, ...}`, 括号内是各属性名，以小写字母开头。

Javascript中该对象像这样：

	var customer = {id: 100, name: "john"};

而C++中会这样定义结构:

	struct Customer 
	{
		int id;
		string name;
	};

对象中可包含复杂类型的属性，属性名中使用前缀"%"标明结构体或字典，用前缀"@"标明数组，如：

	Customer: {id, name, %address, @orders}
	Address: {country, city}
	Order: {id, total, @lines=[OrderLine]}
	OrderLine: {itemId, qty, price}

Customer对象的属性address是一个结构体，用前缀"%"标明；属性orders是一个数组，用前缀"@"标明。
这里属性address和orders没有定义类型名，这意味着类型名与属性名同名，即属性address的类型为Address, 而orders的类型为Orders。
因为类型Orders未定义，但定义了其单数形式Order类型，这隐含着Orders是Order的数组。
如果写完整则应像这样描述：

	Customer: {id, name, %address=Address, @orders=[Order]}

将Order放在中括号中，即表示属性orders是Order类型的数组。
Order类型的属性lines描述为

	@lines=[OrderLine]
	
表示它是OrderLine的数组。

属性可以缺省，还可指定缺省值，如：

	Address: {country?=CN, city?}

这表示属性country可以没有，如果没有的话则当作值为"CN"。

## 字典(dictionary)

它是一个集合，其中每一项是一个键值对(key-value pair)。也称哈希表(hash table)或映射表(map)。

使用花括号描述字典，形式为 `{key => value}`，其中value可以是基本类型或复杂类型。
字典类型的命名应使用复数形式，或以"List", "Map"结尾，如：

	Customers: { id => name }

上面定义表示，Customers是一个字典类型，每一项中主键表示id字段，而值表示name字段（基本类型）。

在Javascript中的字典类型像这样：

	var customers = {100: "john", 101: "bella"};

注意：虽然与对象类型形式上一样，但含义完全不同。它是对象的集合，名称也是复数形式。

在C++中一般这样定义：

	typedef map<int, string> Customers;

字典中每项的值也可以是一个对象，表示为：

	Customers: {id=>Customer}
	Customer: {id, name}

或可以合并为：

	Customers: { id=> {id, name} }

在Javascript中使用：

	var customers = {
		100: {id: 100, name: "john"},
		101: {id: 101, name: "bella"}
	};

在C++中可以这样定义类型：

	struct Customer 
	{
		int id;
		string name;
	};
	typedef map<int, Customer> Customers;


## 数组（Array）

又称顺序表或列表。

使用中括号描述数组，形式为 `[ element ]`。命名时应使用复数形式，或以"List", "Arr"结尾。

要表示一个简单的列表IdList，它的每个元素是一个id，可以表示为

	IdList: [id]

Customers是Customer的数组，可以表示为：

	Customers: [Customer]
	Customer: {id, name}

或合并为：

	Customers: [ {id, name} ]

在Javascript中数组类型的变量：

	var customers = [ {id: 100, name: "john"}, {id: 101, name: "bella"} ]

在C++中类型常定义为:

	struct Customer 
	{
		int id;
		string name;
	};
	typedef vector<Customer> Customers;


如果定义了Customer, 而没有定义复数形式Customers, 则默认Customers是它的数组；反之，如果定义了复数形式Customers/CustomerList/CustomerMap/CustomerArr, 则Customer可以不定义，默认它是前者集合中的值元素。

有时可以用数组替代对象类型，比如Customer有两个元素，第一个是id, 第二个是name, 则可以表示为：

	Customer: [id, name]

用数组定义对象时，形式为 `[ field1, field2, ... ]`, 它至少应有两个字段。

在Javascript中的例子：

	var customer1 = [100, "john"];
	var customer2 = [101, "bella"];

在C++中一般不这样设计，非要用的话也可以用Variant之类的通用类型：

	typedef Vector<Variant> Customer; // [id, name]
	Customer customer1 = {Variant(100), Variant("john")};


## 基本类型描述

一个复杂类型最终应分解到它所有的属性，要么是基本类型，要么是已分解到基本类型的复杂类型。

在定义属性时，应通过属性名暗示类型，或通过后缀标识符标明。规则如下：

- Integer: 后缀标识符为"&", 或以"Id", "Cnt"等结尾, 如 customerId, age&
- Double: 后缀标识符为"#", 如 avgValue#
- Currency: 后缀标识符为"@", 或以"Price", "Total", "Qty", "Amount"结尾, 如 unitPrice, price2@。
- Datetime/Date/Time: 分别以"Tm"/"Dt"/"Time"结尾，如 tm 可表示日期时间如"2010-1-1 9:00"，comeDt 只表示日期如"2010-1-1"，而 comeTime只表示时间如"9:00"
- Boolean/TinyInt(1-byte): 以Flag结尾, 或以is开头.
- String: 未显示指明的一般都作为字符串类型。

在根据设计实现时，应将上述类型对应到所用编程语言所支持的实际类型。

例如以下定义：

	Customer: {id, name, age&, score#, balance@, specialPrice, createTm, isNew}

简明地定义了Customer各属性的基本类型，在使用C++定义类型时，可能是这样：

	class Money {};

	struct Customer
	{
		int id;
		string name;
		int age;
		double score;
		Money balance;
		Money specialPrice;
		time_t createTm;
		bool isNew;
	};

