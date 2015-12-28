---
layout: post
title:  如何读懂复杂的C语言声明
date:   2014-02-25 19:29
category: "C/C++"
---

<p><span style="font-size: 16px;">C语言中有时会出现复杂的声明，比如</span>
{% highlight cpp %}
char* const*(*next) (); //这是什么东东
{% endhighlight %}
<p><span style="font-size: 16px;">如果你想了解如何读懂复杂的C语言声明，此文正适合你。</span>
</p>

<br />
<p><span style="font-size: 16px;">C语言变量的声明应始终<strong>遵守两点</strong>：</span></p>
<p><span style="font-size: 16px;">1. <span style="color: #0000ff;">声明和使用的语法尽量保持一致</span></span></p>
<p><span style="font-size: 16px;">例如:</span></p>

{% highlight cpp %}
#include <iostream>
using namespace std;

double (*fun)(double); //声明一个函数指针

int main() {
    fun=sin;
    doube reslut=(*fun)(0.5); //使用这个函数指针
    ...
}
{% endhighlight %}

<p><span style="font-size: 16px;">2. 声明语句的阅读<span style="color: #ff0000;">不是按照从左往右的阅读顺序</span>，而是<span style="color: #ff0000;">要根据各个符号的优先级进行阅读</span>的</span></p>
<p><span style="font-size: 16px;">这点非常重要！先列出C语言声明的优先级规则，再举个例子就能掌握求解方法了。</span></p>
<p>&nbsp;</p>
<p><span style="font-size: 16px;"><strong>C语言声明的优先级规则</strong></span></p>
<p><span style="font-size: 16px;">A 声明<span style="color: #0000ff;">从它的名字开始读取</span>，然后<span style="color: #0000ff;">按照优先级顺序</span>依次读取；</span></p>
<p><span style="font-size: 16px;">B 优先级从高到低依次是：</span></p>
<p><span style="font-size: 16px;">&nbsp;&nbsp;&nbsp; B.1 声明中被括号括起来的那部分；</span></p>
<p><span style="font-size: 16px;">&nbsp;&nbsp;&nbsp; B.2 后缀操作符：</span></p>
<p><span style="font-size: 16px;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 括号()表示这是一个函数，而</span></p>
<p><span style="font-size: 16px;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 放括号[]表示这是一个数组；</span></p>
<p><span style="font-size: 16px;">&nbsp;&nbsp;&nbsp; B.3 前缀操作符：星号*表示这是一个&ldquo;指向...的指针&rdquo;；</span></p>
<p><span style="font-size: 16px;">C 如果<span style="color: #0000ff;">const和（或）volatile</span>关键字的后面紧跟类型说明符（如int，long等），那么它作用于类型说明符。在其他情况下，它作用于关键字左边紧邻的指针星号。</span></p>

<p><span style="font-size: 16px;">需要强调的一个重要注意点是，对于<span style="color: #ff0000;">优先级: () &gt; [] &gt; *</span> 。</span></p>
<p><span style="font-size: 16px;">举例，用优先级规则分析C语言声明一例:</span></p>

{% highlight cpp %}
char * const * (*next) ();
{% endhighlight %}

<p><span style="font-size: 16px;">按照规则解读 char * const * (*next) ();</span></p>
<p><span style="font-size: 16px;">首先，(*next) 表示next是一个指针，它指向某个东东；</span></p>
<p><span style="font-size: 16px;">根据B.2和最右边的括号，next指向一个函数；</span></p>
<p><span style="font-size: 16px;">根据B.3，第二个星号表示，该函数返回一个指针；</span></p>
<p><span style="font-size: 16px;">char * const是函数返回的指针所指向的类型；</span></p>
<p><span style="font-size: 16px;">char* 是字符指针，const修饰左边的星号，即字符指针是常量的，该指针不可修改（该指针指向的字符内容是可修改的）；</span></p>
<p><span style="font-size: 16px;">综合地表述为，next是一个指针，它指向一个函数，该函数返回一个指针，该指针指向一个类型为char的常量指针。</span></p>
<p><span style="font-size: medium;"><span style="line-height: 24px;">还是不明白？再往下看就明白了！</span></span></p>
<p>&nbsp;</p>

