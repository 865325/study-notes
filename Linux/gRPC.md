### Protocol Buffers

#### 概述

Protocol Buffers 是一种与语言无关、与平台无关的可扩展机制，用于序列化结构化数据

它类似于 JSON，但体积更小、速度更快，并且会生成本机语言绑定。只需定义一次希望如何对数据进行结构化，然后即可特殊生成的源代码轻松地将结构化数据写入各种数据流并从中读取数据

#### C++使用

```bash
# 安装
sudo apt install protobuf-compiler
```

定义addressbook.proto，存放数据结构

```c++
syntax = "proto2";

// 类似命名空间
package tutorial;

// optional 可选字段
// repeated 可重复字段
message Person {
	optional string name = 1;
	optional int32 id = 2;
	optional string email = 3;

	enum PhoneType {
		MOBILE = 0;
		HOME = 1;
		WORK = 2;
	}

	message PhoneNumber {
		optional string number = 1;
		optional PhoneType type = 2 [default = HOME];
	}

	repeated PhoneNumber phones = 4;
}

message AddressBook {
	repeated Person people = 1;
}
```

```c++
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"

using namespace std;

// 该函数根据用户输入填充 Person 消息。
void PromptForAddress(tutorial::Person *person)
{
	cout << "Enter person ID number: ";
	int id;
	cin >> id;
	person->set_id(id);	   // 设置ID到 Person 对象中
	cin.ignore(256, '\n'); // 忽略换行符

	cout << "Enter name: ";
	getline(cin, *person->mutable_name()); // 读取姓名并设置到 Person 对象中

	cout << "Enter email address (blank for none): ";
	string email;
	getline(cin, email);
	if (!email.empty())
	{
		person->set_email(email); // 设置电子邮件
	}

	while (true) // 循环输入电话号码
	{
		cout << "Enter a phone number (or leave blank to finish): ";
		string number;
		getline(cin, number);
		if (number.empty())
		{
			break; // 退出循环
		}

		tutorial::Person::PhoneNumber *phone_number = person->add_phones(); // 创建并添加 PhoneNumber 对象
		phone_number->set_number(number);									// 设置电话号码

		cout << "Is this a mobile, home, or work phone? ";
		string type;
		getline(cin, type);
		if (type == "mobile")
		{
			phone_number->set_type(tutorial::Person::MOBILE);
		}
		else if (type == "home")
		{
			phone_number->set_type(tutorial::Person::HOME);
		}
		else if (type == "work")
		{
			phone_number->set_type(tutorial::Person::WORK);
		}
		else
		{
			cout << "Unknown phone type.  Using default." << endl;
		}
	}
}

// 输出联系人信息
void PrintAddressBook(const tutorial::AddressBook &address_book)
{
	for (int i = 0; i < address_book.people_size(); ++i)
	{
		const tutorial::Person &person = address_book.people(i);
		cout << "Person ID: " << person.id() << endl; // 输出ID
		cout << "Name: " << person.name() << endl;	  // 输出姓名
		if (person.has_email())
		{
			cout << "Email: " << person.email() << endl; // 输出电子邮件
		}

		// 输出电话号码
		for (int j = 0; j < person.phones_size(); ++j)
		{
			const tutorial::Person::PhoneNumber &phone_number = person.phones(j);
			cout << "Phone #" << j + 1 << ": " << phone_number.number() << " (";
			switch (phone_number.type())
			{
			case tutorial::Person::MOBILE:
				cout << "mobile";
				break;
			case tutorial::Person::HOME:
				cout << "home";
				break;
			case tutorial::Person::WORK:
				cout << "work";
				break;
			default:
				cout << "unknown";
				break;
			}
			cout << ")" << endl; // 输出电话号码类型
		}
		cout << endl; // 分隔不同联系人的输出
	}
}

// 主函数：从文件读取整个地址簿，
// 添加一个基于用户输入的联系人，然后将其写回到同一个文件。
int main(int argc, char *argv[])
{
	GOOGLE_PROTOBUF_VERIFY_VERSION; // 确认 Protobuf 版本匹配

	if (argc != 2)
	{
		cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
		return -1;
	}

	tutorial::AddressBook address_book; // 创建 AddressBook 对象

	{
		// 读取现有地址簿。
		fstream input(argv[1], ios::in | ios::binary); // 打开输入文件流
		if (!input)
		{
			cout << argv[1] << ": File not found.  Creating a new file." << endl;
		}
		else if (!address_book.ParseFromIstream(&input)) // 解析地址簿文件
		{
			cerr << "Failed to parse address book." << endl;
			return -1;
		}
	}

	// 输出读取到的联系人信息
	PrintAddressBook(address_book);

	// 允许用户输入新的联系人信息
	PromptForAddress(address_book.add_people()); // 调用函数，添加用户输入的地址信息

	{
		fstream output(argv[1], ios::out | ios::trunc | ios::binary);
		if (!address_book.SerializeToOstream(&output)) // 将地址簿序列化并写入文件
		{
			cerr << "Failed to write address book." << endl;
			return -1;
		}
	}

	google::protobuf::ShutdownProtobufLibrary(); // 清理 Protobuf 库的全局对象

	return 0; // 正常结束程序
}
```

```cmake
# 指定构建项目所需的最小 CMake 版本为 2.8
cmake_minimum_required(VERSION 2.8)

# 定义项目名称为 Person
project(Person)

# 查找 Protobuf 库，如果未找到，则停止构建
find_package(Protobuf REQUIRED)

# 检查 Protobuf 库是否被找到
if (PROTOBUF_FOUND)
	message(STATUS "protobuf library found")  # 输出消息，表明 Protobuf 库已找到
else ()
	message(FATAL_ERROR "protobuf library is needed but cant be found")  # 如果未找到，输出致命错误消息并停止构建
endif ()

# 包含 Protobuf 的头文件目录和当前二进制目录（通常是构建目录）
include_directories(${PROTOBUF_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# 根据 addressbook.proto 文件生成 C++ 源文件和头文件
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS addressbook.proto)

# 输出生成的源文件和头文件的列表，便于调试
message(${PROTO_SRCS} ${PROTO_HDRS})

# 定义可执行文件目标 Person，包含 Person.cpp 和生成的 proto 文件
add_executable(Person Person.cpp ${PROTO_SRCS} ${PROTO_HDRS})

# 将 Protobuf 库链接到可执行文件 Person
target_link_libraries(Person ${PROTOBUF_LIBRARIES})
```

### gRPC
