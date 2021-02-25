# static
	
## 对static的理解

	static是Java类中的静态描述符，static的变量会存在jvm开放的静态区域里面，供所有的对象访问。

### 1.它可以修饰变量

	public class Person{
		static int value=5;
	}

	调用时，可以直接Person.value,无需实例化person对象

### 2.可以修饰方法块
	
	方法块的全局变量以及调用的方法也必须是static修饰的。

	public class Person{
		static double value=10;
		public static void tellHello(){
			value=value+10;
			return addSum（value,value）
		}
	
		public static void addSum(double v1,double v2){
			return v1+v2;
		}
	}

	调用时，可以直接Person.tellHello（）,无需实例化person对象

### 3.可以修饰内部类，也被称为静态内部类
	
	public class Person{
		int age;
		static class Watch{
			String wachName;
		}
	}
	
	

## 2.静态内部类和内部类的区别

非静态内部类

	内部类是和外部类相互绑定的，内部类可以访问外部的private变量，private方法，private静态方法，
	关系相当于公交车和公交车方向盘,公交车方向盘只能通过公交车去获取
	
	public class OutClass {
	    private int x;
	    private static int y;
	    public int z;
	
	    private int getX() {
	        return x;
	    }
	
	    public void setX(int x) {
	        this.x = x;
	    }
	
	    public static int getY() {
	        return y;
	    }
	
	    public static void setY(int y) {
	        OutClass.y = y;
	    }
	
	    class InnerClass {
        	static int hell0=10;//报错，非静态内部类不允许定义静态成员
	        private int v;
	        public int get1(){
	            return x;
	        }
	        private int get2(){
	            return y;
	        }
	        private int get3(){
	            return z;
	        }
	        private int get4(){
	            return getX();
	        }
	    }
	}


如何使用

        OutClass out =new OutClass();
        out.setX(20);
        OutClass.InnerClass innerClass= out.new InnerClass();
        System.out.println(innerClass.get1());
		
		打印结果：20	

		原因是innerClass里面保有outClass.this的隐式引用



静态内部类

	静态内部类是和外部类关联不紧密的，相当于公交车和公交车的乘客，相互独立。
	公交车的乘客无需通过公交车获取

	内部类可以访问外部的private静态变量，不可以访问外部的非静态变量，
	内部类可以访问外部的private静态方法，不可以访问外部的非静态方法，
	
	public class OutCls {
	    private static int x = 3;
	    public int y;
	
	    public int getY() {
	        return y;
	    }
	
	    public void setY(int y) {
	        this.y = y;
	    }
	
	    private static int getX() {
	        return x;
	    }
	
	    public void setX(int x) {
	        this.x = x;
	    }
	
	    static class InnerStatic {
	        public int get1() {
	            return getX();
	        }
	
	        public int get2() {
	            return getY();//报错，不可以访问外部的非静态方法，
	        }
	        public int get3() {
	            return y;     //报错，不可以访问外部的非静态变量，
	        }
	    }
	}


如何使用

        OutCls innerStaticClass=new OutCls();
        innerStaticClass.setX(10);

        OutCls.InnerStatic inner=new OutCls.InnerStatic();
        System.out.println(inner.get1());

		打印结果：10

## 3.面试
	
面试中常会遇到几类问题

### 1.static的启动顺序问题

	public class TestStatic{
		public static void main(String[] args){
			Person person=new Person();
		}
	}
	public class Person{
		private int age;
		static{
			System.out.println("person static");
		}
		public Person(){
			System.out.println("Person construct");
		}
	}
	
	public class Doctor extend Person{
		private int workyear;
		static{
			System.out.println("Doctor static");
		}
		public Doctor(){
			System.out.println("Doctor construct");
		}
	}

### 2.static的值问题

	public class TestStatic{
		public static void main(String[] args){
			Person.age=1;
			Person person1=new Person();
			person1.age=10;
			Person person2=new Person();
			person2.age=100;
		}
	}

	public class Person{
		static int age;
		public Person(){
			System.out.println("Person construct");
		}
		public int getAge(){
			return age;
		}
	}
