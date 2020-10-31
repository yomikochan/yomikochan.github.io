---
layout: page
title: 轻量级Json解析器
date: 2018-07-21 13:38:00+8
tags:
  - C/C++
  - Json
categories:
  - 词法分析树
cover_index: /assets/1632277878.jpg
---

# 更新日志

该项目更新日志历史按最新时间排序。

### Update 2018-10-12

**此次更新的主要内容**

1. 修复了`TextChecker()`、`Erase()`、`ToString()`的bug;
2. 增加了`ToHttpRequestHeader()`功能，用于将json文本解析为http协议请求头部格式；
3. `Text`类中增加了`RemovePair()`功能，与`Journey`类中的`Erase()`功能相同，都是用于移除一对`key-value`。

### Update 2018-10-09

**此次更新的主要内容**

1. 增加了`TextChecker()`api，其作用是检测json文本是否合法。用法
```C++
Text t;
const char*s = "{\"hello\":\"world\",\"\":{\"112\":\"few\",\"er\":[1,2,3]}}";
std::cout << (t.TextChecker(s) ? "true" : "false") << std::endl;
```
2. 改变了一些`api`的名称，比如之前的`JsonFormatPrint`改为了`Output`、`WriteText`改为了`WriteToText`、`Tojson`改为了`ToString`，两个类中获取对象名称分别改为了`GetValueIsObject`、`GetPair`等。
3. 增加了生成器`Journey`类，将基础类型值数据结构从`Value类`中拿了出来，并做了许多赋值运算符`=`与中括符`[]`的重载。使得使用更方便。用法见：[生成器用例](https://github.com/darkchii/C-libhttp/blob/master/exam_json_1.cpp)

接下来打算添加迭代器。

源码见：[libjson.hpp](https://github.com/darkchii/C-libhttp/blob/master/libjson.hpp)

### Update 2018-10-07

### pm 12:23

**此次更新的主要内容**

1. 增加了对解析好的文本的可操作性（增删改查等），示例见：[example](https://github.com/darkchii/C-libhttp/blob/master/exam_json.cpp)
2. 完善了之前一些没注意到的问题。

### Update 2018-10-05

### pm 02:52

**此次更新的主要内容**

1. 完善了之前一些没注意到的问题。

### Update 2018-10-05

### pm 07:44

**此次更新的主要内容**

1. 重新设计了数据结构，让数据使用变得更安全；
2. 修复了字符串转浮点型算法的bug，目前是这样的:
```
double str2dou(std::string s)
{
	double num = 0, point_back = 0, sign = 1;
	size_t i = 0, pos = 0;
	if (s[i] == '-') i++, sign *= -1;
	while (s[i] != '.') num = num * 10 + (s[i++] - '0');
	pos = i++;
	while (s[i]) point_back = point_back * 10 + (s[i++] - '0');
	num += point_back / std::pow(10, i - pos - 1);
	return num * sign;
}
```
3. 更换了解析方式，之前的解析的json格式：
```
{
	"string": "hello",
	"double": "-123.43",
	"int": "10",
	"bool": "true",
	"null": "null",
	"array": "["2", "3", "4"]"
}
```
* 现在的：
```
{
	"string": "hello",
	"double": -123.43,
	"int": 10,
	"bool": true,
	"null": null,
	"array": [2, 3, 4]
}
```
4. 去掉一些多余的变量；
5. 解析器的使用：
```C++
#include <iostream>
#include <fstream>
#include <string>
#include "libjson.h"
using namespace std;
int main()
{
	Text t;
	fstream text("json.txt", fstream::in | fstream::out | fstream::app);
	string line, str;
	while (getline(text, line))
		str += line;
	text.close();
	t.Parse(str);
	t.JsonFormatPrint();
	return 0;
}
```
* output:
```
{
        "Latin_Modern" : "fn_lm",
        "Verdana" : "fn_jvn",
        "time" : -123.23,
        "flag" : true
}
```
6. 决定将解析器与生成器分离开，这样更易于使用和维护。（目前基本上完成了解析器部分，生成器正在进行中）

### Update 2018-09-28

### pm 07:55

**此次更新的主要内容**

1. 利用其他方式完善了 `add()`、`setvalue()`，还有 `operator[]`，解决了不同类型的传值的增改功能（虽然比较方法比较笨拙）。
2. 完善了不同类型转换为 `string` 的算法，这些都是是辅助算法，应考虑单独放在一个头文件中调用。

需要注意的地方：

1. `add`、`setvalue`函数只能传入两个字符串，不管 `value` 是什么类型都使用字符串进行传入。
用例：
```C++
r.add("string", "yes"); // `value`为字符串类型
r.set("string", "no"); // 同上

r.add("array", "[\"one\", \"two\", \"three\", \"four\", \"five\"]"); // `value`为字符串数组类型
r.setvalue("array", "[\"Json\", \"Test\", \"Fuck\"]"); // 同上

r.add("double", "324.23"); // `value`为浮点类型
r.setvalue(...); // 同上

r.add("bool", "true"); // `value`为布尔类型
r.setvalue(...); // 同上

r.add("root", "{\"parent\":\"children\"}")// `value`为Json对象类型
r.setvalue(..); // 同上
```
2. 通过 `[]`，获取的 `value` 不是原地址，所以不能进行赋值或修改。也暂不能获取对象类型。`getvalue()` 暂且保留原样。

```C++
#include <iostream>
#include <fstream>
#include <vector>
#include <math.h>
#include <string>
#define st size_t
#define ll long long
#define index(c) (c - 'A')
const int N = 1000010;
typedef enum { BOOL, INTEGER, FLOAT, STRING, ARRAY, OBJECT } Type;
st weight(const char*s)
{
    st i = 0, x = 0;
    while (s[i]) x += index(s[i])*i, i += 1;
    return x + i;
}
void bits(char s[], st &i, ll x)
{
    if (!x) return ;
    bits(s, i, x/10);
    s[i++] = x%10 + '0';
}
void bits(std::string &s, st &i, ll x)
{
    if (!x) return ;
    bits(s, i, x/10);
    s[i++] = x%10 + '0';
}
bool str2bool(const char *s)
{
	bool var;
	if (s[0] == 't' && s[1] == 'r' && s[2] == 'u' && s[3] == 'e' && s[4] == '\0') var = true;
	else var = false;
	return var;
}
const char* bool2con_ch_arr(bool v)
{
    if (v) return "true";
    return "false";
}
ll con_char2int(const char *s)
{
    ll num = 0, sign = 1;
    if (*s == '-') s++, sign *= -1;
	while (*s) num = num * 10 + (*s - '0');
	return num*sign;
}
ll str2int(std::string s)
{
    return con_char2int(s.c_str());
}
std::string int2str(ll num)
{
    std::string s; st i = 0;
    if (num < 0) s += "-", i++;
    bits(s, i, num);
    return s;
}
const char* int2con_ch_arr(ll num)
{
    return int2str(num).c_str();
}
double str2float(const char*s, st p, st r)
{
	double num = 0, propoint = 0, sign = 1;
	if (*s == '-') s++, sign *= -1;
	while (*s != '.') num = num * 10 + (*s++ - '0');
	s++;
	while (*s) propoint = propoint * 10 + (*s++ - '0');
	num += propoint / std::pow(10, r - p);
	return num*sign;
}
double str2dou(const char*s) // 与 str2float() 功能一样
{
    double num = 0, propoint = 0, sign = 1;
    st i = 0, pos = 0;
	if (s[i] == '-') i++, sign *= -1;
	while (s[i] != '.') num = num * 10 + (s[i] - '0'), pos = i++;
	i++;
	while (s[i]) propoint = propoint * 10 + (s[i] - '0');
	num += propoint / std::pow(10, i - pos);
	return num*sign;
}
char* float2ch_arr(double num)
{
    char*s = new char; st i = 0;
    if (num < 0) s[i++] = '-', num *= -1;
    ll x = (ll)num;
    double y = num - x;
    bits(s, i, x);
    s[i++] = '.';
    do {
        y *= 10;
        s[i++] = int(y + 0.1)%10 + '0';
    } while (int(ceil(y)) != int(y + 0.1));
    s[i] = 0;
    return s;
}
const char* float2con_ch_arr(double num)
{
    return (const char*)float2ch_arr(num);
}
std::string float2str(double num)
{
    std::string s(float2con_ch_arr(num));
    return s;
}
void str2arr(std::vector<std::string>& arr_str, const char*s)
{
    std::string str; st i = 0;
    while (*s)
    {
        switch(*s)
        {
            case ',':
                if (i > 0)
                {
                    str[i] = '\0';
                    arr_str.push_back(str);
                    i = 0;
                }
                break;
            case ' ': break;
            default : str[i++] = *s; break;
        }
        s++;
    }
}
const char* arr2con_ch_arr(std::vector<std::string> arr_str)
{
    char*s = new char; st k = 0;
    for (st i = 0; i < arr_str.size(); i++)
    {
        for (st j = 0; j < arr_str[i].size(); j++)
            s[k++] = arr_str[i][j];
        if (i != arr_str.size() - 1) s[k++] = ',';
    }
    s[k] = 0;
    return (const char*)s;
}
class BaseNode {
public:
	bool b_v;
	ll i_v;
	double d_v;
	std::string s_v;
	std::vector<std::string> arr_v;

	std::string key;
	st keys;
	bool isvalue;
	Type type;
	ll table[N];
	BaseNode *children[N], *parent;

	BaseNode()
	{
		keys = 0, isvalue = false, type = OBJECT, parent = nullptr;
		for (int i = 0; i < N; i++) table[i] = -1, children[i] = nullptr;
	}
	BaseNode(BaseNode*p, const char *k)
	{
		std::string sk(k);
		key = sk, keys = 0, isvalue = false, parent = p;
		for (int i = 0; i < N; i++) table[i] = -1, children[i] = nullptr;
	}
	~BaseNode() {}
};
class JsonTree {
private:
	BaseNode *root;
	std::string pstr;
	/**/
	void print(BaseNode*r, int format)
	{
		if (r)
		{
			std::cout << '{' << std::endl;
			for (st i = 0; i < r->keys; i++)
			{
				BaseNode*node = r->children[i];
				if (node)
				{
				    for (int j = 0; j < format; j++)
                        std::cout << '\t';
					std::cout << "\""  << node->key << "\" : ";
					switch (node->type)
					{
					case BOOL:
						std::cout << (node->b_v ? "true" : "false");
						break;
					case INTEGER:
						std::cout << node->i_v;
						break;
					case FLOAT:
						std::cout << node->d_v;
						break;
					case STRING:
						std::cout << "\"" << node->s_v << "\"";
						break;
					case ARRAY:
						std::cout << '[';
						for (st j = 0; j < node->arr_v.size(); j++)
						{
							std::cout << "\"" << node->arr_v[j] << "\"";
							if (j != node->arr_v.size() - 1) std::cout << ", ";
						}
						std::cout << ']';
						break;
					case OBJECT:
						print(r->children[i], format + 1);
						break;
					}
					if (i != root->keys - 1) std::cout << ",";
					std::cout << std::endl;
				}
			}
			for (int j = 0; j < format - 1; j++)
                std::cout << '\t';
			std::cout << '}' << std::endl;
			for (int j = 0; j < format - 1; j++)
                std::cout << '\t';
		}
	}
	BaseNode* findkey(BaseNode*r, const char*s)
	{
	    if (r && r->table[weight(s)] != -1) return r->children[r->table[weight(s)]];
        return nullptr;
	}
	void destory(BaseNode*&r)
	{
		if (r)
		{
			for (st i = 0; i < r->keys; i++)
				destory(r->children[i]);
			delete r;
		}
	}
public:
	JsonTree() { root = nullptr; }
	JsonTree(const char* s)
	{
		root = nullptr;
		parse(s);
	}
	JsonTree(std::string s)
	{
		root = nullptr;
		parse(s.c_str());
	}
	~JsonTree()
	{
		destory(root);
	}
	void parse(std::string s)
	{
	    parse(s.c_str());
	}
	void parse(const char *s)
	{
		char str[N]; st i = 0;
		bool arr_sw = false, obj_sw = false, str_sw = false;
		if (!root) root = new BaseNode();
		BaseNode *nd = root;
		while (*s)
		{
			switch (*s)
			{
			case '{': if (obj_sw) { nd = nd->children[nd->keys - 1]; nd->type = OBJECT; } break;
			case '}': nd->isvalue = false; nd = nd->parent; obj_sw = false; break;
			case '"': if (str_sw) str_sw = false; else str_sw = true;
			    if (i > 0)
                {
                    bool b = false; double dounum = 0;
                    ll intnum = 0; st f = 0, pos = 0;
                    str[i] = '\0';
                    if (!arr_sw)
                    {
                        if (i < 11)
                        {
                            if ((str[0] == 't' && str[1] == 'r' && str[2] == 'u' && str[3] == 'e' && str[4] == '\0') ||
                                (str[0] == 'f' && str[1] == 'a' && str[2] == 'l' && str[3] == 's' && str[4] == 'e' && str[5] == '\0'))
                                f = 3, b = str2bool(str);
                            else
                            {
                                for (st j = 0; j < i; j++)
                                    if (str[j] == '.' || str[0] == '-' || (str[j] >= '0' && str[j] <= '9')) f = 1;
                                    else { f = 0; break; }
                                if (f)
                                {
                                    for (st j = 0; j < i; j++)
                                        if (str[j] != '.' || (str[j] >= '0' && str[j] <= '9')) f = 2;
                                        else  { f = 1, pos = j; break; }
                                    if (f == 2) intnum = str2int(str);
                                    else dounum = str2float(str, pos, i - 1);
                                }
                            }
                        }
                    }
                    else f = 4;
                    if (nd->isvalue)
                    {
                        if (nd->children[nd->keys - 1])
                        {
                            std::string ps(str);
                            switch (f)
                            {
                            case 0:
                                nd->children[nd->keys - 1]->type = STRING;
                                nd->children[nd->keys - 1]->s_v = ps;
                                break;
                            case 1:
                                nd->children[nd->keys - 1]->type = FLOAT;
                                nd->children[nd->keys - 1]->d_v = dounum;
                                break;
                            case 2:
                                nd->children[nd->keys - 1]->type = INTEGER;
                                nd->children[nd->keys - 1]->i_v = intnum;
                                break;
                            case 3:
                                nd->children[nd->keys - 1]->type = BOOL;
                                nd->children[nd->keys - 1]->b_v = b;
                                break;
                            case 4:
                                nd->children[nd->keys - 1]->type = ARRAY;
                                nd->children[nd->keys - 1]->arr_v.push_back(ps);
                                break;
                            }
                        }
                    }
                    else
                    {
                        if (nd->table[weight(str)] == -1)
                        {
                            BaseNode *n = new BaseNode(nd, str);
                            nd->children[nd->keys] = n;
                        }
                    }
                    i = 0, obj_sw = true;
                }
                break;
			case ':': if (nd->table[weight(str)] == -1) nd->table[weight(str)] = nd->keys++; nd->isvalue = true; break;
			case ',': if (!arr_sw) { nd->isvalue = false; if (str_sw) str[i++] = *s; } break;
			case ' ': if (!arr_sw) if (str_sw) str[i++] = *s; break;
			case '[': arr_sw = true; break;
			case ']': arr_sw = false; break;
			case '/': if (str_sw) str[i++] = *s; break;
			case '\n': break;
			default:  str[i++] = *s; break;
			}
			s++;
		}
	}
	std::string parse2url(const char*s)
	{
		parse(s);
		for (st i = 0; i < root->keys; i++)
		{
		    BaseNode*node = root->children[i];
			std::string str = node->key;
			for (st j = 0; j < str.size(); j++)
				pstr += str[j];
			pstr += "=";
			str.clear();
			switch(node->type)
			{
            case BOOL: if (node->b_v) str = "true"; else str = "false";
                break;
            case INTEGER: str = int2str(node->i_v);
                break;
            case FLOAT: str = float2str(node->d_v);
                break;
            case STRING: str = node->s_v;
                break;
            default: std::cerr << "String parse error!" << std::endl;
                return "";
			}
			for (st j = 0; j < str.size(); j++)
				pstr += str[j];
			if (i != root->keys - 1)
				pstr += "&";
		}
		return pstr;
	}
	auto getvalue(const char*key)
	{
	    BaseNode*node = findkey(root, key);
	    if (node)
        {
//            switch(node->type)
//            {
//            case BOOL: return node->b_v;
//            case INTEGER: return node->i_v;
//            case FLOAT: return node->d_v;
//            case STRING: return node->s_v;
//            case ARRAY: return node->arr_v;
//            default: return node->children[node->table[weight(key)]];
//            }
            return node->d_v;
        }
        return 0.0;
	}
	auto operator[](const char*key)
	{
	    BaseNode*node = findkey(root, key);
	    if (node)
        {
            switch(node->type)
            {
            case BOOL: return bool2con_ch_arr(node->b_v);
            case INTEGER: return int2con_ch_arr(node->i_v);
            case FLOAT: return float2con_ch_arr(node->d_v);
            case STRING: return node->s_v.c_str();
            case ARRAY: return arr2con_ch_arr(node->arr_v);
            default: return "";
            // default: return node->children[node->table[weight(key)]];
            }
        }
        return "";
	}
	template<class T> T& operator=(T v)
	{
		return v;
	}
	void setvalue(const char*key, const char*value)
	{
        BaseNode*node = findkey(root, key);
        if (node)
        {
            std::string document, k(key), v(value);
            document += "{\"", document += k, document += "\":";
            document += "\"" , document += v, document += "\"}";
            parse(document.c_str());
        }
        else
        {
            std::cerr << "Not found the key!" << std::endl;
            return;
        }
	}
	void add(const char*key, const char*value)
	{
	    BaseNode*node = findkey(root, key);
        if (!node)
        {
            std::string document, k(key), v(value);
            document += "{\"", document += k, document += "\":";
            document += "\"" , document += v, document += "\"}";
            parse(document.c_str());
        }
        else
        {
            std::cerr << "The key was existed!" << std::endl;
            return;
        }
	}
	void erase(const char*key)
	{
	    BaseNode*node = findkey(root, key);
        if (node)
        {
            BaseNode*p = node->parent;
            p->keys--;
            p->table[weight(node->key.c_str())] = -1;
            destory(node);
        }
        else
        {
            std::cerr << "Not found the key!" << std::endl;
            return;
        }
	}
	void visual()
	{
		print(root, 1);
	}
};
int main()
{
    JsonTree r;

    std::fstream context("json.txt", std::fstream::in | std::fstream::out | std::fstream::app);
    std::string line, str;
    while (std::getline(context, line))
        str += line;
    context.close();

    r.parse(str);

    r.add("hello", "world");
    std::cout << r["hello"] << std::endl;
    r.visual();

    r.setvalue("hello", "[\"Json\", \"Test\", \"Fuck\"]");
    std::cout << r["hello"] << std::endl;
    std::cout << r["time"] << std::endl;
    r.visual();
	return 0;
}
```

output:

```bash
world
{
        "Latin_Modern" : "fn_lm",
        "Verdana" : "fn_jvn",
        "Object" : {
                "Color" : ["red", "blue", "green", "white"],
        }
        ,
        "time" : -123.23,
        "flag" : true,
        "hello" : "world"
}
Json,Test,Fuck
-123.230000000(
{
        "Latin_Modern" : "fn_lm",
        "Verdana" : "fn_jvn",
        "Object" : {
                "Color" : ["red", "blue", "green", "white"],
        }
        ,
        "time" : -123.23,
        "flag" : true,
        "hello" : ["Json", "Test", "Fuck"]
}
{
        "Latin_Modern" : "fn_lm",
        "Verdana" : "fn_jvn",
        "Object" : {
                "Color" : ["red", "blue", "green", "white"],
        }
        ,
        "time" : -123.23,
        "flag" : true,
        "hello" : ["Json", "Test", "Fuck"],
        "root" : {
                "parent" : "children",
        }

}
```

### am 04:00 左右

**此次更新的主要内容**

1. 修改了字符串加权函数名 `has()` 为 `weight()`，这样更合适。
2. 前面代码中还有一个明显的安全隐患，那就是 `weight()` 函数的设计，因为考虑到一些字符的`ascii`码比字符`A`小，会出现负数，这时会出现数组越界问题。但我考虑暂不做更改，因为我默认所有的 `key` 都不会出现那些奇怪的字符。
3. 完善了一些不合理的地方。

##### 个人遇到的一些技术难点

为了做出更好的`Json解析器`查了许多资料，但我有许多问题得不到解答。在`C++`中，不能对传参相同，返回类型不同的函数进行重载，模板也不能自动对返回类型进行推断（除非使用 C++11 auto 以及 C++14 编译器），这样我很难完成下面这样的功能：
```C++
container n;
n["one"] = 1;
cout << n["one"] << endl;
```
即赋值和索引功能。不能自动推断返回值类型，模板就没意义了。。。也许是我的数据结构没设计好？

##### 代码

```C++
#include <iostream>
#include <fstream>
#include <vector>
#include <math.h>
#include <string>
#define st size_t
#define ll long long
#define index(c) (c - 'A')
const int N = 1000010;
typedef enum { BOOL, INTEGER, FLOAT, STRING, ARRAY, OBJECT } Type;
st weight(const char*s)
{
	st i = 0, x = 0;
	while (s[i]) x += index(s[i])*i, i += 1;
	return x + i;
}
double str2float(const char *s, st p, st r)
{
	double num = 0, propoint = 0, sign = 1;
	if (*s == '-') s++, sign *= -1;
	while (*s != '.') num = num * 10 + (*s++ - '0');
	s++;
	while (*s) propoint = propoint * 10 + (*s++ - '0');
	num += propoint / std::pow(10, r - p);
	return num * sign;
}
void bits(char s[], st &i, ll x)
{
	if (!x) return;
	bits(s, i, x / 10);
	s[i++] = x % 10 + '0';
}
void bits(std::string &s, st &i, ll x)
{
	if (!x) return;
	bits(s, i, x / 10);
	s[i++] = x % 10 + '0';
}
std::string float2str(double num)
{
	ll x = (ll)num; st i = 0;
	double y = num - x;
	std::string s;
	bits(s, i, x);
	s[i++] = '.';
	do {
		y *= 10;
		s[i++] = int(y + 0.1) % 10 + '0';
	} while (int(ceil(y)) != int(y + 0.1));
	s[i] = 0;
	return s;
}
char* float2ch_arr(double num)
{
	ll x = (ll)num; st i = 0;
	double y = num - x;
	char* s = new char[N >> 1];
	bits(s, i, x);
	s[i++] = '.';
	do {
		y *= 10;
		s[i++] = int(y + 0.1) % 10 + '0';
	} while (int(ceil(y)) != int(y + 0.1));
	s[i] = 0;
	return s;
}
ll con_char2int(const char *s)
{
	ll num = 0, sign = 1;
	if (*s == '-') s++, sign *= -1;
	while (*s) num = num * 10 + (*s - '0');
	return num * sign;
}
ll str2int(std::string s)
{
	return con_char2int(s.c_str());
}
std::string int2str(ll num)
{
	std::string s; st i = 0;
	if (num < 0) s += "-", i++;
	bits(s, i, num);
	return s;
}
bool str2bool(const char *s)
{
	bool var;
	if (s[0] == 't' && s[1] == 'r' && s[2] == 'u' && s[3] == 'e' && s[4] == '\0') var = true;
	else var = false;
	return var;
}
class BaseNode {
public:
	bool b_v;
	ll i_v;
	double d_v;
	std::string s_v;
	std::vector<std::string> arr_v;

	std::string key;
	st keys;
	bool isvalue;
	Type type;
	ll table[N];
	BaseNode *children[N], *parent;

	BaseNode()
	{
		keys = 0, isvalue = false, type = OBJECT, parent = nullptr;
		for (int i = 0; i < N; i++) table[i] = -1, children[i] = nullptr;
	}
	BaseNode(BaseNode*p, const char *k)
	{
		std::string sk(k);
		key = sk, keys = 0, isvalue = false, parent = p;
		for (int i = 0; i < N; i++) table[i] = -1, children[i] = nullptr;
	}
	~BaseNode() {}
};
class JsonTree {
private:
	BaseNode *root;
	std::string pstr;
	/**/
	void print(BaseNode*r, int format)
	{
		if (r)
		{
			std::cout << '{' << std::endl;
			for (st i = 0; i < r->keys; i++)
			{
				BaseNode*node = r->children[i];
				if (node)
				{
					for (int j = 0; j < format; j++)
						std::cout << '\t';
					std::cout << "\"" << node->key << "\" : ";
					switch (node->type)
					{
					case BOOL:
						std::cout << (node->b_v ? "true" : "false");
						break;
					case INTEGER:
						std::cout << node->i_v;
						break;
					case FLOAT:
						std::cout << node->d_v;
						break;
					case STRING:
						std::cout << "\"" << node->s_v << "\"";
						break;
					case ARRAY:
						std::cout << '[';
						for (st j = 0; j < node->arr_v.size(); j++)
						{
							std::cout << "\"" << node->arr_v[j] << "\"";
							if (j != node->arr_v.size() - 1) std::cout << ", ";
						}
						std::cout << ']';
						break;
					case OBJECT:
						print(r->children[i], format + 1);
						break;
					}
					if (i != root->keys - 1) std::cout << ",";
					std::cout << std::endl;
				}
			}
			for (int j = 0; j < format - 1; j++)
				std::cout << '\t';
			std::cout << '}' << std::endl;
			for (int j = 0; j < format - 1; j++)
				std::cout << '\t';
		}
	}
	BaseNode* findkey(BaseNode*r, const char*s)
	{
		if (r) return r->children[r->table[weight(s)]];
		return nullptr;
	}
	void destory(BaseNode*&r)
	{
		if (r)
		{
			for (st i = 0; i < r->keys; i++)
				destory(r->children[i]);
			delete r;
		}
	}
public:
	JsonTree() { root = nullptr; }
	JsonTree(const char* s)
	{
		root = nullptr;
		parse(s);
	}
	JsonTree(std::string s)
	{
		root = nullptr;
		parse(s.c_str());
	}
	~JsonTree()
	{
		destory(root);
	}
	void parse(std::string s)
	{
		parse(s.c_str());
	}
	void parse(const char *s)
	{
		char str[N]; st i = 0;
		bool arr_sw = false, obj_sw = false, str_sw = false;
		root = new BaseNode();
		BaseNode *nd = root;
		while (*s)
		{
			switch (*s)
			{
			case '{': if (obj_sw) { nd = nd->children[nd->keys - 1]; nd->type = OBJECT; } break;
			case '}': nd = nd->parent; obj_sw = false; break;
			case '"': if (str_sw) str_sw = false; else str_sw = true;
				if (i > 0)
				{
					bool b = false; double dounum = 0;
					ll intnum = 0; st f = 0, pos = 0;
					str[i] = '\0';
					if (!arr_sw)
					{
						if (i < 11)
						{
							if ((str[0] == 't' && str[1] == 'r' && str[2] == 'u' && str[3] == 'e' && str[4] == '\0') ||
								(str[0] == 'f' && str[1] == 'a' && str[2] == 'l' && str[3] == 's' && str[4] == 'e' && str[5] == '\0'))
								f = 3, b = str2bool(str);
							else
							{
								for (st j = 0; j < i; j++)
									if (str[j] == '.' || str[0] == '-' || (str[j] >= '0' && str[j] <= '9')) f = 1;
									else { f = 0; break; }
								if (f)
								{
									for (st j = 0; j < i; j++)
										if (str[j] != '.' || (str[j] >= '0' && str[j] <= '9')) f = 2;
										else { f = 1, pos = j; break; }
									if (f == 2) intnum = str2int(str);
									else dounum = str2float(str, pos, i - 1);
								}
							}
						}
					}
					else f = 4;
					if (nd->isvalue)
					{
						if (nd->children[nd->keys - 1])
						{
							std::string ps(str);
							switch (f)
							{
							case 0:
								nd->children[nd->keys - 1]->type = STRING;
								nd->children[nd->keys - 1]->s_v = ps;
								break;
							case 1:
								nd->children[nd->keys - 1]->type = FLOAT;
								nd->children[nd->keys - 1]->d_v = dounum;
								break;
							case 2:
								nd->children[nd->keys - 1]->type = INTEGER;
								nd->children[nd->keys - 1]->i_v = intnum;
								break;
							case 3:
								nd->children[nd->keys - 1]->type = BOOL;
								nd->children[nd->keys - 1]->b_v = b;
								break;
							case 4:
								nd->children[nd->keys - 1]->type = ARRAY;
								nd->children[nd->keys - 1]->arr_v.push_back(ps);
								break;
							}
						}
					}
					else
					{
						if (nd->table[weight(str)] == -1)
						{
							BaseNode *n = new BaseNode(nd, str);
							nd->children[nd->keys] = n;
						}
					}
					i = 0, obj_sw = true;
				}
				break;
			case ':': if (str_sw) str[i++] = *s; else { if (nd->table[weight(str)] == -1) nd->table[weight(str)] = nd->keys++; nd->isvalue = true; } break;
			case ',': if (str_sw) str[i++] = *s; else { if (!arr_sw) nd->isvalue = false; } break;
			case ' ': if (str_sw) str[i++] = *s; break;
			case '[': arr_sw = true; break;
			case ']': arr_sw = false; break;
			case '/': if (str_sw) str[i++] = *s; break;
			case '\n': break;
			default:  str[i++] = *s; break;
			}
			s++;
		}
	}
	std::string parse2url(const char*s)
	{
		parse(s);
		for (st i = 0; i < root->keys; i++)
		{
			BaseNode*node = root->children[i];
			std::string str = node->key;
			for (st j = 0; j < str.size(); j++)
				pstr += str[j];
			pstr += "=";
			str.clear();
			switch (node->type)
			{
			case BOOL: if (node->b_v) str = "true"; else str = "false";
				break;
			case INTEGER: str = int2str(node->i_v);
				break;
			case FLOAT: str = float2str(node->d_v);
				break;
			case STRING: str = node->s_v;
				break;
			default: std::cerr << "String parse error!" << std::endl;
				return "";
			}
			for (st j = 0; j < str.size(); j++)
				pstr += str[j];
			if (i != root->keys - 1)
				pstr += "&";
		}
		return pstr;
	}
	auto getvalue(const char*key)
	{
		BaseNode*node = findkey(root, key);
		if (node)
	    {
	        /*switch(node->type)
	        {
	        case BOOL: return node->b_v;
	        case INTEGER: return node->i_v;
	        case FLOAT: return node->d_v;
	        case STRING: return node->s_v;
	        case ARRAY: return node->arr_v;
	        default: return node->children[node->table[weight(key)]];
	        }*/
			return node->s_v.c_str();
	    }
		return "";
	}
	auto operator[](const char*key)
	{
		return getvalue(key);
	}
	template<class T> T& operator=(T v)
	{
		return v;
	}
	template<class T> void setvalue(const char*key, T value)
	{
		BaseNode*node = findkey(root, key);
		if (node)
		{
			switch (node->type)
			{
			case BOOL: node->b_v = value; break;
			case INTEGER: node->i_v = value; break;
			case FLOAT: node->d_v = value; break;
			case STRING: node->s_v = value; break;
			case ARRAY: node->arr_v = value; break;
			default:
				/* // 如果是直接在原本的value中进行添加，则不用清除，否则就清除所有子节点
	            for (st i = 0; i < node->keys; i++)
	                destory(node->children[i]);
	            node->keys = 0;*/
				for (st i = 0; i < value.size(); i++)
				{
					node->table[weight(value[i]->key)] = node->keys;
					value[i]->parent = node;
					node->children[node->keys++] = value[i];
				}
				break;
			}
		}
		else
		{
			std::cerr << "Not found the key!" << std::endl;
			return;
		}
	}
	template<class T>void add(const char*key, T value)
	{
		BaseNode*node = findkey(root, key);
		if (!node)
		{
			BaseNode*n = new BaseNode(root, key);
			switch (n->type)
			{
			case BOOL: n->b_v = value; break;
			case INTEGER: n->i_v = value; break;
			case FLOAT: n->d_v = value; break;
			case STRING: n->s_v = value; break;
			case ARRAY: n->arr_v = value; break;
			default:
				n->type = OBJECT;
				for (st i = 0; i < value.size(); i++)
				{
					n->table[weight(value[i]->key)] = n->keys;
					value[i]->parent = n;
					n->children[n->keys++] = value[i];
				}
				break;
			}
			root->table[weight(key)] = root->keys++;
			root->children[root->keys - 1] = n;
		}
		else
		{
			std::cerr << "The key was existed!" << std::endl;
			return;
		}
	}
	void erase(const char*key)
	{
		BaseNode*node = findkey(root, key);
		if (node)
		{
			BaseNode*p = node->parent;
			p->keys--;
			p->table[weight(node->key.c_str())] = -1;
			destory(node);
		}
		else
		{
			std::cerr << "Not found the key!" << std::endl;
			return;
		}
	}
	void visual()
	{
		print(root, 1);
	}
};
int main()
{
	JsonTree r;

	std::fstream context("json.txt", std::fstream::in | std::fstream::out | std::fstream::app);
	std::string line, str;
	while (std::getline(context, line))
		str += line;
	context.close();
	r.parse(str);
	r.visual();
	// std::cout << r.getvalue("time") << std::endl;
	std::cout << r["Verdana"] << std::endl;
	// r.setvalue("time", 154.2);
	std::cout << r.getvalue("Latin_Modern") << std::endl;
	return 0;
}
```

### Update 2018-09-27

##### am 05:03

**此次更新的主要内容**

1. 增加了通过`key`获取/设置对应的`value`的api:`getvalue()`、`setvalue()`；
2. 增加了通过`key`删除`key-value`的api；
3. 增加了函数`weight()`，它的作用是在`json`树内建一个`weighth-table`，使得能够通过传入关键字`key`，在 $ O(n) $（为关键字`key`字符串的长度）时间复杂度定位到`key`。详细`防止映射冲突`算法是通过将字符串每一位字符 `c` 减去 `ascii` 码 `A`的值乘以该字符所处的字符串中的位置，然后将每一位求和，最后再加上字符串长度。设字符串为 $ s $，长度为 $ l $，则用数学公式表示为：
$$
weight(s) = l + \sum_{i = 0}^{l} (s[i] - 'A')*i
$$
4. 重载了`[]`符，使其能够通过字符串`key`来获取或设置`value`值。但并不完善。只能对一种类型的`value`进行操作；
5. 增加了许多内置转换函数：`bits()`、`float2str()`、`float2ch_arr()`、`int2str()`、`con_char2int`；
6. 修复了`str2float()`与`str2int()`为负数时的bug。
7. 完善了之前一些没注意到地方的。

##### 代码

```C++
#include <iostream>
#include <fstream>
#include <vector>
#include <math.h>
#include <string>
#define st size_t
#define ll long long
#define index(c) (c - 'A')
const int N = 1000010;
typedef enum { BOOL, INTEGER, FLOAT, STRING, ARRAY, OBJECT } Type;
st weight(const char*s)
{
    st i = 0, x = 0;
    while (s[i]) x += index(s[i])*i, i += 1;
    return x + i;
}
double str2float(const char *s, st p, st r)
{
	double num = 0, propoint = 0, sign = 1;
	if (*s == '-') s++, sign *= -1;
	while (*s != '.') num = num * 10 + (*s++ - '0');
	s++;
	while (*s) propoint = propoint * 10 + (*s++ - '0');
	num += propoint / std::pow(10, r - p);
	return num*sign;
}
void bits(char s[], st &i, ll x)
{
    if (!x) return ;
    bits(s, i, x/10);
    s[i++] = x%10 + '0';
}
void bits(std::string &s, st &i, ll x)
{
    if (!x) return ;
    bits(s, i, x/10);
    s[i++] = x%10 + '0';
}
std::string float2str(double num)
{
    ll x = (ll)num; st i = 0;
    double y = num - x;
    std::string s;
    bits(s, i, x);
    s[i++] = '.';
    do {
        y *= 10;
        s[i++] = int(y + 0.1)%10 + '0';
    } while (int(ceil(y)) != int(y + 0.1));
    s[i] = 0;
    return s;
}
char* float2ch_arr(double num)
{
    ll x = (ll)num; st i = 0;
    double y = num - x;
    char* s = new char[N >> 1];
    bits(s, i, x);
    s[i++] = '.';
    do {
        y *= 10;
        s[i++] = int(y + 0.1)%10 + '0';
    } while (int(ceil(y)) != int(y + 0.1));
    s[i] = 0;
    return s;
}
ll con_char2int(const char *s)
{
    ll num = 0, sign = 1;
    if (*s == '-') s++, sign *= -1;
	while (*s) num = num * 10 + (*s - '0');
	return num*sign;
}
ll str2int(std::string s)
{
    return con_char2int(s.c_str());
}
std::string int2str(ll num)
{
    std::string s; st i = 0;
    if (num < 0) s += "-", i++;
    bits(s, i, num);
    return s;
}
bool str2bool(const char *s)
{
	bool var;
	if (s[0] == 't' && s[1] == 'r' && s[2] == 'u' && s[3] == 'e' && s[4] == '\0') var = true;
	else var = false;
	return var;
}
class BaseNode {
public:
	bool b_v;
	ll i_v;
	double d_v;
	std::string s_v;
	std::vector<std::string> arr_v;

	std::string key;
	st keys;
	bool isvalue;
	Type type;
	ll table[N];
	BaseNode *children[N], *parent;

	BaseNode()
	{
		keys = 0, isvalue = false, type = OBJECT, parent = nullptr;
		for (int i = 0; i < N; i++) table[i] = -1, children[i] = nullptr;
	}
	BaseNode(BaseNode*p, const char *k)
	{
		std::string sk(k);
		key = sk, keys = 0, isvalue = false, parent = p;
		for (int i = 0; i < N; i++) table[i] = -1, children[i] = nullptr;
	}
	~BaseNode() {}
};
class JsonTree {
private:
	BaseNode *root;
	std::string pstr;
	/**/
	void print(BaseNode*r, int format)
	{
		if (r)
		{
			std::cout << '{' << std::endl;
			for (st i = 0; i < r->keys; i++)
			{
				BaseNode*node = r->children[i];
				if (node)
				{
				    for (int j = 0; j < format; j++)
                        std::cout << '\t';
					std::cout << "\""  << node->key << "\" : ";
					switch (node->type)
					{
					case BOOL:
						std::cout << (node->b_v ? "true" : "false");
						break;
					case INTEGER:
						std::cout << node->i_v;
						break;
					case FLOAT:
						std::cout << node->d_v;
						break;
					case STRING:
						std::cout << "\"" << node->s_v << "\"";
						break;
					case ARRAY:
						std::cout << '[';
						for (st j = 0; j < node->arr_v.size(); j++)
						{
							std::cout << "\"" << node->arr_v[j] << "\"";
							if (j != node->arr_v.size() - 1) std::cout << ", ";
						}
						std::cout << ']';
						break;
					case OBJECT:
						print(r->children[i], format + 1);
						break;
					}
					if (i != root->keys - 1) std::cout << ",";
					std::cout << std::endl;
				}
			}
			for (int j = 0; j < format - 1; j++)
                std::cout << '\t';
			std::cout << '}' << std::endl;
			for (int j = 0; j < format - 1; j++)
                std::cout << '\t';
		}
	}
	BaseNode* findkey(BaseNode*r, const char*s)
	{
	    if (r) return r->children[r->table[weight(s)]];
        return nullptr;
	}
	void destory(BaseNode*&r)
	{
		if (r)
		{
			for (st i = 0; i < r->keys; i++)
				destory(r->children[i]);
			delete r;
		}
	}
public:
	JsonTree() { root = nullptr; }
	JsonTree(const char* s)
	{
		root = nullptr;
		parse(s);
	}
	JsonTree(std::string s)
	{
		root = nullptr;
		parse(s.c_str());
	}
	~JsonTree()
	{
		destory(root);
	}
	void parse(std::string s)
	{
	    parse(s.c_str());
	}
	void parse(const char *s)
	{
		char str[N]; st i = 0;
		bool arr_sw = false, obj_sw = false, str_sw = false;
		root = new BaseNode();
		BaseNode *nd = root;
		while (*s)
		{
			switch (*s)
			{
			case '{': if (obj_sw) { nd = nd->children[nd->keys - 1]; nd->type = OBJECT; } break;
			case '}': nd = nd->parent; obj_sw = false; break;
			case '"': if (str_sw) str_sw = false; else str_sw = true;
			    if (i > 0)
                {
                    bool b = false; double dounum = 0;
                    ll intnum = 0; st f = 0, pos = 0;
                    str[i] = '\0';
                    if (!arr_sw)
                    {
                        if (i < 11)
                        {
                            if ((str[0] == 't' && str[1] == 'r' && str[2] == 'u' && str[3] == 'e' && str[4] == '\0') ||
                                (str[0] == 'f' && str[1] == 'a' && str[2] == 'l' && str[3] == 's' && str[4] == 'e' && str[5] == '\0'))
                                f = 3, b = str2bool(str);
                            else
                            {
                                for (st j = 0; j < i; j++)
                                    if (str[j] == '.' || str[0] == '-' || (str[j] >= '0' && str[j] <= '9')) f = 1;
                                    else { f = 0; break; }
                                if (f)
                                {
                                    for (st j = 0; j < i; j++)
                                        if (str[j] != '.' || (str[j] >= '0' && str[j] <= '9')) f = 2;
                                        else  { f = 1, pos = j; break; }
                                    if (f == 2) intnum = str2int(str);
                                    else dounum = str2float(str, pos, i - 1);
                                }
                            }
                        }
                    }
                    else f = 4;
                    if (nd->isvalue)
                    {
                        if (nd->children[nd->keys - 1])
                        {
                            std::string ps(str);
                            switch (f)
                            {
                            case 0:
                                nd->children[nd->keys - 1]->type = STRING;
                                nd->children[nd->keys - 1]->s_v = ps;
                                break;
                            case 1:
                                nd->children[nd->keys - 1]->type = FLOAT;
                                nd->children[nd->keys - 1]->d_v = dounum;
                                break;
                            case 2:
                                nd->children[nd->keys - 1]->type = INTEGER;
                                nd->children[nd->keys - 1]->i_v = intnum;
                                break;
                            case 3:
                                nd->children[nd->keys - 1]->type = BOOL;
                                nd->children[nd->keys - 1]->b_v = b;
                                break;
                            case 4:
                                nd->children[nd->keys - 1]->type = ARRAY;
                                nd->children[nd->keys - 1]->arr_v.push_back(ps);
                                break;
                            }
                        }
                    }
                    else
                    {
                        if (nd->table[weight(str)] == -1)
                        {
                            BaseNode *n = new BaseNode(nd, str);
                            nd->children[nd->keys] = n;
                        }
                    }
                    i = 0, obj_sw = true;
                }
                break;
			case ':': if (str_sw) str[i++] = *s; else { if (nd->table[weight(str)] == -1) nd->table[weight(str)] = nd->keys++; nd->isvalue = true; } break;
			case ',': if (str_sw) str[i++] = *s; else { if (!arr_sw) nd->isvalue = false; } break;
			case ' ': if (str_sw) str[i++] = *s; break;
			case '[': arr_sw = true; break;
			case ']': arr_sw = false; break;
			case '/': if (str_sw) str[i++] = *s; break;
			case '\n': break;
			default:  str[i++] = *s; break;
			}
			s++;
		}
	}
	std::string parse2url(const char*s)
	{
		parse(s);
		for (st i = 0; i < root->keys; i++)
		{
		    BaseNode*node = root->children[i];
			std::string str = node->key;
			for (st j = 0; j < str.size(); j++)
				pstr += str[j];
			pstr += "=";
			str.clear();
			switch(node->type)
			{
            case BOOL: if (node->b_v) str = "true"; else str = "false";
                break;
            case INTEGER: str = int2str(node->i_v);
                break;
            case FLOAT: str = float2str(node->d_v);
                break;
            case STRING: str = node->s_v;
                break;
            default: std::cerr << "String parse error!" << std::endl;
                return "";
			}
			for (st j = 0; j < str.size(); j++)
				pstr += str[j];
			if (i != root->keys - 1)
				pstr += "&";
		}
		return pstr;
	}
	auto getvalue(const char*key)
	{
	    BaseNode*node = findkey(root, key);
	    if (node)
        {
//            switch(node->type)
//            {
//            case BOOL: return node->b_v;
//            case INTEGER: return node->i_v;
//            case FLOAT: return node->d_v;
//            case STRING: return node->s_v;
//            case ARRAY: return node->arr_v;
//            default: return node->children[node->table[weight(key)]];
//            }
            return node->d_v;
        }
        return 0.0;
	}
	auto operator[](const char*key)
	{
	    BaseNode*node = findkey(root, key);
	    if (node)
        {
//            switch(node->type)
//            {
//            case BOOL: return node->b_v;
//            case INTEGER: return node->i_v;
//            case FLOAT: return node->d_v;
//            case STRING: return node->s_v;
//            case ARRAY: return node->arr_v;
//            default: return node->children[node->table[weight(key)]];
//            }
            return node->s_v.c_str();
        }
        return "";
	}
	template<class T> T& operator=(T v)
	{
		return v;
	}
	template<class T> void setvalue(const char*key, T value)
	{
        BaseNode*node = findkey(root, key);
        if (node)
        {
            switch(node->type)
            {
            case BOOL: node->b_v = value; break;
            case INTEGER: node->i_v = value; break;
            case FLOAT: node->d_v = value; break;
            case STRING: node->s_v = value; break;
            case ARRAY: node->arr_v = value; break;
            default: // 如果是直接在原本的value中进行添加，则不用清除，否则就清除所有子节点
    //            for (st i = 0; i < node->keys; i++)
    //                destory(node->children[i]);
    //            node->keys = 0;
                for (st i = 0; i < value.size(); i++)
                {
                    node->table[weight(value[i]->key)] = node->keys;
                    value[i]->parent = node;
                    node->children[node->keys++] = value[i];
                }
                break;
            }
        }
        else
        {
            std::cerr << "Not found the key!" << std::endl;
            return;
        }
	}
	template<class T> void add(const char*key, T value)
	{
        BaseNode*node = findkey(root, key);
        if (!node)
        {
            BaseNode*n = new BaseNode(root, key);
            switch(n->type)
            {
            case BOOL: n->b_v = value; break;
            case INTEGER: n->i_v = value; break;
            case FLOAT: n->d_v = value; break;
            case STRING: n->s_v = value; break;
            case ARRAY: n->arr_v = value; break;
            default:
                n->type = OBJECT;
                for (st i = 0; i < value.size(); i++)
                {
                    n->table[weight(value[i]->key)] = n->keys;
                    value[i]->parent = n;
                    n->children[n->keys++] = value[i];
                }
                break;
            }
            root->table[weight(key)] = root->keys++;
            root->children[root->keys - 1] = n;
        }
        else
        {
            std::cerr << "The key was existed!" << std::endl;
            return;
        }
	}
	void erase(const char*key)
	{
	    BaseNode*node = findkey(root, key);
        if (node)
        {
            BaseNode*p = node->parent;
            p->keys--;
            p->table[weight(node->key.c_str())] = -1;
            destory(node);
        }
        else
        {
            std::cerr << "Not found the key!" << std::endl;
            return;
        }
	}
	void visual()
	{
		print(root, 1);
	}
};
int main()
{
	JsonTree r;

    std::fstream context("json.txt", std::fstream::in | std::fstream::out | std::fstream::app);
    std::string line, str;
    while (std::getline(context, line))
        str += line;
    context.close();
	r.parse(str);
	r.visual();
	// r.setvalue("time", 156.35);
	std::cout << r.getvalue("time") << std::endl;
    std::cout << r["Verdana"] << std::endl;
	return 0;
}
```

##### example

```json
{
        "Latin_Modern" : "fn_lm",
        "Verdana" : "fn_jvn",
        "Object" : {
                "Color" : ["red", "blue", "green", "white"]
        },
        "time" : "-123.23",
        "flag" : "true"
}
```

output:

```bash
{
        "Latin_Modern" : "fn_lm",
        "Verdana" : "fn_jvn",
        "Object" : {
                "Color" : ["red", "blue", "green", "white"],
        }
        ,
        "time" : -123.23,
        "flag" : true
}
-123.23 // r.getvalue("time")
fn_jvn  // r["Verdana"]
```

### Update 2018-09-26

##### PS pm 15:21

**此次更新所作的主要工作：**

1. 将 `obj_sw` 完全设定为对象开关，如果遇到字符 `{` 便进入到更深的一层；
2. 在数据结构中添加了 `isvalue` ，正如其名，用来判断正在解析的是否为 `value`；
3. 完善了按 `Json` 格式化输出。

##### 代码

```C++
#include <iostream>
#include <vector>
#include <math.h>
#include <string>
#define st size_t
const int N = 1000010;
typedef enum { BOOL, INTEGER, FLOAT, STRING, ARRAY, OBJECT } Type;
double str2float(const char *s, st p, st r)
{
	double num = 0, propoint = 0;
	while (*s != '.') num = num * 10 + (*s++ - '0');
	s++;
	while (*s) propoint = propoint * 10 + (*s++ - '0');
	num += propoint / std::pow(10, r - p);
	return num;
}
st str2int(const char *s)
{
	st num = 0;
	while (*s)
		num = num * 10 + (*s - '0');
	return num;
}
bool str2bool(const char *s)
{
	bool var;
	if (s[0] == 't' && s[1] == 'r' && s[2] == 'u' && s[3] == 'e' && s[4] == '\0') var = true;
	else var = false;
	return var;
}
class BaseNode {
public:
	bool b_v;
	st i_v;
	double d_v;
	std::string s_v;
	std::vector<std::string> arr_v;

	std::string key;
	st keys;
	bool isvalue;
	Type type;
	BaseNode *children[N], *parent;

	BaseNode()
	{
		keys = 0, isvalue = false, type = OBJECT, parent = nullptr;
		for (int i = 0; i < N; i++) children[i] = nullptr;
	}
	BaseNode(BaseNode*p, const char *k)
	{
		std::string sk(k);
		key = sk, keys = 0, isvalue = false, parent = p;
		for (int i = 0; i < N; i++) children[i] = nullptr;
	}
	~BaseNode() {}
};
class JsonTree {
private:
	BaseNode *root;
	std::string pstr;
	/**/
	void print(BaseNode*r, int format)
	{
		if (r)
		{
			std::cout << '{' << std::endl;
			for (st i = 0; i < r->keys; i++)
			{
				BaseNode*node = r->children[i];
				if (node)
				{
				    for (int j = 0; j < format; j++)
                        std::cout << '\t';
					std::cout << "\""  << node->key << "\" : ";
					switch (node->type)
					{
					case BOOL:
						std::cout << (node->b_v ? "true" : "false");
						break;
					case INTEGER:
						std::cout << node->i_v;
						break;
					case FLOAT:
						std::cout << node->d_v;
						break;
					case STRING:
						std::cout << "\"" << node->s_v << "\"";
						break;
					case ARRAY:
						std::cout << '[';
						for (st j = 0; j < node->arr_v.size(); j++)
						{
							std::cout << "\"" << node->arr_v[j] << "\"";
							if (j != node->arr_v.size() - 1) std::cout << ", ";
						}
						std::cout << ']';
						break;
					case OBJECT:
						print(r->children[i], format + 1);
						break;
					}
					if (i != root->keys - 1) std::cout << ",";
					std::cout << std::endl;
				}
			}
			for (int j = 0; j < format - 1; j++)
                std::cout << '\t';
			std::cout << '}' << std::endl;
			for (int j = 0; j < format - 1; j++)
                std::cout << '\t';
		}
	}
	void destory(BaseNode*&r)
	{
		if (r)
		{
			for (st i = 0; i < r->keys; i++)
				destory(r->children[i]);
			delete r;
		}
	}
public:
	JsonTree() { root = nullptr; }
	JsonTree(const char* s)
	{
		root = nullptr;
		parse(s);
	}
	JsonTree(std::string s)
	{
		root = nullptr;
		parse(s.c_str());
	}
	~JsonTree()
	{
		destory(root);
	}
	void parse(const char *s)
	{
		char str[N]; int i = 0;
		bool arr_sw = false, obj_sw = false;
		root = new BaseNode();
		BaseNode *nd = root;
		while (*s)
		{
			switch (*s)
			{
			case '{': if (obj_sw) { nd = nd->children[nd->keys - 1]; nd->type = OBJECT; } break;
			case '}': nd = nd->parent; obj_sw = false; break;
			case '"':   if (i > 0)
			{
				bool b = false; double dounum = 0;
				st intnum = 0, f = 0, pos = 0;
				str[i] = '\0';
				if (!arr_sw)
				{
					if (i < 11)
					{
					    if ((str[0] == 't' && str[1] == 'r' && str[2] == 'u' && str[3] == 'e' && str[4] == '\0') ||
							(str[0] == 'f' && str[1] == 'a' && str[2] == 'l' && str[3] == 's' && str[4] == 'e' && str[5] == '\0'))
							f = 3, b = str2bool(str);
                        else
                        {
                            for (int j = 0; j < i; j++)
                                if (str[j] == '.' || (str[j] >= '0' && str[j] <= '9')) f = 1;
                                else { f = 0; break; }
                            if (f)
                            {
                                for (int j = 0; j < i; j++)
                                    if (str[j] >= '0' && str[j] <= '9') f = 2;
                                    else { f = 1, pos = j; break; }
                                if (f == 2) intnum = str2int(str);
                                else dounum = str2float(str, pos, i - 1);
                            }
                        }
					}
				}
				else f = 4;
				if (nd->isvalue)
				{
					std::string ps(str);
					switch (f)
					{
					case 0:
					    nd->children[nd->keys - 1]->type = STRING;
						nd->children[nd->keys - 1]->s_v = ps;
						break;
					case 1:
						nd->children[nd->keys - 1]->type = FLOAT;
						nd->children[nd->keys - 1]->d_v = dounum;
						break;
					case 2:
						nd->children[nd->keys - 1]->type = INTEGER;
						nd->children[nd->keys - 1]->i_v = intnum;
						break;
					case 3:
						nd->children[nd->keys - 1]->type = BOOL;
						nd->children[nd->keys - 1]->b_v = b;
						break;
					case 4:
						nd->children[nd->keys - 1]->type = ARRAY;
						nd->children[nd->keys - 1]->arr_v.push_back(ps);
						break;
					}
				}
				else
				{
					BaseNode *n = new BaseNode(nd, str);
					nd->children[nd->keys] = n;
				}
				i = 0, obj_sw = true;
			}
						break;
			case ':': nd->keys += 1; nd->isvalue = true; break;
			case ',': if (!arr_sw) nd->isvalue = false; break;
			case ' ': break;
			case '[': arr_sw = true; break;
			case ']': arr_sw = false; break;
			case '\n': break;
			default:  str[i++] = *s; break;
			}
			s++;
		}
	}
	std::string parse2url(const char*s)
	{
		parse(s);
		for (st i = 0; i < root->keys; i++)
		{
			std::string str = root->children[i]->key;
			for (auto p : str)
				pstr += p;
			pstr += "=";
			str = root->children[i]->s_v;
			for (auto p : str)
				pstr += p;
			if (i != root->keys - 1)
				pstr += "&";
		}
		return pstr;
	}
	void visual()
	{
		print(root, 1);
	}
};
int main()
{
	char str[] = "{\"Latin_Modern\":\"fn_lm\", \"Verdana\":\"fn_jvn\", \"Object\": {\"Color\": [\"red\", \"blue\", \"green\", \"white\"]}, \"time\" : \"123.23\", \"flag\": \"true\"}";

	JsonTree r;

	r.parse(str);
	r.visual();
	return 0;
}
```

output:

```bash
{
        "Latin_Modern" : "fn_lm",
        "Verdana" : "fn_jvn",
        "Object" : {
                "Color" : ["red", "blue", "green", "white"],
        }
        ,
        "time" : 123.23,
        "flag" : true
}
```

##### PS am 3点左右

**此次更新所作的主要工作：**

1. 对数组类型做了解析判断；
2. 重命名了变量 `k` 为 `obj_sw`，它的主要作用是判断当前数据 ~~是 `key` 还是 `value`或者是~~ 新的对象的入口。
3. 添加了按 Json 格式输出的函数；
4. 完善了一些之前的问题。

##### 代码

```C++
#include <iostream>
#include <vector>
#include <math.h>
#include <string>
#define st size_t
const int N = 1000010;
typedef enum { BOOL, INTEGER, FLOAT, STRING, ARRAY, OBJECT } Type;
double str2float(const char *s, st p, st r)
{
	double num = 0, propoint = 0;
	while (*s != '.') num = num * 10 + (*s++ - '0');
	s++;
	while (*s) propoint = propoint * 10 + (*s++ - '0');
	num += propoint / std::pow(10, r - p);
	return num;
}
st str2int(const char *s)
{
	st num = 0;
	while (*s)
		num = num * 10 + (*s - '0');
	return num;
}
bool str2bool(const char *s)
{
	bool var;
	if (s[0] == 't' && s[1] == 'r' && s[2] == 'u' && s[3] == 'e' && s[4] == '\0') var = true;
	else var = false;
	return var;
}
class BaseNode {
public:
	bool b_v;
	st i_v;
	double d_v;
	std::string s_v;
	std::vector<std::string> arr_v;
	BaseNode *children[N];
	std::string key;
	st keys;
	Type type;

	BaseNode()
	{
		keys = 0, type = OBJECT;
	}
	BaseNode(const char *k, st n)
	{
		std::string sk(k);
		key = sk, keys = n;
	}
	~BaseNode() {}
};
class JsonTree {
private:
	BaseNode *root;
	std::string pstr;
	/**/
	void print(BaseNode*r)
	{
		if (r)
		{
			std::cout << '{' << std::endl;
			for (st i = 0; i < r->keys; i++)
			{
				BaseNode*node = r->children[i];
				if (node)
				{
					std::cout << '\t' << "\""  << node->key << "\" : ";
					switch (node->type)
					{
					case BOOL:
						std::cout << (node->b_v ? "true" : "false");
						break;
					case INTEGER:
						std::cout << node->i_v;
						break;
					case FLOAT:
						std::cout << node->d_v;
						break;
					case STRING:
						std::cout << "\"" << node->s_v << "\"";
						break;
					case ARRAY:
						std::cout << '[';
						for (st j = 0; j < node->arr_v.size(); j++)
						{
							std::cout << "\"" << node->arr_v[j] << "\"";
							if (j != node->arr_v.size() - 1) std::cout << ", ";
						}
						std::cout << ']';
						break;
					case OBJECT:
						print(r->children[i]);
						break;
					}
					if (i != root->keys - 1) std::cout << ",";
					std::cout << std::endl;
				}
			}
			std::cout << '}' << std::endl;
		}
	}
	void destory(BaseNode*&r)
	{
		if (r)
		{
			for (st i = 0; i < r->keys; i++)
				destory(r->children[i]);
			delete r;
		}
	}
public:
	JsonTree() { root = nullptr; }
	JsonTree(const char* s)
	{
		root = nullptr;
		parse(s);
	}
	JsonTree(std::string s)
	{
		root = nullptr;
		parse(s.c_str());
	}
	~JsonTree()
	{
		destory(root);
	}
	void parse(const char *s)
	{
		char str[N]; bool arr_sw = false;
		int i = 0, obj_sw = 0;
		root = new BaseNode();
		BaseNode *nd = root;
		while (*s)
		{
			switch (*s)
			{
			case '{': if (obj_sw > 0) { nd = nd->children[nd->keys - 1]; nd->type = OBJECT; obj_sw = 0; } break;
			case '}': break;
			case '"':   if (i > 0)
			{
				bool b; double dounum = 0; 
				st intnum = 0, f = 0, pos = 0;
				str[i] = '\0';
				if (!arr_sw)
				{
					if (i < 11)
					{
						for (int j = 0; j < i; j++)
							if (str[j] == '.' || (str[j] >= '0' && str[j] <= '9')) f = 1;
							else { f = 0; break; }
						if (f)
						{
							for (int j = 0; j < i; j++)
								if (str[j] >= '0' && str[j] <= '9') f = 2;
								else { f = 1, pos = j; break; }
							if (f == 2) intnum = str2int(str);
							else dounum = str2float(str, pos, i - 1);
						}
						else if ((str[0] == 't' && str[1] == 'r' && str[2] == 'u' && str[3] == 'e' && str[4] == '\0') ||
							(str[0] == 'f' && str[1] == 'a' && str[2] == 'l' && str[3] == 's' && str[4] == 'e' && str[5] == '\0'))
							f = 3, b = str2bool(str);
					}
				}
				else f = 4;
				if (obj_sw & 1)
				{
					std::string ps(str);
					switch (f)
					{
					case 0:
						nd->children[nd->keys - 1]->s_v = ps;
						nd->children[nd->keys - 1]->type = STRING;
						break;
					case 1:
						nd->children[nd->keys - 1]->type = FLOAT;
						nd->children[nd->keys - 1]->d_v = dounum;
						break;
					case 2:
						nd->children[nd->keys - 1]->type = INTEGER;
						nd->children[nd->keys - 1]->i_v = intnum;
						break;
					case 3:
						nd->children[nd->keys - 1]->type = BOOL;
						nd->children[nd->keys - 1]->b_v = b;
						break;
					case 4:
						nd->children[nd->keys - 1]->type = ARRAY;
						nd->children[nd->keys - 1]->arr_v.push_back(ps);
						obj_sw -= 1;
						break;
					}
				}
				else
				{
					BaseNode *n = new BaseNode(str, 0);
					nd->children[nd->keys] = n;
				}
				i = 0, obj_sw += 1;
			}
						break;
			case ':': nd->keys += 1; break;
			case ',': break;
			case ' ': break;
			case '[': arr_sw = true; break;
			case ']': arr_sw = false; break;
			case '\n': break;
			default:  str[i++] = *s; break;
			}
			s++;
		}
	}
	std::string parse2url(const char*s)
	{
		parse(s);
		for (st i = 0; i < root->keys; i++)
		{
			std::string str = root->children[i]->key;
			for (auto p : str)
				pstr += p;
			pstr += "=";
			str = root->children[i]->s_v;
			for (auto p : str)
				pstr += p;
			if (i != root->keys - 1)
				pstr += "&";
		}
		return pstr;
	}
	void visual()
	{
		print(root);
	}
};
int main()
{
	char str[] = "{\"Latin_Modern\":\"fn_lm\", \"Verdana\":\"fn_jvn\", \"Color\":[\"red\",\"blue\",\"green\",\"white\"]}";
	JsonTree r;

	r.parse(str);
	r.visual();
	return 0;
}
```

output:

```bash
{
        "Latin_Modern" : "fn_lm",
        "Verdana" : "fn_jvn",
        "Color" : ["red", "blue", "green", "white"]
}
```

### Update 2018-9-25

改用了部分`C++`的特点来编写该`Json 解析器`，目前只是一个十分简易的解析器，还有许多需要完善的地方。

```C++
#include <iostream>
#include <math.h>
#include <string>
#define st size_t
const int N = 1000010;
typedef enum { BOOL, INTEGER, FLOAT, STRING, ARRAY, OBJECT } Type;
double str2float(const char *s, st p, st r)
{
	double num = 0, propoint = 0;
	while (*s != '.') num = num * 10 + (*s++ - '0');
	s++;
	while (*s) propoint = propoint * 10 + (*s++ - '0');
	num += propoint / std::pow(10, r - p);
	return num;
}
int str2int(const char *s)
{
	int num = 0;
	while (*s) num = num * 10 + (*s - '0');
	return num;
}
bool str2bool(const char *s)
{
	bool var;
	if (s[0] == 't' && s[1] == 'r' && s[2] == 'u' && s[3] == 'e' && s[4] == '\0') var = true;
	else var = false;
	return var;
}
class BaseNode {
public:
	bool b_v;
	int i_v;
	double d_v;
	std::string s_v;
	BaseNode *children[N];
	std::string key;
	st keys;
	Type type;

	BaseNode()
	{
		keys = 0, type = OBJECT;
	}
	BaseNode(const char *k, st n)
	{
		std::string sk(k);
		key = sk, keys = n;
	}
	~BaseNode() {}
};
class JsonTree {
private:
	BaseNode *root;
	std::string pstr;
	/**/
	void destory(BaseNode*&r)
	{
		if (r)
		{
			for (st i = 0; i < r->keys; i++)
				destory(r->children[i]);
			delete r;
		}
	}
public:
	JsonTree() { root = nullptr; }
	JsonTree(const char* s)
	{
		root = nullptr;
		parse(s);
	}
	JsonTree(std::string s)
	{
		root = nullptr;
		parse(s.c_str());
	}
	~JsonTree()
	{
		destory(root);
	}
	void parse(const char *s)
	{
		char str[N]; int i = 0, k = 0;
		root = new BaseNode();
		BaseNode *nd = root;
		while (*s)
		{
			switch (*s)
			{
			case '{': if (k > 0) { nd = nd->children[nd->keys - 1]; nd->type = OBJECT; k = 0; } break;
			case '}': break;
			case '"':   if (i > 0)
			{
				bool b; double dounum = 0; 
				int intnum = 0, f = 0, pos = 0;
				str[i] = '\0';
				for (int j = 0; j < i; j++)
					if (str[j] == '.' || (str[j] >= '0' && str[j] <= '9')) f = 1;
					else { f = 0; break; }
				if (f)
				{
					for (int j = 0; j < i; j++)
						if (str[j] >= '0' && str[j] <= '9') f = 2;
						else { f = 1, pos = j; break; }
					if (f == 2) intnum = str2int(str);
					else dounum = str2float(str, pos, i - 1);
				}
				else if ((str[0] == 't' && str[1] == 'r' && str[2] == 'u' && str[3] == 'e' && str[4] == '\0') ||
					(str[0] == 'f' && str[1] == 'a' && str[2] == 'l' && str[3] == 's' && str[4] == 'e' && str[5] == '\0'))
					f = 3, b = str2bool(str);
				if (k & 1)
				{
					std::string ps(str);
					nd->children[nd->keys - 1]->s_v = ps;
					switch (f)
					{
					case 0:
						nd->children[nd->keys - 1]->type = STRING;
						break;
					case 1:
						nd->children[nd->keys - 1]->type = FLOAT;
						nd->children[nd->keys - 1]->d_v = dounum;
						break;
					case 2:
						nd->children[nd->keys - 1]->type = INTEGER;
						nd->children[nd->keys - 1]->i_v = intnum;
						break;
					case 3:
						nd->children[nd->keys - 1]->type = BOOL;
						nd->children[nd->keys - 1]->b_v = b;
						break;
					}
				}
				else
				{
					BaseNode *n = new BaseNode(str, 0);
					nd->children[nd->keys] = n;
				}
				i = 0, k += 1;
			}
						break;
			case ':': nd->keys += 1; break;
			case ',': break;
			case ' ': break;
			case '\n': break;
			default:  str[i++] = *s; break;
			}
			s++;
		}
	}
	std::string parse2url(const char*s)
	{
		parse(s);
		for (st i = 0; i < root->keys; i++)
		{
			std::string str = root->children[i]->key;
			for (auto p : str)
				pstr += p;
			pstr += "=";
			str = root->children[i]->s_v;
			for (auto p : str)
				pstr += p;
			if (i != root->keys - 1)
				pstr += "&";
		}
		return pstr;
	}
};
int main()
{
	char str[] = "{\"Latin_Modern\":\"fn_lm\", \"Verdana\":\"fn_jvn\", \"time\":\"723.34\"}"; // "{\"Latin_Modern\":\"fn_lm\",\"Verdana\":\"fn_jvn\", \"Color\":[\"red\",\"blue\",\"green\"]}"
	JsonTree r;

	std::string s = r.parse2url(str);
	std::cout << "https://darkchii.cn/s/" << s << std::endl;
	return 0;
}
```

output-vs2017 最新版本

```bash
https://darkchii.cn/s/Latin_Modern=fn_lm&Verdana=fn_jvn&time=723.34

C:\Users\adimin\source\repos\JsonParseMachine\Debug\JsonParseMachine.exe (process 18988) exited with code 0.
To automatically close the console when debugging stops, enable Tools->Options->Debugging->Automatically close the conso
le when debugging stops.
Press any key to close this window . . .
```

output-g++ 6.3.0

```bash
https://darkchii.cn/s/Latin_Modern=fn_lm&Verdana=fn_jvn&time=723.34

Process returned 0 (0x0)   execution time : 0.243 s
Press any key to continue.
```

### 未完成版

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define SIZE 10010
#define ll size_t
typedef enum { BOOLEAN, INTEGER, FLOAT, STRING, ARRAY, OBJECT } Type;
typedef struct obj {
	union {
		bool b_v;
		int i_v;
		double d_v;
		const char *s_v;
		struct obj *children[SIZE];
	} value;

	const char *key;
	ll keys;
	Type type;
} JsonTrees;
struct JsonObj {
	JsonTrees *root;
};
void initialized(struct JsonObj **prt)
{
	*prt = (struct JsonObj *)malloc(sizeof(struct JsonObj));
	(*prt)->root = (JsonTrees *)malloc(sizeof(JsonTrees));
	memset((*prt)->root->value.children, 0, sizeof((*prt)->root->value.children));
	(*prt)->root->type = OBJECT;
	(*prt)->root->key = NULL;
	(*prt)->root->keys = 0;
}
JsonTrees *parse(const char *s)
{
	struct JsonObj *prt;
	JsonTrees *r, *nd;
	char str[SIZE];
	int i = 0, k = 0;
	memset(str, 0, sizeof(str));
	initialized(&prt);
	nd = r = prt->root;
	while (*s)
	{
		switch (*s)
		{
		case '{': if (k > 0) { nd = nd->value.children[nd->keys - 1]; nd->type = OBJECT; k = 0; } break;
		case '}': break;
		case '"': if (i > 0) {
					str[i] = '\0'; i = 0;
					if (k & 1)
					{
						nd->value.children[nd->keys - 1]->type = STRING;
						nd->value.children[nd->keys - 1]->value.s_v = str;
					}
					else
					{
						JsonTrees *n = (JsonTrees *)malloc(sizeof(JsonTrees));
						memset(n->value.children, 0, sizeof(n->value.children));
						n->keys = 0;
						n->key = str;
						nd->value.children[nd->keys] = n;
					}
					k++;
				}
				  break;
		case ':': nd->keys++; break;
		case ',': break;
		case ' ': break;
		case '\n': break;
		default:  str[i++] = *s; break;
		}
		s++;
	}
	return r;
}
void parse2url(JsonTrees *pt, char *url_endings)
{
	int j = 0;
	for (int i = 0; i < pt->keys; i++)
	{
		const char *ps = pt->value.children[i]->key;
		while (*ps)
			url_endings[j++] = *ps++;
		url_endings[j++] = '=';
		ps = pt->value.children[i]->value.s_v;
		while (*ps)
			url_endings[j++] = *ps++;
		url_endings[j++] = '&';
	}
	url_endings[j - 1] = '\0';
}
void destory(struct JsonObj **prt, JsonTrees **pt)
{
	if (*pt)
	{
		for (int i = 0; i < (*pt)->keys; i++)
			destory(prt, &(*pt)->value.children[i]);
		free(*pt);
		if (*prt)
			free(*prt);
	}
}
int main()
{
	char str[] = "{\"Latin_Modern\":\"fn_lm\",\"Verdana\":\"fn_jvn\"}"; // "{\"Latin_Modern\":\"fn_lm\",\"Verdana\":\"fn_jvn\", \"Color\":[red,blue,green]}"
	char url_eds[SIZE];

	memset(url_eds, 0, sizeof(url_eds));
	JsonTrees *r = parse(str);
	parse2url(r, url_eds);
	printf("https://darkchii.cn/s/");
	puts(url_eds);
	getchar();
	return 0;
}
```

代码有个迷のbug，貌似每一个`children`的地址都被最后一个节点覆盖了，所以不管`key`or`value`都是最后一个字符串`fn_jvn`，但我没找到原因。

这颗树类似这样的建立方式：

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
struct T {
    T *next[3];
    int v;
};
struct P {
    T* root;
};
T*init()
{
    P *pt = (P*)malloc(sizeof(P));
    pt->root = (T*)malloc(sizeof(T));
    T *next_n;
    next_n = pt->root;
    for (int i = 0; i < 3; i++)
    {
        T *n = (T*)malloc(sizeof(T));
        memset(n->next, 0, sizeof(n->next));
        n->v = i;
        next_n->next[i] = n;
    }
    return pt->root;
}
int main()
{
    T*head = init();
    for (int i = 0; i < 3; i++)
    {
        printf("%d\n", head->next[i]->v);
    }
    return 0;
}
```

这样写没有任何问题。
