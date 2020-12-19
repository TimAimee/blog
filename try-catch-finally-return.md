# try-catch-finally-return
	
## 前言

	//可能是我们村写的最好的
	try-catch-finally-return的问题一直萦绕在我的心中，正所谓念念不忘，必有回想，想一次性解决这个疑难杂症，

	讨论这个问题时，我们要思考几个问题，并假定好条件
	
	1. finally是否一定会执行?
	2. 讨论的问题是try-catch-finally-catch块中的基本数据类型还是引用数据类型（如int和StringBuffer）
	

## 1.finally是否会执行?
	
	对于大部分文章都说，无论try..catch中是否包含return，finally都一定会执行。但是还是存在以下极端突发情况会导致finally不会执行。
	1. try..catch..语句中执行System.exit();
	2. 守护线程被中止
	3. jvm崩溃
	4. 突然的断电

 

## 2.我们讨论是基本的数据类型int
	
最简单的情况

	public void sayXx(){
		int value=0;
		try{
			value=value+10;//Step1
			......//未知语句
			value=value+100;//Step2	
		}catch(Exception e){		
			value=value+1000;//Step3	
		}finally{
			value=value+10000;//Step4	
		}
		return value;
	}

	我们来讨论最简单的情况，不包含return语句的情况,
	当...执行异常,sayXx()=11010:Step1->Step3->Step4 
	当...执行正常,sayXx()=11110:Step1->Step2->Step3->Step4

接下讨论的情况比较复杂，包含【正常，异常,2种情况】，【1个Return，2个Return，3个Return，6种情况】，
一共12种情况，不用担心情况太多，其实耐心看完就发现很简单
	
包含return，且try语句正常的情况

第一种 try-catch-finally,只有一个return,在try块中 

	public int sayXx(){
		int value=0;
		try{
			value=value+10;
			System.out.println("Step1->"+value);
			......//未知语句
			value=value+100;	
			System.out.println("Step2->"+value);
			return ++value;//+1再返回
		}catch(Exception e){
			value=value+1000;	
			System.out.println("Step3->"+value);
		}finally{
			value=value+10000;	
			System.out.println("Step4->"+value);
		}
        return value;
	}

	当...执行正常，catch语句形同虚设，打印结果如下:

	Step1->10
	Step2->110
	Step4->10111
	sayXx->111

	当...执行异常，catch语句执行，打印结果如下:

	Step1->10
	Step3->1010
	Step4->11010
	sayXx->11010

第二种 try-catch-finally，只有一个return,在catch块中;

	public int sayXx(){
		int value=0;
		try{
			value=value+10;
			System.out.println("Step1->"+value);
			......//未知语句
			value=value+100;	
			System.out.println("Step2->"+value);
		}catch(Exception e){
			value=value+1000;	
			System.out.println("Step3->"+value);
			return value++;//返回再+1
		}finally{
			value=value+10000;	
			System.out.println("Step4->"+value);
		}
        return value;
	}

	当...执行正常，catch语句执行，打印结果如下:

	Step1->10
	Step2->110
	Step4->10110
	sayXx->10110

	当...执行异常，catch语句执行，打印结果如下:

	Step1->10
	Step3->1010
	Step4->11011
	sayXx->1010

第三种 try-catch-finally，只有一个return,在finally块中;

	public int sayXx(){
        int value=0;
        try{
            value=value+10;
            System.out.println("Step1->"+value);
			......//未知语句
            value=value+100;
            System.out.println("Step2->"+value);
        }catch(Exception e){
            value=value+1000;
            System.out.println("Step3->"+value);
        }finally{
            value=value+10000;
            System.out.println("Step4->"+value);
            return ++value;//+1再返回
        }
		//return value; 编译器显示此行多余
	}

	当...执行正常，catch语句执行，打印结果如下:

	Step1->10
	Step2->110
	Step4->10110
	sayXx->10111

	当...执行异常，catch语句执行，打印结果如下:

	Step1->10
	Step3->1010
	Step4->11010
	sayXx->11011

第四种 try-catch-finally，2个return,在try,catch中
	
	public int sayXx(){
        int value=0;
        try{
            value=value+10;
            System.out.println("Step1->"+value);
            ......//未知语句
            value=value+100;
            System.out.println("Step2->"+value);
            return ++value;
        }catch(Exception e){
            value=value+1000;
            System.out.println("Step3->"+value);
            return ++value;
        }finally{
            value=value+10000;
            System.out.println("Step4->"+value);
        }
		//return value; 编译器显示此行多余
    }
	
	当...执行正常，catch语句执行，打印结果如下:

	Step1->10
	Step2->110
	Step4->10111
	sayXx->111

	当...执行异常，catch语句执行，打印结果如下:

	Step1->10
	Step3->1010
	Step4->11011
	sayXx->1011


