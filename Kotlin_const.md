# const 关键字

## const 必须修饰val

	const  只允许在top-level级别
	
	const val THOUSAND = 1000

------------------

	const 只允许在 object中声明

	object myObject {
	    const val constNameObject: String = "constNameObject"
	}

------------------
	
	const 只允许在 object中声明

	class MyClass {
	
	    companion object Factory {
	        const val constNameCompanionObject: String = "constNameCompanionObject"
	    }
	}

------------------

## const val和val区别

	const val 可见性为public final static，可以直接访问

------------------

	val 可见性为private final static，并且val 会生成方法getNormalObject()，通过方法调用访问。

## 总结

	当定义常量时，出于效率考虑，我们应该使用const val方式，避免频繁函数调用
