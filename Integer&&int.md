# Integer类
	
## 前言
	Integer是Java中最基本的类，但平时我们写代码的时候用的多的是int,我们知道Java中存在语法糖中的自动装箱与拆箱用于二者的替换
	面试中经常会问Integer跟int的区别，或者出一些Integer跟int是否相等的问题，
	对于这类问题，首先我们不能靠背，我们应该要去理解一下Integer类。

## int
	
	基本的数据类型，没什么好说的，4个字节，共32个bit

## Integer

	Integer是一个类，对于类来说肯定要实例化它，那么实例化Integer有3种方法 
	1.Integer.valueOf(100)	
	2.new Integer(100)	
	3.Integer=100
	其中第三种等同于第二种

对于Integer.valueOf()而言，我们看一下它的源码
	
	public static Integer valueOf(int i){
	    assert IntegerCache.high >= 127;
	    if (i >= IntegerCache.low && i <= IntegerCache.high){
	        return IntegerCache.cache[i + (-IntegerCache.low)];
	    }
	    return new Integer(i);
	}

我们会发现Integer存在一个IntegerCache，当你的值在-128到127之间时，会去IntegerCache中取。
	
那么于一个值而言，比如100和200，我们以下方式
	
	int value=100;
	Integer value=new Integer(100);
	Integer value=100;等同于Integer.value(100)
	Integer value=Integer.value(100);//从IntegerCache中取
	Integer value2=Integer.value(200);//通过new Integer（200）;

## Integer&&int的对比

### Integer==int

理解了上面的原理，假如我们实例化A,B均为相同的值，我们再讨论A==B的问题是不是就清晰多了。

	1.int && int:
		这种情况比较的是基本数据类型int，也就是值，返回true
		
	2.int && new Integer（）：
		这种情况new Integer（）,Integer类会先调用Integer.intValue()变成基本数据类型int，最终比较的是值，返回true

	3.int && Integer.value（）
		这种情况new Integer（）,Integer类会先调用Integer.intValue()变成基本数据类型int，最终比较的是值，返回true

	4.int && Integer
		等同于3，返回true
	--
	
	5.new Integer（） && new Integer（）
		这种情况new Integer（）,比较的是对象，最终比较的是地址，返回false
	
	6.new Integer（） && Integer.value（）
		这种情况new Integer（）,比较的是对象，最终比较的是地址，返回false
	
	7.new Integer（） && Integer
		等同于情况6，返回true

	8.Integer.value（） && Integer.value（），值在-128到127
		这种情况new Integer（）,比较的是对象，最终比较的是地址，取决于二者都是从IntegerCache中取，返回true

	9.Integer.value（） && Integer，值在-128到127
		等同于情况8，返回true

	10.Integer.value（） && Integer.value（），值大于127或少于-128
		这种情况new Integer（）,比较的是对象，最终比较的是地址，等同于new Integer（） && new Integer（），返回false

	11.Integer.value（） && Integer，值大于127或少于-128
		等同于情况10，返回true

### Integer.equals（Integer） 

	这种情况太简单了，equals比较的是值，只要值一样就返回true，但是值的注意是equals方法只有是类才存在，int是基本类型，不存在equals方法。
 
## 代码实例

看下代码打印的结果是否符合
	
	int A0 = 100;
	int B0 = 100;
	Integer C0 = 100;
	
	Integer E1 = new Integer(100);
	Integer E2 = new Integer(100);
	
	Integer F1 = Integer.valueOf(100);
	Integer F2 = Integer.valueOf(100);
	
	Integer G1 = Integer.valueOf(200);
	Integer G2 = Integer.valueOf(200);
	Integer G3 = 200;

打印结果
	
	--------==------
	
	1.int && int                          ->A0==B0,true
	2.int && new Integer                  ->A0==E1,true
	3.int && Integer.value                ->A0==F1,true
	4.int && Integer                      ->A0==C0,true

	5.new Integer && new Integer          ->E1==E2,false
	6.new Integer && Integer.value        ->E1==F1,false
	7.new Integer && Integer              ->E1==C0,false

	8.Integer.value && Integer.value(100) ->F1==F2,true
	9.Integer.value && Integer(100)       ->F1==C0,true

	10.Integer.value && Integer.value(200)->G1==G2,false
	11.Integer.value && Integer(200)      ->G1==G3,false

	--------equal------

	1.new Integer && new Integer          ->E1.equals(E2),true
	2.new Integer && Integer.value        ->E1.equals(F1),true
	3.new Integer && Integer              ->E1.equals(C0),true
	4.Integer.value && Integer.value(100) ->F1.equals(F2),true
	5.Integer.value && Integer(100)       ->F1.equals(C0),true
	6.Integer.value && Integer.value(200) ->G1.equals(G2),true
	7.Integer.value && Integer(200)       ->G1.equals(G3),true

Process finished with exit code 0

## 结论

最终我们不难发现，只有三类情况是返回true,

第一种任何实例与int==时都返回true,

第二种当对象与对象==时，只有都从IntegerCache中取（Integer，Integer.vauleOf()值在[-128,127]）才会返回true。

第三种就是对象间的equals,返回true

## 附加

	Integer a = 200;
	double b = 200;
	System.out.println(a == b); //true
	两者都会先拆箱比较值
