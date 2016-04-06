---
title: Volley使用说明
date: 2016-04-06 16:39:37
tags: [Volley,Android]
---


###1、使用Volley手册---多用组合，少用继承；针对接口编程，不针对具体实现编程。
1、基本的http通信是使用-即简单的`String`类型的Http请求按照如下的格式：  

		requestQueue mRequestQueue=Volley.newRequestQueue(Context); 

需要注意的地方在于：requestQueue为一个请求队列对象，可以缓存所有的Http请求，并且能按照一定算法并发的发出请求。本身的设计就是为适合高并发的操作，所以程序当中只需要建立一个RequestQueue的实例。  

---
例如请求`String`类型的`get`请求数据如下所示：
  
			StringRequest stringRequest = new StringRequest("http://www.baidu.com",  
                        new Response.Listener<String>() {  
                            @Override  
                            public void onResponse(String response) {  
                                Log.d("TAG", response);  
                            }  
                        }, new Response.ErrorListener() {  
                            @Override  
                            public void onErrorResponse(VolleyError error) {  
                                Log.e("TAG", error.getMessage(), error);  
                            }  
                        });  

需要使用`post`请求则使用一下的请求方式来进行操作：  

		StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener); 

需要注意的地方在于`StringRequest`中并没有提供设置`POST`参数的方法，但是当发出POST请求的时候，`Volley`会尝试调用`StringRequest`的父类——`Request`中的`getParams()`方法来获取POST参数，那么解决方法自然也就有了，我们只需要在StringRequest的匿名类中重写getParams()方法.  

		StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener) {  
	 	@Override  
	 	protected Map<String, String> getParams() throws AuthFailureError {  
        	Map<String, String> map = new HashMap<String, String>();  
        	map.put("params1", "value1");  
        	map.put("params2", "value2");  
        	return map;  
	    }  
	};  

对于Volley中支持的请求类型分为一下的几类：  
1、ImageRequest    
2、JsonArrayRequest  
3、JsonObjectRequest  
4、JsonRequest  
5、StringRequest  

###2.Volley中概念

Volley 的调用比较简单，通过 ` newRequestQueue(…)` 函数新建并启动一个请求队列RequestQueue后，只需要往这个RequestQueue不断 `add Request` 即可。

**Volley**：`Volley` 对外暴露的 API，通过 `newRequestQueue(…)` 函数新建并启动一个请求队列`RequestQueue`。

**Request**：表示一个请求的抽象类。`StringRequest`、`JsonRequest`、`ImageRequest` 都是它的子类，表示某种类型的请求。

**RequestQueue**：表示请求队列，里面包含一个`CacheDispatcher`用于处理走缓存请求的调度线程)、`NetworkDispatcher`数组(用于处理走网络请求的调度线程)，一个`ResponseDelivery`(返回结果分发接口)，通过 start() 函数启动时会启动`CacheDispatcher`和`NetworkDispatchers`。

**CacheDispatcher**：一个线程，用于调度处理走缓存的请求。启动后会不断从缓存请求队列中取请求处理，队列为空则等待，请求处理结束则将结果传递给`ResponseDelivery`去执行后续处理。当结果未缓存过、缓存失效或缓存需要刷新的情况下，该请求都需要重新进入`NetworkDispatcher`去调度处理。

**NetworkDispatcher**：一个线程，用于调度处理走网络的请求。启动后会不断从网络请求队列中取请求处理，队列为空则等待，请求处理结束则将结果传递给`ResponseDelivery`去执行后续处理，并判断结果是否要进行缓存。

**ResponseDelivery**：返回结果分发接口，目前只有基于`ExecutorDelivery`的在入参 `handler` 对应线程内进行分发。

**HttpStack**：处理 `Http` 请求，返回请求结果。目前 Volley 中有基于 `HttpURLConnection` 的`HurlStack`和 基于 `Apache HttpClient` 的`HttpClientStack`。

