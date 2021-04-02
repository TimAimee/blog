
## 作用域函数
	
	/**
	 * 范围函数 let also ,with run apply
	 * 区别在两点
	 * 一、 引用上下文是 it 还是 this
	 * 二、 返回值
	 *
	 */
	class ScopeFunction {
	    val TAG = "ScopeFunction"
	
	    /**
	     * T.let(block: (T) -> R): R
	     * T.also(block: (T) -> Unit)
	     *
	     * -----------let also--------- 作为lambda表达式的传入参数 使用 it ----------------------------------
	     * let 返回 lambda 表达式的结果
	     * also 的返回值是上下文对象本身。
	     *
	
	     * with(receiver: T, block: T.() -> R): R
	     * T.run(block: T.() -> R): R
	     * T.apply(block: T.() -> Unit): T
	     * ------------ with run apply --------作为 lambda 表达式的接收者 使用this----------------------------------
	     * apply 返回值是上下文对象本身
	     *
	     * run 及 with 返回 lambda 表达式的结果
	     */


	ScopeFunction: ---let--------
	ScopeFunction: Person(age=10, name='timaimee', city='china')
	ScopeFunction: Person(age=11, name='timaimee', city='HK')
	ScopeFunction: --注意看apple-let color=red--------

------------

	    /**
	     * T.let(block: (T) -> R): R
	     * let  作为lambda表达式的传入参数 , 返回值是lambda 表达式的结果
	     *
	     * 仅使用非空值执行代码块
	     *
	     * @param per
	     */
	    fun lets(per: Person) {
	        Log.i(TAG, "---let--------")
	        val age = per.let { it ->
	
	            Log.i(TAG, per.toString())
	            it.incrementAge()
	            it.moveto("HK")
	            Log.i(TAG, per.toString())
	
	            Apple("red")
	
	        }.color
	        Log.i(TAG, "--注意看apple-let color=${age}--------")
	    }
	

------------


	    /**
	     * T.also(block: (T) -> Unit): T
	     *
	     * also 作为lambda表达式的传入参数 , 返回值是上下文对象本身
	     *
	     * 并且用该对象执行以下操作
	     *
	     * @param per
	     */
	    fun alsos(per: Person) {
	        Log.i(TAG, "---also--------")
	        val age = per.also { it ->
	
	            Log.i(TAG, per.toString())
	            it.incrementAge()
	            it.moveto("Taiwang")
	            Log.i(TAG, per.toString())
	            Apple("red")
	
	        }.age
	        Log.i(TAG, "---注意看年龄是12，let age=${age}--------")
	
	    }

	ScopeFunction: Person(age=11, name='timaimee', city='HK')
	ScopeFunction: Person(age=12, name='timaimee', city='Taiwang')
	ScopeFunction: ---注意看年龄是12，let age=12--------

------------
	
	
	    /**
	     * with(receiver: T, block: T.() -> R): R
	     *
	     * with 作为 lambda 表达式的接收者，返回值 lambda 表达式的结果
	     *
	     *
	     * 引入一个辅助对象，其属性或函数将用于计算一个值。
	     *
	     * @param per
	     */
	    fun withs(per: Person) {
	        Log.i(TAG, "---with--------")
	        val apple = with(per) {
	            if (this.name.equals("timaimee")) {
	                Apple("timaimee love red")
	            } else {
	                Apple("other person love green")
	            }
	        }
	        Log.i(TAG, apple.toString())
	    }
	
	ScopeFunction: ---with--------
	ScopeFunction: Apple(color='timaimee love red')

------------

	    /**
	     * T.run(block: T.() -> R): R
	     * run 作为 lambda 表达式的接收者，返回值 lambda 表达式的结果
	     *
	     * @param per
	     */
	    fun runs(per: Person) {
	        Log.i(TAG, "---run--------")
	        val apple = per.run {
	            if (this.name.equals("tim")) {
	                Apple("timaimee love red")
	            } else {
	                Apple("other person love green")
	            }
	        }
	        Log.i(TAG, apple.toString())
	    }
	
	ScopeFunction: ---run--------
	ScopeFunction: Apple(color='other person love green')

------------
	
	    /**
	     * T.apply(block: T.() -> Unit): T
	     * apply 作为 lambda 表达式的接收者，返回值 返回值是上下文对象本身
	     *
	     * 将以下赋值操作应用于对象
	     *
	     * @param per
	     */
	    fun applys(per: Person) {
	        Log.i(TAG, "---apply--------")
	        val per = per.apply {
	            Log.i(TAG, per.toString())
	            name = "lijinliang"
	            if (this.name.equals("tim")) {
	                Apple("timaimee love red")
	            } else {
	                Apple("other person love green")
	            }
	        }
	        Log.i(TAG, per.toString())
	    }
	
	}

	ScopeFunction: ---apply--------
	ScopeFunction: Person(age=12, name='timaimee', city='Taiwang')
	ScopeFunction: Person(age=12, name='lijinliang', city='Taiwang')
