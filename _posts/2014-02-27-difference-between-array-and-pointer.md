---
layout: post
title:  数组和指针何时等同，何时不同？
date:   2014-02-27 21:34
category: "C/C++"
---

<p><span style="font-size: 16px;">每个人都知道在C语言中，数组和指针非常相似。我甚至经常听到一种说法，"数组就是指针，指针就是数组"，但<span style="color: #ff0000;">这是一种完全错误的说法</span>。</span></p>

<p><span style="font-size: 16px;">先来看看两者不等同的一个例子:</span></p>
<p><span style="font-size: 16px;">在<span style="color: #0000ff;"><strong>文件一</strong></span>(file1.cpp)中含下列代码，<span style="color: #0000ff;">定义p为一个数组。</span></span></p>

{% highlight cpp %}
//file1.cpp
int p[5] = {1, 2, 3, 4, 5};
{% endhighlight %}


<p><span style="font-size: 16px;">在<strong><span style="color: #0000ff;">文件二</span></strong>(file2.cpp)中含下列代码，<span style="color: #0000ff;">p被声明为一个指针。</span></span></p>

{% highlight cpp %}
//file2.cpp
#include <iostream>
#include “file1.cpp”
using namespace std;

int main() {
extern int *p;
cout << p[1] << endl;
}
{% endhighlight %}

<p><span style="font-size: 16px;">对file2.cpp 编译，但<span style="color: #ff0000;"><strong>出现编译错误</strong></span>，如下:</span></p>

{% highlight cpp %}
g++ file2.cpp
错误: int *p与先前的的外部声明不匹配
{% endhighlight %}

<p><span style="font-size: 16px;">这个例子说明了数组和指针是会出现不等同的情况的。</span></p>
<p>&nbsp;</p>
<p><strong><span style="font-size: 16px;">那到底数组和指针在哪些情况下等同，哪些情况下不等同呢？</span></strong></p>
<p><span style="font-size: 16px;">一、在表达式中使用时，数组和指针可以互换使用。</span></p>
<p><span style="font-size: 16px;">二、在声明时</span></p>
<ol>
<li><span style="font-size: 16px;"><span style="color: #0000ff;">作为函数参数</span>， 数组<span style="color: #0000ff;">等同</span>于指针，如function(char a[]) 也可以写成function(char *a)；</span></li>
<li><span style="font-size: 16px;"><span style="color: #0000ff;">其他情况</span>，<span style="color: #ff0000;">定义和声明须匹配</span>(定义是特殊的声明)。如 上例中的 extern int p[5] 改写成指针形式 int *p 就出现了错误；</span></li>
</ol>
<p>&nbsp;</p>
<p><strong><span style="font-size: 16px;">为什么这么多人说数组就是指针呢？</span></strong></p>
<p><span style="font-size: 16px;">把上面这几点搞清楚之后，我们就能理解为什么人们会错误地理解数组和指针是可以完全互换。因为在初学编程时，人们总是把所有代码都放到main函数里。随着水平进步，人们开始把代码分别放到几个函数中。随着水平继续提高，人们终于学会了如何用几个文件来构造一个程序。所以在学编程的很长一段时间内，人们往往只是接触到指针和数组能够互换的情况。比如

{% highlight cpp %}
#include <stdio.h>

int main() {
char array[5] = "abcde";
char *p = array;
printf("%s %s\n", array, p);   //输出: abcd abcd
}
{% endhighlight %}

<p><span style="font-size: 16px;">上例清楚地展示了数组和指针的可互换性。但是人们<span style="color: #0000ff;">忽视了这只是发生在特定上下文环境</span>中，也就是作为函数调用的参数使用。<span style="color: #0000ff;"><strong>更糟糕的情</strong><strong>况</strong><strong>如下</strong></span>，</span></p>

{% highlight cpp %}
print("数组的地址是 %x, 对应的字符串是 %s\n", array, array);
//输出: 数组的地址是 bfa3d027, 对应的字符串是 abcd
{% endhighlight %}

<p><span style="font-size: 16px;">在上面这条语句中，<span style="text-decoration: underline;">数组名既作为一个地址(指针)出现， 又作为一个字符数组出现</span>，更是给人们带来误解&mdash;&mdash;&ldquo;喔，数组就是指针&rdquo;。但其实这条语句<span style="color: #0000ff;">之所以可行是因为printf 是一个函数</span>，而作为参数的array 其实是作为指针来传递的。类似的情况还在main函数中出现，main函数的参数 char **argv 和 char *argv[] 也可以互换。再加上平时常常能够见到在表达式中数组和指针可以交换，久而久之，人们便容易把数组和指针混淆起来。</span></p>
<p>&nbsp;</p>

<p><span style="font-size: 16px;"><strong>参考文献</strong></span></p>
<p><a href="http://book.douban.com/subject/2377310/" target="_blank">《C专家编程》</a>, Peter Van Der Linden 著, 徐波 译</p>
