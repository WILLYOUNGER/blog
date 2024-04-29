---
title: 王新晓的代码规范（C++）
description: 记录我的代码规范
slug: rulecpp
date: 2024-04-29 00:00:00+0000
categories:
    - 代码规范
    - C++
tags:
    - 文档
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
params:
    author: 王新晓
---


# C/C++编码规范
以下编码规则仅适用于王新晓的 C/C++代码

## 一、文件结构

### 头文件规范

1. 一个头文件需要 #define保护，命名格式为：`_<PROJECT>_<PATH>_<FILE>_H_`
如：项目xzpt中，头文件xzpt\Classes\xzpt\xzptGameSence.h应定义如下：（需要写全路径）（宏定义 #endif 后须加注释）
```
#ifndef _XZPT_XZPT_XZPTGAMESENCE_H_
#define _XZPT_XZPT_XZPTGAMESENCE_H_
#endif // _XZPT_XZPT_XZPTGAMESENCE_H_
```

2. `#include`的顺序（每种头文件间应空行）
    1. 当前文件的对应头文件
    2. C系统文件
    3. C++系统文件
    4. 其他库的头文件
    5. 本项目的头文件
## 二、命名规范
### 文件命名
使用小驼峰法命名如：
`xzptGameSence.cpp`
`xzptGameSence.h`

### 类型命名
class 开头为 C， struct开头为S， interface开头为I
大驼峰法
例如：
`CXzptGameSence`
`SXzptUserInfo`
`IXzptLogin`

### 变量命名
指针需在结尾添加_ptr，若为常量指针（我这里的意思是指针不能改变指向的东西，但是可以改变这个指向的东西的内容）需在结尾添加_cPtr
#### 局部变量
以下划线开头，第一个单词代表变量的类型(STL中的情况使用小驼峰法命名)，第二个单词为项目名称，第三个单词为变量意义（使用小驼峰法），每个单词之间加下滑线，例如：
`Sprite* _sprite_xzpt_up_leaves_ptr;`
`int _int_xzpt_Level;`
`std::vector<Sprite> _vectorSprite_xzpt_listMoveShape;`
`std::list<xzptSprite*> _listXzptSpritePtr_xzpt_listShape;`

#### 全局变量

以g_开头，第一个单词代表变量的类型(STL中的情况使用小驼峰法命名)，第二个单词为项目名称，第三个单词为变量意义（使用小驼峰法），每个单词间添加下滑线，例如：
`char* g_char_xzpt_userInfoData_ptr;`
`int g_int_xzpt_version;`

#### 成员变量

以m_开头，第一个单词代表变量的类型(STL中的情况使用小驼峰法命名)，第二个单词为项目名称，第三个单词为变量意义（使用小驼峰法），每个单词间添加下滑线，例如：
`Sprite* m_sprite_xzpt_hand_ptr;`
`int m_int_xzpt_actionNum;`

#### 函数中的参数

1. 直接写变量含义，例如：
`calMinDistanceLineWithShape(Vec2 pointX, Vec2 pointY, Vec2 pointM);`
2. 函数中的参数如果是传出数据使用，再开头添加out，使用小驼峰法，例如：
`calMaxTwoNumber(int num0, int num1, int& outResultNumber)`

#### 其他补充规则
1. for循环中括号内的局部循环标记量可直接定义为字母，使用字母顺序如下 `i` `j` `m` `n` `x` `y` `z` ，之后的不做顺序规定
2. 其中基础类型名称在对象名中体现可以只写首字母，例如：`int  _i_xzpt_level;`
3. 单例对象用大写A开头，整体采用大驼峰法


### 常量命名


以c_开头，第一个单词代表变量的类型(STL中的情况使用小驼峰法命名)，第二个单词为项目名称，第三个单词为变量意义（使用小驼峰法），每个单词间添加下滑线，例如：
`const char* c_char_xzpt_direction_ptr;`

**注意：常量与变量相同拥有全局常量，局部常量等情况，故规定如下**
    1. **局部常量为 _c_开头**
    2. **其他情况可直接与开头字母合并，如下(若为全局变量)**
    `const char* const cg_char_xzpt_hurdleInfoData_cPtr = "123.txt";` 
    或
    `const char* const gc_char_xzpt_hurdleInfoData_cPtr = "123.txt";`
    **此处gc与cg皆可**
