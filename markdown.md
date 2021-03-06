---
authou:    nyfblack
day:    2019.01.16
title:    markdown 
---
<!-- MarkdownTOP -->

<!-- /MarkdownTOP -->
# 1.分割线
```
最常使用就是三个或以上的 *  - 和 _
***
---
_____ 
```
***
---
_____ 

***
# 2.标题
***
```
一级标题
=
二级标题
-
# 第一级标题 `<h1>` 
## 第二级标题 `<h2>` 
### 第三级标题 `<h3>` 
#### 第二四级标题 `<h4>` 
##### 第五级标题 `<h5>` 
###### 第六级标题 `<h6>` 
```
一级标题
=
二级标题
-
# 第一级标题 `<h1>` 
## 第二级标题 `<h2>` 
### 第三级标题 `<h3>` 
#### 第二四级标题 `<h4>` 
##### 第五级标题 `<h5>` 
###### 第六级标题 `<h6>` 
***
# 3.段落
前后要有空行，所谓的空行是指没有文字内容。若想在段内强制换行的方式是使用两个以上空格加上回车（引用中换行省略回车）
***
# 4.区块引用
***
```
> 区块引用
> > 嵌套引用
> > >三嵌套引用
> > > > 四嵌套引用
```
> 区块引用
> > 嵌套引用
> > >三嵌套引用
> > > > 四嵌套引用
***
# 5.代码块
***
代码块:在每行加上4个空格或者一个制表符（如同写代码一样）。   
       代码之间分别用三个反引号包起来，且两边的反引号单独占一行;  
注意⚠️：需要和普通段落之间存在空行  
单行代码：代码之间分别用一个反引号包起来；

`create database hero;`  
   
    public 

***
# 6.强调
***
```
**这是加粗的文字**  
__这是加粗的文字__  
*这是倾斜的文字*  
_这是倾斜的文字_  
***这是斜体加粗的文字***  
~~这是加删除线的文字~~  
```
**这是加粗的文字**  
__这是加粗的文字__  
*这是倾斜的文字*  
_这是倾斜的文字_  
***这是斜体加粗的文字***  
~~这是加删除线的文字~~  
***
# 7.列表 （有序，无序）
***
## 7.1无序列表用 
```
'- + *'都可以
注意：标记后面最少有一个_空格_或_制表符_。若不在引用区块中，必须和前方段落之间存在空行。
- 列表1
+ 列表2
* 列表3
```
- 列表1
+ 列表2
* 列表3
***
## 7.2有序列表
```
数字加点
注意：序号跟内容之间要有空格
1. 列表1  
2. 列表2  
3. 列表3 
```
1. 列表1  
2. 列表2  
3. 列表3  
***
## 7.3嵌套列表
```
上一级和下一级之间敲三个空格即可
- 1
   - 1.2
      - 1.2.1
```
- 1
   - 1.2
      - 1.2.1
***
# 8.链接
***
链接可以由两种形式生成，行内式 和 参考式。  
```
[GitHub](http://github.com)  
自动生成连接  <http://www.github.com/>
```
[GitHub](http://github.com)  
自动生成连接  <http://www.github.com/>
***
# 9.图片
***   
```
添加图片形式和链接相似，只需要在链接的基础上前方加一个 ！号。
格式：![图片alt](图片地址 'title')  
    title是鼠标移动到图片上，显示的文字
![GitHub set up](http://zh.mweb.im/asset/img/set-up-git.gif 'title')
![](http://zh.mweb.im/asset/img/set-up-git.gif "title")
```
![](http://zh.mweb.im/asset/img/set-up-git.gif "title")
***
# 10.特殊符号的使用
***
`不转义特殊符号（原样输出）`    
'Ctrl+A'
***
# 11.表格
***
```
第一格表头 | 第二格表头
---------| -------------
内容单元格 第一列第一格 | 内容单元格第二列第一格
内容单元格 第一列第二格 多加文字 | 内容单元格第二列第二格
内容单元格 第一列第三格 多加文字 | 内容单元格第二列第三格
内容单元格 第一列第四格 多加文字 | 内容单元格第二列第四格
```
第一格表头 | 第二格表头
---------| -------------
内容单元格 第一列第一格 | 内容单元格第二列第一格
内容单元格 第一列第二格 多加文字 | 内容单元格第二列第二格
内容单元格 第一列第三格 多加文字 | 内容单元格第二列第三格
内容单元格 第一列第四格 多加文字 | 内容单元格第二列第四格
***
```

第二行分割表头和内容。
- 有一个就行，为了对齐，多加了几个
文字默认居左
-两边加：表示文字居中
-右边加：表示文字居右
注：原生的语法两边都要用 | 包起来。此处省略

姓名|技能|排行
--|:--:|--:
刘备|哭|大哥
关羽|打|二哥
张飞|骂|三弟
```
姓名|技能|排行
--|:--:|--:
刘备|哭|大哥
关羽|打|二哥
张飞|骂|三弟
***
# 12.流程图
***
flow
st=>start: 开始
op=>operation: My Operation
cond=>condition: Yes or No?
e=>end
st->op->cond
cond(yes)->e
cond(no)->op
&
***





