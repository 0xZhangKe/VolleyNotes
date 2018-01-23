### Volley

Volley is an HTTP library that makes networking for Android apps easier and, most
importantly, faster.

For more information about Volley and how to use it, visit the [Android developer training
page](https://developer.android.com/training/volley/index.html).
# VolleyNotes
本项目用于研究 Volley 的源码，直接 clone Volley 的仓库然后在其中添加 README 及各种注释。

## HttpStack
HttpStack 是个接口，只有下面这个：
```
HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders)
                            throws IOException, AuthFailureError;
```
传入请求对象及参数，返回相应数据，很简单。
## BaseHttpStack
BaseHttpStack 是个抽象类，实现了 HTTPStack 接口及其中的 performRequest 方法，同时增加了一个抽象方法：
```
public abstract HttpResponse executeRequest(
            Request<?> request, Map<String, String> additionalHeaders)
            throws IOException, AuthFailureError;
```
但是对于 performRequest 这个方法，并没有实现网络请求细节。
它会调用 executeRequest () 方法发起真正的网络请求，获取到返回
的 com.android.volley.toolbox.HttpResponse 对象之后将其转换成 org.apache.http.HttpResponse 对象。
## HurlStack
其中实现了具体发送网路请求的代码，继承 BaseHttpStack ，主要方法就是 executeRequest.
通过 Request 中的 header 及 body 使用 HttpURLConnection 发起网络请求，其流程如下：</p>
①循环将 Request 中的 header 及 additionalHeaders 放入 Header 中（connection.addRequestProperty）；</p>
②通过 request.getMethod() 设置请求函数（如 connection.setRequestMethod("POST") ）；
③判断 method 是否需要设置 body（GET 不需要，POST 需要等等）；
④如果需要设置 body，那调用 addBody 方法来设置，首先开启 doOutPut ，然后获取 Request 中对应的 ContentType 添加到 RequestProperty 中。
⑤开启输出流，将 body 输出到输出流中，关闭输出流；
⑥返回码为 -1 抛出异常；
⑦通过请求函数及响应吗判断是否存在 body，不存在将 header 从 Map 转成 List ，包装成不包含 body 的 HttpResponse 返回，结束。
⑧存在 body， 将 header 从 Map 转成 List ，获取输出流，有异常则获取异常输出流，包装成包含 body(InputStream) 的 HttpResponse 返回，结束。

## RequestQueue && NetworkDispatcher
Volley 并不是使用线程池来管理网络连接请求，而是使用线程数组：NetworkDispatcher[]。
默认情况下，Volley 会创建一个长度为 4 的线程数组，每个线程都是一个 NetworkDispatcher 对象，
其中会通过 while(true) 不停地从 RequestQueue 中的阻塞队列：mNetworkQueue 中取出 Request ，并通过 Network 发起网络请求。
所以一个 Volley 对象对应着四条工作线程，每一条都一直在运行，没有 Request 时就进入阻塞状态。
## Request
Request 是个泛型抽象类，其中泛型表示 HTTP 相应数据的类型（String、Json、Bitmap等）。
抽象方法即：
```
//将响应数据转换为对应的泛型对象
Response<T> parseNetworkResponse(NetworkResponse response);
//请求结束后将该响应发送至监听器（需要根据泛型响应对应类型的数据）
void deliverResponse(T response);
```
## StringRequest
响应数据为 String 类型的 Request，继承 Request，其中主要是就是实现了 parseNetworkResponse
方法，根据默认字符串类型（UTF-8）将 response 中的 data 转为 String，另外就是一些无关紧要的代码了。
除此之外还有 JsonRequest、JsonArrayRequest、JsonObjectRequest、ImageRequest等等，其中相差不大，就不做过多解释了。