### 静态量命名
以s_开头，第一个单词代表变量的类型(STL中的情况使用小驼峰法命名)，第二个单词为项目名称，第三个单词为变量意义（使用小驼峰法），每个单词间添加下滑线，例如：
`static bool s_bool_xzpt_firstTimeGetDesign;`
**此处注意与常量注意相同，若遇到常量，静态，全局等因素组合，除局部因素将_s加在开头外，其他因素皆可直接组合例如：**
`static cont bool scg_bool_xzpt_firstTimeGetDesign = true;`
**scg可任意排列组合**

### 函数命名
1. 函数名必须直观，并且能正确表达其内在功能
2. 对于函数参数中的引用和指针类型视情况以const修饰
3. 对于不修改数据成员的类成员函数，以const修饰
4. 函数名以小写开头，使用小驼峰法
5. 返回值为布尔型的一些检测函数，正确的要返回true，错误的返回false，不要调转含义

### 宏命名规范

1. 尽量不使用宏定义
2. 如果使用宏需全部大写，每个单词以下滑线分割

### typedef类型命名规范

`typedef std::vector<int> SeqInt;` //vector的全部都在前面加Seq。
`typedef ::std::map<int, int> DictIntInt;`//map在cdl中加前缀Dict，在代码中用Map
1. 必须以大写开头。
2. 命名翻译的是内部的组成，例如SeqInt，一看就是vector<int>。不能改为PlayerId这种具有确切意义的命名，缺乏通用性。具体含义应该在变量名上反映出来。


## 代码风格

### 空格放置
1. 循环后加空格再加括号，例：`for (int i = 0; i < 10; ++i)`
2. 函数名后不需要加空格，紧跟`(` `)`
3. `(` 后紧跟与 `)`后紧跟不加空格
4. 嵌套`(` `)` 与`(` `)` 之间加空格如：`if ( (x == 1) && (x == 2) )`
5.  `;`不是结尾加空格
6. 除`[ ]`、`.`、`->`、`::`外，双目操作符， 如`=`、`+=`、`>=`、`<= `、`+`、`* `、`% `、`&& `、`||`、`<<`、`^`等两侧各留一个空格。
7. 单目操作符如`!`、`~`、`++`、`--`、`&`（地址运算符）与操作数之间不留空格。
8. 当一个函数的返回值是指针变量或引用变量时，类型与操作符（`*`或`&`）之间不留空格，操作符之后留一个空格；
9. 定义指针时，`*`放在对象前，与对象中间无空格，当作为函数入参时，`*`放在类后，与类中间无空格
10. `{` `}` 用于初始化时不换行，在中间加空格

### 布局规范
1. 每个函数，成员函数声明之间保留一个空格
2. 同一个函数体内，有语义转换时，保留一个空行
3. `{` `}`会换行
4. `++` `--`前后可选择时，放在前面
5. `{` `}` 用于初始化时不换行，在中间加空格

### 注释规范
1. 块注释使用`/* */`， 行注释使用`//`
2. 函数声明前需注释函数含义（除非特别简单能从函数命名推测出来）（若函数功能简单，无需介绍返回值与参数也可以使用 `//`直接写注释）具体格式如下：
```
/** 
*  @brief 这里写函数功能
* @param i 参数1
* @return 返回说明
*    -<em>false</em> fail
*    -<em>true</em> succeed
*/
bool funciotn(int i)
```
3. 代码的注释放在该行代码上
4. 函数实现的开头需要写函数实现的大致步骤（特别简单的可不写）
5. 每个文件最前面需写注释，格式如下(可适当删除不需要的项）)：
```
/**
*  @file     Example.h
*  @brief    对文件的简述 
*  Details. 
* 
*  @author   wangxinxiao
*  @email    wxx1035@163.com
*  @version  1.0.0.1(版本号) 
*  @date     2022/5/11
*  @license  GNU General Public License (GPL) 
* 
*  Remark         : Description
*
*  Change History : 
*  <Date>     | <Version> | <Author>       | <Description> 
*  2014/01/24 | 1.0.0.1   | wangxinxiao      | Create file 
*
*/
```

### 其他规范
1. 符合表达式需要使用小括号，不要使用默认优先级
2. 判断指针是否为空，如：`if (ptr == nullptr)`
3. 判断是否为某个值时，值放在前面，如：`if (0 == i)`
4. 判断`bool`时，如：`if (flag)` 与 `if (!flag)`