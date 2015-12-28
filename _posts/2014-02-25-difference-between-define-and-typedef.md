---
layout: post
title:  typedef 和 define 的区别
date:   2014-02-25 11:46
category: "C/C++"
---

<p><span style="font-size: 16px;">typedef 和#define 都经常用来定义一个关键字或标志符的别名，但他们之间有本质的区别。</span></p>

<p>&nbsp;</p>
<p><span style="font-size: 16px;">typedef&nbsp;</span>是语言<span style="color: #ff0000;">编译过程</span>的一部分；</span></p>
<p><span style="font-size: 16px;">#define是<span style="color: #ff0000;">宏定义语句</span>，它本身并不在编译过程中进行，而是在这之前的<span style="color: #0000ff;">预处理过程</span>就已经完成了。</span></p>

<p><span style="font-size: 16px;"><strong>要理解两者的关键区别，可以这么来考虑</strong>:</span></p>
<p><span style="font-size: 16px;">把typedef 看成是一种<span style="color: #0000ff;">彻底的"封装"类型</span>，<u>相当于产生了一个新的变量类型</u>(这点与struct 类似，待会会与struct 进行类比来增进理解)。而#define 仅仅是进行<span style="color: #0000ff;">宏文本替换</span>。</span></p>
<p>&nbsp;</p>
<p><span style="font-size: 16px;">typedef 和 #define 的<strong>区别体现在两个方面</strong>:</span></p>
<p><span style="font-size: 16px;"><strong>首先</strong>， 可以用其他类型说明符对宏类型名<span style="color: #0000ff;">进行扩展</span>，但typedef 所定义的类型名却<span style="color: #0000ff;">不能进行扩展</span>。</span></p>
<p><span style="font-size: 16px;">如下:</span></p>

{% highlight cpp %}
#define peach int
unsigned peach i;    //没问题，可以这么使用。

typedef int banana;
unsigned banana i;   //错误，非法！
{% endhighlight %}

<p><span style="font-size: 16px;">#define 仅仅是进行宏文本替换，它并没有产生新的变量类型。以上面代码为例，<u>在预处理阶段， peach 重新被替换为 int</u>, 故可在peach 前面加unsigned；</span></p>
<p><span style="font-size: 16px;">而typedef 的工作方式则和#define完全不一样，它相当于产生了一个新的变量类型，这个新的变量类型前面不能再进行扩展。这和struct 类型类似，定义了一个struct 也相当于产生了一个新的变量类型，<u>而我们从不会见到类似于 unsigned struct student a; 这种声明</u>。</span></p>
<p>&nbsp;</p>
<p><span style="font-size: 16px;"><strong>第二</strong>个区别，在连续几个变量的声明中，用typedef 定义的类型能够<span style="color: #0000ff;">保证声明中的所有变量均为同一种类型</span>，而用<span style="color: #0000ff;">#define 则无法保证</span>。</span></p>

{% highlight cpp %}
#define intp int *
intp a, b;
{% endhighlight %}

经过宏扩展，第二行变为: int * a, b;

<p><span style="font-size: 16px;">a是一个指向int变量的指针变量，<span style="color: #ff0000;">b是一个int型变量！</span></p>

{% highlight cpp %}
typedef int * int_p;
int_p c, d; //c, d都是指向int变量的指针变量！
{% endhighlight %}

<p><span style="font-size: 16px;">以上述代码为例，#define仅仅是进行宏替换，在预处理阶段intp被替换为 int *，这显然只能声明a是个指针变量， 而b是int型变量。typedef则相当于定义了一个新的变量类型，能对c、d都起作用。同样与struct作类比， 对于 struct student e,f; e和f都被声明是student类型的变量。</span></p>
<p>&nbsp;</p>

<p><span style="font-size: 16px;"><strong>参考文献</strong></span></p>
<p><a href="http://book.douban.com/subject/2377310/" target="_blank">《C专家编程》</a>, Peter Van Der Linden 著, 徐波 译</p>
