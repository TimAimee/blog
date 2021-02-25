
	在一个手机界面中，存在着交互逻辑（control）以及界面变化(view)，而界面中存在的数据都是我们现实世界的抽象出来的一个模型(model)。

# mvc
	
	这是一种最常见最简单的开发模式，在Activity类中同时存在交互逻辑（control）的细节以及界面变化(view)的处理，而model一般会定义一个单独bean类。

	mvc的优点是开发便捷，缺点是代码耦合比较严重，不方便单元测试，所以衍生出mvp的开发模式。

# mvp

	mvp的开发模式主要是为了分离交互逻辑（control）以及界面变化(view)
	
	1. 我们将Activity页面中存在的逻辑抽象出来定义一个接口IxxxPresent
	2. 我们将Activity页面中存在的界面变化出来定义一个接口IxxxView
	3. 构建一个IxxxPresentImp类，IxxxPresentImp类继承自接口IxxxPresent,IxxxPresentImp类声明一个构造方法,要求传入参数IxxxView
	4. 复写并实现接口的方法,在接口的方法内部可调用IxxxView的方法。
	5. xxxActivity类继承接口IxxxView，复写并实现接口的方法
	6. 在xxxActivity类的oncreat中实例化IxxxPresentImp对象
	7. 在xxxActivity类必要的地方去调用IxxxPresentImp对象的方法
	
	到这里一个mvp的模型就完成了。
	mvp的优点是交互逻辑（control）以及界面变化(view)的代码耦合度低，代码清晰度高，方便维护，可复用
	缺点是代码文件变多了，文件结构变复杂了。

# mvvm
	
 

 


