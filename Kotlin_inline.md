## inline 内联关键字

	
	
	/**
	 * java中的方法调用执行完成-->栈帧入栈、栈帧出栈的过程
	 *
	 * 内联函数
	 *
	 */
	class InlineFunction {
	
	    /**
	     * 案例1
	     *----------------------------------------------------------
	     */
	    fun test1() {
	        //多次调用 sum() 方法进行求和运算
	        println(sum(1, 2, 3))
	        println(sum(100, 200, 300))
	        println(sum(12, 34))
	        //....可能还有若干次
	    }
	
	    /**
	     * vararg 可变参数
	     * 求和计算
	     */
	    fun sum(vararg ints: Int): Int {
	        var sum = 0
	        for (i in ints) {
	            sum += i
	        }
	        return sum
	    }
	
	    /**
	     * 案例2， 为了避免多次调用 sum() 方法带来的性能损耗，我们期望的代码类似这样子的：
	     * ---------------------------------------------------------
	     * 虽然代码量增加了，但是性能确提升减少了调用sum函数的出栈入栈
	     *
	     */
	    fun test2() {
	        var sum = 0
	        for (i in arrayOf(1, 2, 3)) {
	            sum += i
	        }
	        println(sum)
	
	        sum = 0
	        for (i in arrayOf(100, 200, 300)) {
	            sum += i
	        }
	        println(sum)
	
	        sum = 0
	        for (i in arrayOf(12, 34)) {
	            sum += i
	        }
	        println(sum)
	    }
	
	    /**
	     * 案例3，内联就是为了解决这一问题。
	     * ---------------------------------------------------------
	     * 会自动把内联函数 sum_inline() 方法体内的代码，替换到调用该方法的地方。查看编译后的字节码，会发现 test() 方法里已经没了对 sum() 方法的调用,出现的都是方法体内的字节码了
	     */
	    fun test3() {
	        //多次调用 sum() 方法进行求和运算
	        println(sum(1, 2, 3))
	        println(sum(100, 200, 300))
	        println(sum(12, 34))
	        //....可能还有若干次
	    }
	
	    /**
	     * 减少了java栈的调用，直接在调用的地方把源码替换进去
	     *
	     * @param ints
	     * @return
	     */
	    inline fun sum_inline(vararg ints: Int): Int {
	        var sum = 0
	        for (i in ints) {
	            sum += i
	        }
	        return sum
	    }
	}
