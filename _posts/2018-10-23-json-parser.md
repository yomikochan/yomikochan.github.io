---
title: 轻量级Json解析器
layout: page
date: '2018-07-21 13:38:00 +0800'
tags:
- C/C++
- Json
categories:
- 词法分析树
cover_index: "/assets/1632277878.jpg"
comments: true
---

以前写的垃圾代码，有很多很糟糕的地方，并且由于数据结构设计不正确，不能解析数组中存放对象的情况：

```cpp
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

```
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
