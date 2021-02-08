## Builder模式

	buidler模式的运用非常常见，在各大流行的三方框架中都能发现它的身影。

------
主要的调用如下：

	class MainTest{
		public static void main(String[] args) {
			Person reqeust=new Person.Builder().setName("Mom").setAge(40).setHeight(160).builder();
			System.out.println(reqeust.toString());
		}
	}

-------
分析如下：

	从调用来看new Person.Builder(),就说明要构造一个静态的Builder类，
	从方法.setAge(40) .setHeight(160)说明setxxx方法是可以累加的，也就是说，setxxx也要返回Builder类，
	最后的结束构造方法.builder()要返回Person类，而此前并没有实例化Person,
	所以要在Builder类中的builder方法中通过传递Builder来实例化Person，
	也就是说Person类要留有一个构造方法来传递Builder类来实例化，至此分析结果，再在Person类添加set/get,toString方法。

------
具体实现如下：

	class Person{
		private int age;
		private String name;
		private int height;
	
		public Request(Builder builder){
			this.age=buidler.age;
			this.name=builder.name;
			this.height=builer.height;
		}
	
		public String getName(){
			return name;
		}
		public String getHeight(){
			return height;
		}
		public String getAge(){
			return age;
		}
	
		@Override
		public String toString(){
				return "name="+name
				+"\nage="+age
				+"\nheight="+height;
		}
	
		static class Builder{
			public int age;
			public String name;
			public int height;
	
			public Buidler(){
			
			}
			public Buidler(int height){
				this.height=height;
			}
			public Buidler setAge(int age){
				this.age=age;
				return this;
			}
			public Buidler setName(String name){
				this.name=name;
				return this;
			}
			public Buidler setHeight(int height){
				this.height=height;
				return this;
			}
	
			public Person builder(){
				new Person(this);
			}
		}
	
	}
	


