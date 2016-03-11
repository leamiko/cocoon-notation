# 蚕茧标记法

Cocoon Notation

一种易理解易书写的快速数据结构标记方法，像抽丝剥茧一样层层解析一个对象，常用于数据建模，接口描述，代码注释等。因常用中括号和花括号，又称括号标记法。

下面将以实例的方式描述标记规则，并且给出了Javascript和C++两种实现供参考。

## 结构体(structure)

Javascript语言称为对象(object).

一个对象Customer，有两个字段（或称属性）id和name，则可以表示为

	Customer: {id, name}

Javascript中这样使用：

	var customer = {id: 100, name: "john"};

C++中这样定义:

	struct Customer 
	{
		int id;
		string name;
	};

结构体中可包含其它复杂类型，如

	Customer: {id, name, %address, @orders}
	Address: {country, city}
	Order: {id, total, @lines=[OrderLine]}
	OrderLine: {itemId, qty, price}

Customer中的Address是一个结构体，用前缀"%"标明(如果是字典，也用"%"标注)；属性orders是一个数组，用前缀"@"标明；这里因为属性为与接下来的类型名同名，所以省略了类型，如果写完整则是这样：

	Customer: {id, name, %address=Address, @orders=[Order]}

其中Order在中括号中表示是Order类型的数组。

属性可以缺省，还可指定缺省值：

	Address: {country?=CN, city?}

规定：

- 使用花括号描述对象，形式为 {field1, field2, ...}
- 属性用前缀"%"标明结构体或字典，用前缀"@"标明数组。

## 字典(dictionary)

也称哈希表(hash table)或映射表(map)。表中每一项是一个键值对(key-value pair)。

	Customers: { id => name }

上面定义表示，Customers这个对象是一个字典，在Javascript中这样使用：

	var customers = {100: "john", 101: "bella"};

在C++中这样定义：

	typedef map<int, string> Customers;

映射表中每项的值也可以是一个对象：

	Customers: {id=>Customer}
	Customer: {id, name}

或可以简写为：

	Customers: { id=> {id, name} }

在Javascript中使用：

	var customers = {
		100: {id: 100, name: "john"},
		101: {id: 101, name: "bella"}
	};

在C++中可以这样定义：

	struct Customer 
	{
		int id;
		string name;
	};
	typedef map<int, Customer> Customers;

规定：

- 使用花括号描述字典，形式为 {key => value}，其中value可以是任何复杂结构。

建议：

- 映射表对象命名应使用复数形式，或以"List", "Map"结尾。

## 数组（Array）

又称顺序表或列表。

要表示一个简单的列表IdList，它的每个元素是一个id，可以表示为

	IdList: [id]

Customers是Customer的数组，可以表示为：

	Customers: [Customer]
	Customer: {id, name}

或合并为：

	Customers: [ {id, name} ]

在Javascript中：

	var customers = [ {id: 100, name: "john"}, {id: 101, name: "bella"} ]

在C++中:

	struct Customer 
	{
		int id;
		string name;
	};
	typedef vector<Customer> Customers;

规定：

- 使用中括号描述数组，形式为 [ element ]

建议：

- 命名时应使用复数形式，或以"List", "Arr"结尾。
- 如果定义了Customer, 则默认Customers是它的数组，可以不定义直接使用；如果定义了Customers/CustomerList/CustomerMap/CustomerArr, 则Customer可以不定义，默认它是前者的元素。

有时可以用数组替代对象，比如Customer有两个元素，第一个是id, 第二个是name, 则可以表示为：

	Customer: [id, name]

在Javascript中：

	var customer1 = [100, "john"];
	var customer2 = [101, "bella"];

在C++中一般不这样设计，非要用的话也可以用Variant之类的通用类型：

	typedef Vector<Variant> Customer; // [id, name]
	Customer customer1 = {Variant(100), Variant("john")};

规定：

- 用数组定义对象时，形式为 [ field1, field2, ... ], 它至少应有两个字段。

## 基本类型描述

建议：

根据需要可不标示基本类型，或通过名字后缀反映类型，或通过后缀符号显示标明。规则如下：

- Integer: 标识符为"&", 或以"Id", "Cnt"等结尾, 如 customerId, age&
- Double: 标识符为"#", 如 avgValue#
- Currency: 标识符为"@", 以"Price", "Total", "Qty", "Amount"结尾, 如 unitPrice, price2@。如果没有Currency类型也可以用Double替代。
- Datetime/Date/Time: 分别以"Tm"/"Dt"/"Time"结尾，如 tm 可表示日期时间如"2010-1-1 9:00"，comeDt 只表示日期如"2010-1-1"，而 comeTime只表示时间如"9:00"
- Boolean/TinyInt(1B): 以Flag结尾, 或以is开头.
- 未显示指明的一般是字符串类型。

例如以下定义：

	Customer: {id, name, age&, score#, balance@, specialPrice, createTm, isNew}

简明地定义了基本类型。