<p><span style="font-size: 16px;">总结一下，分析复杂的C语言声明，要采用"<strong><span style="color: #0000ff;">由内而外，层层剥离</span></strong>&rdquo;的策略。</span></p>
<p><span style="font-size: 16px;"><span style="color: #0000ff;">从哪里开始</span>剥？从语句的最左边的标志符开始剥(上例为从next开始)。</span></p>
<p><span style="font-size: 16px;"><span style="color: #0000ff;">往哪个方向</span>剥？依照C语言的优先级规则一层层剥。</span></p>
<p>&nbsp;</p>

<p><span style="font-size: 16px;">再举一例作详细说明:</span></p>
<div class="cnblogs_code">
<pre><span style="font-size: 16px;"><span style="color: #0000ff;">char</span> *(* c[<span style="color: #800080;">10</span>]) (<span style="color: #0000ff;">int</span> **p);</span></pre>
</div>
<p><span style="font-size: 16px;">第一步，char *(* <span style="color: #ff0000;"><strong>c</strong></span>[10]) (int **p); &nbsp;最左边的标志符是c，表示"c是一个什么东东"；</span></p>
<p><span style="font-size: 16px;">第二步，char&nbsp;*(* &nbsp;<strong><span style="color: #ff0000;">[10]</span></strong>) (int **p); &nbsp;和[10]结合，表示"c是一个长度为10的数组";</span></p>
<p><span style="font-size: 16px;">第三步，char&nbsp;*(<strong><span style="color: #ff0000;">*</span></strong> &nbsp; &nbsp; &nbsp; &nbsp;) (int **p); &nbsp;和*结合，表示"这个数组存放着指针";</span></p>
<p><span style="font-size: 16px;">第四步，char&nbsp;* &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <strong><span style="color: #ff0000;">(int **p)</span></strong>; &nbsp;和(int **p)结合，表示"这个指针指向一个函数，函数的参数是二维指针";</span></p>
<p><span style="font-size: 16px;">第五步，char&nbsp;<strong><span style="color: #ff0000;">*</span></strong> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ; &nbsp;和 * 结合，表示"这个函数返回一个指针";</span></p>
<p><span style="font-size: 16px;">第六步，<strong><span style="color: #ff0000;">char</span></strong>　　　　　　　　 &nbsp; &nbsp; &nbsp;; 这个指针指向一个字符；</span></p>
<p>&nbsp;</p>
<p><span style="font-size: 16px;">把上面六步串起来，读作: c是一个数组[0..9]，它存放着指针，指针指向的函数参数是一个二维指针返回值是指向字符的指针。完工！</span></p>
<p>&nbsp;</p>

<p><span style="font-size: 16px;">注1: 这个方法若理解了，什么指针数组和数组指针、指针函数和函数指针等等之类的区别都是小菜一碟了。</span></p>
<p><span style="font-size: 16px;">注2: 合法的<span style="color: #000000;">声明中存在限制条件</span>。如<span style="color: #0000ff;">函数的返回值不能是一个函数，也不能是一个数组</span>，所以像function()()和function()[]是非法的，不能出现。</span><span style="font-size: 16px;"><span style="color: #0000ff;">数组里面能存函数指针，但不能存函数</span>，像int (* function[])()是合法的，function[]()则是非法的。</span></p>
<p>&nbsp;</p>

<p><span style="font-size: 16px;"><strong>参考文献</strong></span></p>
<p><a href="http://book.douban.com/subject/2377310/">《C专家编程》</a>, Peter Van Der Linden 著, 徐波 译</p>