**Network**：调用HttpStack处理请求，并将结果转换为可被`ResponseDelivery`处理的`NetworkResponse`。

**Cache**：缓存请求结果，Volley 默认使用的是基于 sdcard `的DiskBasedCache`。`NetworkDispatcher`得到请求结果后判断是否需要存储在 `Cache`，`CacheDispatcher会从` Cache 中取缓存结果。  


----

* 所有的默认实现在`ToolBox`中体现。  

###3、核心类的介绍
1. **Volley使用**  
	`Volley` 主要有三个重载的静态的方法（其中前两个用的比较多）:
			
		public static RequestQueue newRequestQueue(Context context)

		public static RequestQueue newRequestQueue(Context context, HttpStack stack)
		
		public static RequestQueue newRequestQueue(Context context, int maxDiskCacheBytes) 

		调用newRequestQueue()执行 newRequestQueue(Context context, HttpStack stack, int maxDiskCacheBytes)构造器。具体实现如下：
		--------------------------------------------------------------------
		public static RequestQueue newRequestQueue(Context context, HttpStack stack, int maxDiskCacheBytes) {
			//设置缓存的文件目录
			File cacheDir = new File(context.getCacheDir(), "volley");

        	String userAgent = "volley/0";
	   	 	try {
	            String packageName = context.getPackageName();
	            PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
	            userAgent = packageName + "/" + info.versionCode;
	        } catch (NameNotFoundException e) {

	      	}
	        if (stack == null) {
	            if (Build.VERSION.SDK_INT >= 9) {
					//在Sdk版本2.1之后默认使用HttpUrlConnection中的HurlStack
	                stack = new HurlStack();
	            } else {
	                //2.1之前使用的Apach HttpClient作为默认实现
	                stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
	            }
	        }

	        Network network = new BasicNetwork(stack);
	        RequestQueue queue;
			//创建缓存文件大小
	        if (maxDiskCacheBytes <= -1){
	        	queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
	        } else{
	        	queue = new RequestQueue(new DiskBasedCache(cacheDir, maxDiskCacheBytes), network);
	        }
	
	        queue.start();
	
	        return queue;
		}
2. **Request**  
	代表网络请求抽象类,通过构建一个`Request`的非抽象子类(`StringRequest`、`JsonRequest`、`JsonRequest`或者自定义的对象)，并且将其加入到RequestQueue队列中来完成一次网络请求操作。  

	子类必须要重写的抽象方法为： 
 		
		<!--子类重写此方法，将网络返回的原生字节内容，转换成合适的类型。此方法会在工作线程中被调用。-->
		abstract protected Response<T> parseNetworkResponse(NetworkResponse response);

		<!--子类重写此方法，将解析成合适类型的内容传递给它们的监听回调。-->
		abstract protected void deliverResponse(T response);
		
		<!--重写此方法，可以构建用于 POST、PUT、PATCH 请求方式的 Body 内容。-->
		public byte[] getBody()

		<!--getBody函数没有被重写情况下，此方法的返回值会被 key、value 分别编码后拼装起来转换为字节码作为 Body 内容。-->
		protected Map<String, String> getParams()
	
3. **RequestQueue**  
	Volley 框架的核心类，将请求Request加入到一个运行的 `RequestQueue` 中，来完成请求操作。  
	主要成员变量：  

		<!--当前执行请求的队列-->
		private final Set<Request<?>> mCurrentRequests = new HashSet<Request<?>>();
		<!--当前请求队列中存在相同的并且是能够缓存的请求Url。-->
		private final Map<String, Queue<Request<?>>> mWaitingRequests = new HashMap<String, Queue<Request<?>>>();
	调用`start()`方法开启缓存执行调度线程`CacheDispatcher`和`N`个网络调度线程`NetWorkDispatcher`，这里的n可以根据Cpu核心数以及计算机性能进行配置。

