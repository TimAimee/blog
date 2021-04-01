# takeif

## 源码实现

	public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? 
	    = if (predicate(this)) this else null

------------------

## 使用方法一

	// Original Code
	if (status) { doThis() }
	
	// Modified Code
	takeIf { status }?.apply { doThis() }


-------------------

## 使用方法二

	// Original code
	if (someObject != null && status) {
	   doThis()
	}
	
	// Improved code
	someObject?.takeIf{ status }?.apply{ doThis() }

-------------------

## 使用方法三

	// Original code
	if (someObject != null && someObject.status) {
	   doThis()
	}
	// Better code
	if (someObject?.status == true) {
	   doThis()
	}
	// Improved code
	someObject?.takeIf{ it.status }?.apply{ doThis() }

## 使用方法四
	
	val index 
	   = input.indexOf(keyword).takeIf { it >= 0 } ?: error("Error")
	val outFile 
	   = File(outputDir.path).takeIf { it.exists() } ?: return false	

## 注意事项

	//doThis() 方法一定会执行而无论 status 是 true 还是 false
	// Syntactically still correct. But logically wrong!!
	someObject?.takeIf{ status }.apply{ doThis() }
	
	//doThis() 方法不一定会执行，会判断status是否为空，是否为true
	// The correct one (notice the nullability check ?)
	someObject?.takeIf{ status }?.apply{ doThis() }



 


