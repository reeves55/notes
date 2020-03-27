# Java HTTP -1

form表单提交的数据，如果是通过 ```application/x-www-form-urlencoded``` 类型发送的，就是把表单数据转换成一个字符串，这个字符串把表单所有控件的值都囊括到一起，作为消息内容发送到服务端。而 ```multipart/form-data``` 不一样，这种类型下，消息内容中表单每个控件的值都是分隔开来的，各自成为一块。







##Java HttpUrlConnection发送请求





### 1. 发送 multipart/form-data 格式请求

主要的有两个点，一个是Content-Type HTTP header需要设置成

 ```Content-Type: multipart/form-data;boundary="boundary"``` ；

另一个是HTTP request body的格式要设置成：

```html
--boundary 
Content-Disposition: form-data; name="field1" 
Content-Type: XXX

value1 
--boundary 
Content-Disposition: form-data; name="field2"; filename="example.txt" 
Content-Type: XXX

value2

......

--{boundary}--\r\n (<-这个是结尾)
```



其中，```Content-Disposition``` 指明了这个part的相关信息

```
Content-Disposition: form-data; [name="xxx";] [filename="xxx"]
```











### 2. 发送 application/x-www-form-urlencoded 格式请求

要发送这种格式的http请求，最重要的就是设置http request body的格式是 param=value&...这种格式，并且整个字符串要使用url encode进行编码

```java
try{
  // create http connection from url
	String requestUrl = "<Url here>";
	String urlParameters  = "param1=data1&param2=data2&param3=data3";
  // url encode
  urlParameters = URLEncoder.encode(urlParameters, StandardCharsets.UTF_8);
	byte[] postData = urlParameters.getBytes();
	URL url = new URL(requestUrl);
	HttpURLConnection conn= (HttpURLConnection) url.openConnection(); 

	// set http request method
	conn.setRequestMethod("POST");

	// set http request header
	conn.setRequestProperty("Content-Type", "application/x-www-form-urlencoded"); 
	conn.setRequestProperty("charset", "utf-8");

	// indicate http has request body
	conn.setDoOutput(true);
  // indicate will read response body
  conn.setDoInput(true);

	// send http request body
	BufferedOutputStream wr = new BufferedOutputStream (connection.getOutputStream ());
	wr.write(postData);
	wr.flush ();

	// get response status code
  int statusCode = conn.getResponseCode();
  if (statusCode != 200) {
    return;
  }
  
  HttpResponse response = new HttpResponse();
  response.setHeaders(conn.getHeaderFields());
  response.setStatus(statusCode);
  
  // get response body
	InputStream is = connection.getInputStream();
  if (is != null) {
       ByteArrayOutputStream outStream = new ByteArrayOutputStream();
       byte[] buffer = new byte[1024];
       int len = 0;
       while ((len = is.read(buffer)) != -1) {
           outStream.write(buffer, 0, len);
       }
       response.setBody(outStream.toByteArray());
  }
  
} catch (IOException e) {
  
} finally {
  if(wr != null) {
    try{
      wr.close();
    }catch(IOException e) {
      e.printStackTrace();
    }
  }
  
  if(response != null) {
    try {
      response.close();
    }catch(IOException e) {
      e.printStackTrace();
    }
  }
}


```



上面代码使用了自定义的HttpResponse类：

```java
public class HttpResponse {
    private Map<String, List<String>> headers;
    private byte[] body;
    private String encoding;
    private int status;

    public HttpResponse() {
    }

    public Map<String, List<String>> getHeaders() {
        return headers;
    }

    public void setHeaders(Map<String, List<String>> headers) {
        this.headers = headers;
    }

    public byte[] getBody() {
        return body;
    }

    public void setBody(byte[] body) {
        this.body = body;
    }

    public String getEncoding() {
        return encoding;
    }

    public void setEncoding(String encoding) {
        this.encoding = encoding;
    }

    public int getStatus() {
        return status;
    }

    public void setStatus(int status) {
        this.status = status;
    }
}
```









并不是所有的XXXOutputStream都有Java代码级别的缓存，BufferedOutputStream就有缓存，它的flush方法会调用outputstream.write方法，把缓存的数据都写到流，但是DataOutputStream的flush方法就是直接调用OutputStream的flush方法，默认什么都不做。



Java HttpUrlConnection的connect()方法，是发起连接的意思，URL.openConnection()只是创建了一个HttpURLConnection请求对象，调用HttpURLConnection对象的getOutputStream()方法和getInputStream()方法都会隐式调用HttpURLConnection的connect()方法。







### application/x-www-form-urlencoded 字符编码问题

这种类型的请求中，请求体，也就是消息内容部分的字符是经过url 编码后的字符，规范规定了，url 编码字符集包括：

> Url中只允许包含英文字母（a-zA-Z）、数字（0-9）、-_.~4个特殊字符以及所有保留字符，其中，保留字符包括 !	*	'	(	)	;	:	@	&	=	+	$	,	/	?	#	[	]

对于允许范围之外的字符，则需要使用允许的字符集对其进行编码。



ASCII码中保留字符的编码：

| !     | *     | "     | '     | (     | )     | ;     | :     | @     | &     |
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- |
| `%21` | `%2A` | `%22` | `%27` | `%28` | `%29` | `%3B` | `%3A` | `%40` | `%26` |
| **=** | **+** | **$** | **,** | **/** | **?** | **%** | **#** | **[** | **]** |
| `%3D` | `%2B` | `%24` | `%2C` | `%2F` | `%3F` | `%25` | `%23` | `%5B` | `%5D` |

ASCII码中不安全字符的编码：

> 空格、< > { } | \ ^ ` ~

不安全的字符采用百分号编码，就是 % + 字符字节十六进制字符串



非ASCII码字符的编码：

先对字符进行一次编码，比如UTF-8编码，然后对编码后的字节进行百分号编码，也就是每个字节编码成 % + 字节的十六进制字符串

<span style="color:red">注意：非ASCII码字符是没有统一的标准编码方式的</span>





### HTTP协议数据包的编码方式

传输层TCP只传输字节流，HTTP数据包，包含的是文本内容，因此，接收到HTTP数据包后，应用层需要从二进制流中解析出HTTP数据包的文本内容，一旦涉及到字节和字符转换，就有编码

> HTTP 1.1 uses US-ASCII as basic character set for the request line in requests, the status line in responses (except the reason phrase) and the field names but allows any octet in the field values and the message body.

HTTP1.1协议规定，基本的字符编码是US-ASCII





参考：

https://blog.csdn.net/honghailiang888/article/details/49901993

https://stackoverflow.com/questions/818122/which-encoding-is-used-by-the-http-protocol

https://juejin.im/post/5d521575f265da03ee6a4bda