第五种 try-catch-finally，2个return,在try,finally中

    public int sayXx() {
        int value = 0;
        try {
            value = value + 10;
            System.out.println("Step1->" + value);
            ......//未知语句
            value = value + 100;
            System.out.println("Step2->" + value);
            return ++value;
        } catch (Exception e) {
            value = value + 1000;
            System.out.println("Step3->" + value);
        } finally {
            value = value + 10000;
            System.out.println("Step4->" + value);
            return ++value;
        }
		//return value; 编译器显示此行多余
    }

	当...执行正常，catch语句执行，打印结果如下:

	Step1->10
	Step2->110
	Step4->10111
	sayXx->10112【在这里，我们发现，try return之后，又在finally中再一次return,并把finally的值加上了，也就是说2个return都执行了】

	当...执行异常，catch语句执行，打印结果如下:

	Step1->10
	Step3->1010
	Step4->11010
	sayXx->11011


第六种 try-catch-finally，2个return,在catch,finally中

    public int sayXx() {
        int value = 0;
        try {
            value = value + 10;
            System.out.println("Step1->" + value);
         	......//未知语句
            value = value + 100;
            System.out.println("Step2->" + value);
        } catch (Exception e) {
            value = value + 1000;
            System.out.println("Step3->" + value);
            return ++value;
        } finally {
            value = value + 10000;
            System.out.println("Step4->" + value);
            return ++value;
        }
		//return value; 编译器显示此行多余
    }

	当...执行正常，catch语句执行，打印结果如下:

	Step1->10
	Step2->110
	Step4->10110
	sayXx->10111

	当...执行异常，catch语句执行，打印结果如下:

	Step1->10
	Step3->1010
	Step4->11011
	sayXx->11012


第七种 try-catch-finally，3个return,在try，catch,finally中

    public int sayXx() {
        int value = 0;
        try {
            value = value + 10;
            System.out.println("Step1->" + value);
         	......//未知语句
            value = value + 100;
            System.out.println("Step2->" + value);
            return ++value;
        } catch (Exception e) {
            value = value + 1000;
            System.out.println("Step3->" + value);
            return ++value;
        } finally {
            value = value + 10000;
            System.out.println("Step4->" + value);
            return ++value;
        }
		//return value; 编译器显示此行多余
    }

	当...执行正常，catch语句执行，打印结果如下:

	Step1->10
	Step2->110
	Step4->10111
	sayXx->10112

	当...执行异常，catch语句执行，打印结果如下:

	Step1->10
	Step3->1010
	Step4->11011
	sayXx->11012

第八种 之前讨论的错误都是在try中，现在在catch中也报错

    public int sayXx() {
        int value = 0;
        try {
            value = value + 10;
            System.out.println("Step1->" + value);
            int i=1/0;
            value = value + 100;
            System.out.println("Step2->" + value);
            return ++value;
        } catch (Exception e) {
            value = value + 1000;
            System.out.println("Step3->" + value);
            int i=1/0;
            return ++value;
        } finally {
            value = value + 10000;
            System.out.println("Step4->" + value);
            return ++value;
        }
    }

	打印结果如下:

	Step1->10
	Step3->1010
	Step4->11010
	sayXx->11011



## 3.我们讨论是引用数据类型StringBuffer
	
    //注意：返回StringBuffer跟String差别很大，String会当作基本类型处理，最后的finally语句会执行，但不会把finally值带上。
    public StringBuffer sayXx() {   						  
        StringBuffer stringBuffer = new StringBuffer();
        try {
            stringBuffer.append("try1");
            int i=1/0;
            stringBuffer.append(",try2");
            return stringBuffer;
        } catch (Exception e) {
            stringBuffer.append(",catch");
            return stringBuffer;
        } finally {
            stringBuffer.append(",finally");
        }
    }


	当...执行正常，catch语句执行，打印结果如下:

	sayXx->try1,try2,catch,finlly

	当...执行异常，catch语句执行，打印结果如下:

	sayXx->try1,catch,finlly

	我们发现当有返回引用类型时，会将finally值也带上。

## 结论
	
	对于基本类型而言
	
	1. 1个return时，try,catch，finally语句中，会返回对应块的返回值，finally块一定会执行，
	   若finally块存在表达式计算的，只有是return在finally块中，才会被返回
	
	2. 2个return时，若在finally块中存在return,它会抢断try,catch中的return
	
	3. 3个return时，理解了前面两个，这个就很好理解了。
	
	4. try,catch中的return后面包含表达式或函数时，会先执行表达式或函数，得到值1，然后执行finally，得到值2，
	   若finally块存在return,返回值2  
	   若finally块不存在return,返回值1

	对于引用类型而言
	
	1. 走正常的try..catch..finally，finally块的值会被带出去。


## 原理
	
	jvm,局部变量,引用地址，子例程,
