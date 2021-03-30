## Okhttp原理
----
Okhttp调用方式如下

	Request request = new Request.Builder()
	      .url(BASE_URL + "/date")
	      .build();
	
 	OkHttpClient client = new OkHttpClient.Builder()
	        .addInterceptor(new HttpLoggingInterceptor())
	        .cache(new Cache(cacheDir, cacheSize))
	        .build();
	
	Call call = client.newCall(request);
    Response response = call.execute();

----
从调用来看
# Builder模式
	Request和OkHttpClient的设计都是Builder模式实现的,Request是网络中的请求和Response是网络中的响应，

----

	仔细分析下Okhttp是分层设计的，它设计了好几层的interceptor拦截器，
	
	RetryAndFollowUpInterceptor(重连拦截器，根据情况来重连)
	BridgerInterceptor(桥接和适配拦截器，请求当中缺少的http头)
	CacheInterceptor(缓存拦截器，处理缓存功能)
	ConnectInterceptor（连接拦截器，建立可用的连接）
	CallServerInterceptor（调用服务拦截器，负责将request写入网络流当中，将response返回）
	
	为了将拦截器串连起来，它设计了一个chain链条（责任链模式）来实现。


# 责任链模式
----
拦截器的接口和链条的接口

	public interface IChain {
	    Request getRequest();
	    Response process(Request request);
	}


	public interface Interceptor {
	    Response interceptor(IChain chain);
	}
	
---
拦截器实现类

	public class RealChain implements IChain {
	    public final static String TAG = RealChain.class.getSimpleName();
	    private int index = 0;
	    private List<Interceptor> interceptorsList;
	    Request request;
	
	    public RealChain(int index, List<Interceptor> interceptorsList, Request request) {
	        this.index = index;
	        this.interceptorsList = interceptorsList;
	        this.request = request;
	    }
	
	    @Override
	    public Request getRequest() {
	        return request;
	    }
	
	    @Override
	    public Response process(Request request) throws AssertionError {
	        Log.i(TAG, "process");
	        return proceed(request, "");
	    }
	
	    private Response proceed(Request request, String s) throws AssertionError {
	        if (index >= interceptorsList.size()) {
	            return new Response("1");
	        } else {
	            Log.i(TAG, "index:" + index);

				//通过index的增长，来获取下一个interceptor

	            RealChain nextRealChain = new RealChain(index + 1, interceptorsList, request);
	            Interceptor interceptor = interceptorsList.get(index);
	
	            Response response = interceptor.interceptor(nextRealChain);
	
	            return response;
	        }
	    }
	}

----
设计两个拦截器


	public class AInterceptor implements Interceptor {
	    public static final String TAG = AInterceptor.class.getSimpleName();
	
	    @Override
	    public Response interceptor(IChain chain) {
	        Request request = chain.getRequest();

			//取到request,可以对request进行操作

	        Log.i(TAG, "interceptor");
	        Response response = chain.process(request);

			//取到response,可以对response进行操作

	        Log.i(TAG, "response:"+response.toString());
	        return response;
	    }
	}

	public class BInterceptor implements Interceptor {
	    public static final String TAG=BInterceptor.class.getSimpleName();
	    @Override
	    public Response interceptor(IChain chain) {
	        Request request = chain.getRequest();

			//取到request,可以对request进行操作

	        Log.i(TAG, "interceptor");
	        Response response = chain.process(request);

			//取到response,可以对response进行操作

	        Log.i(TAG, "response:"+response.toString());
	        return response;
	    }
	}

----
OkHttpClient调用

    public void newCall(Request request ) {
	    Log.i(TAG, "newCall");
        int index = 0;
        List<Interceptor> interceptorsList = new ArrayList<>();
        interceptorsList.add(new AInterceptor());
        interceptorsList.add(new BInterceptor());

        RealChain realChain = new RealChain(index, interceptorsList, request);
        try {
            Response response = realChain.process(request);
            Log.i(TAG, response.toString());
        } catch (AssertionError e) {

        }
    }

----
日志返回

	02-08 14:07:34.119 16964-16964/com.timaimee.demohband I/ChainActivity: newCall
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/RealChain: process
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/RealChain: index:0
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/AInterceptor: interceptor
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/RealChain: process
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/RealChain: index:1
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/BInterceptor: interceptor
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/RealChain: process
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/BInterceptor: response:Response{}
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/AInterceptor: response:Response{}
	02-08 14:07:34.120 16964-16964/com.timaimee.demohband I/ChainActivity: Response{}



## Okhttp的缓存

	Okhttp的是LRU缓存,内部是由LinkedHashMap实现，LinkedHashMap是一个双向链表，最常使用的元素会放到链表的尾部，链表的头部为最不常用的数据，新元素也会插入到尾部。
	
	LinkedHashMap.put->HashMap.put->jdk1.8中是插入尾部的。

	当cache的大小超过定义的大小时，会回收缓存 

## 连接池
	
	创建okhttpClient开始就会有生成一个连接池，允许5个连接，空闲连接最多存活5分钟
	
## transmitter
	
	1.new call()            创建 transmitter
	2.excute                传入 transmitter
	3.retryIntercepter      调用 transmitter.prepareconnect (判断连接复用，释放已有连接，创建新连接，生成路由选择，准备代理服务器等)
	4.connectIntercepter    调用 transmitter.excahnge, 主要是socket操作connect
	5.callserverIntercepter 主要是 socket write,socket input