4. **CacheDispatcher**  
	处理缓存的请求。启动之后不断的从后台取出缓存请求中取出请求处理。队列为空则执行等待。请求结束后将结果传递给`ResponseDelivery	`执行后续的处理操作。  
	如果结果未缓存过、缓存失效或者缓存需要刷新的情况下。该请求都是要重新进入到`NetWorkDispatcher`去调度处理。  
	成员变量：  
	1、`BlockingQueue<Request<?>> mCacheQueue` 缓存请求队列  
	2、`BlockingQueue<Request<?>> mNetworkQueue` 网络请求队列  
	3、`Cache mCache` 缓存类，代表了一个可以获取请求结果，存储请求结果的缓存  
	4、`ResponseDelivery mDelivery` 请求结果传递类  
	

###4、关于Http缓存--304

Volley 构建了一套相对完整的符合 Http 语义的缓存机制。
优点和特点
(1). 根据`Cache-Control`和`Expires`首部来计算缓存的过期时间。如果两个首部都存在情况下，以`Cache-Control`为准。
(2). 利用`If-None-Match`和`If-Modified-Since`对过期缓存或者不新鲜缓存，进行请求再验证，并处理 `304` 响应，更新缓存。
(3). 默认的缓存实现，将缓存以文件的形式存储在 Disk，程序退出后不会丢失。

我个人认为的不足之处
缓存的再验证方面，在构建`If-Modified-Since`请求首部时，Volley 使用了服务端响应的Date首部，没有使用`Last-Modified`首部。整个框架没有使用`Last-Modified`首部。这与 Http 语义不符。

	private void addCacheHeaders(Map<String, String> headers, Cache.Entry entry) {
    // If there's no cache entry, we're done.
    if (entry == null) {
        return;
    }

    if (entry.etag != null) {
        headers.put("If-None-Match", entry.etag);
    }

    if (entry.serverDate > 0) {
        Date refTime = new Date(entry.serverDate);
        headers.put("If-Modified-Since", DateUtils.formatDate(refTime));
    	}
	}
服务端根据请求时通过`If-Modified-Since`首部传过来的时间，判断资源文件是否在`If-Modified-Since`时间 以后 有改动，如果有改动，返回新的请求结果。如果没有改动，返回 `304 not modified`。
`Last-Modified`代表了资源文件的最后修改时间。通常使用这个首部构建`If-Modified-Since`的时间。
`Date`代表了响应产生的时间，正常情况下`Date`时间在`Last-Modified`时间之后。也就是`Date>=Last-Modified`。
通过以上原理，既然`Date>=Last-Modified`。那么我利用Date构建，也是完全正确的。

可能的问题出在服务端的 Http 实现上，如果服务端完全遵守 Http 语义，采用时间比较的方式来验证`If-Modified-Since`，判断服务器资源文件修改时间是不是在`If-Modified-Since`之后。那么使用Date完全正确。
可是有的服务端实现不是比较时间，而是直接的判断服务器资源文件修改时间，是否和`If-Modified-Since`所传时间相等。这样使用Date就不能实现正确的再验证，因为Date的时间总不会和服务器资源文件修改时间相等。

尽管使用Date可能出现的不正确情况，归结于服务端没有正确的实现 Http 语义。
但我还是希望Volley也能完全正确的实现Http语义，至少同时处理`Last-Modified`和`Date`,并且优先使用`Last-Modified`。

**已知存在的Bug**

(1). `BasicNetwork.performRequest(…)`

如下代码：

		@Override
		public NetworkResponse performRequest(Request<?> request) throws VolleyError {
    	……
   		while (true) {
        ……
        try {
            ……
         }catch (IOException e) {
            int statusCode = 0;
            NetworkResponse networkResponse = null;
            ……
            if (responseContents != null) {
              ……
            } else {
                throw new NetworkError(networkResponse);
            }
       	 }
    	}
	}
`BasicNetwork.performRequest(…)` 最后的`throw new NetworkError(networkResponse);`
应该是
`throw new NetworkError(e);`更合理。
