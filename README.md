### Volley

Volley is an HTTP library that makes networking for Android apps easier and, most
importantly, faster.

For more information about Volley and how to use it, visit the [Android developer training
page](https://developer.android.com/training/volley/index.html).
# VolleyNotes
本项目用于研究 Volley 的源码，直接 clone Volley 的仓库然后在其中添加 README 各种注释。

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
但是对于 performRequest 这个方法，并没有具体的网络请求细节，