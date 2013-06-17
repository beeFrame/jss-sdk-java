#京东云存储服务Java SDK
============
  [京东云存储服务](http://www.jcloud.com/)(Jingdong Storage Service)是京东商城对外提供海量、安全、低成本、高可用的
  云存储服务基础平台。开发者利用该工具包可以实现：
  1. 管理Bucket信息
  2. 上传与下载Object数据
  3. 生成可公开访问/私有访问的URL
  
##平台优势
  1.  <b>海量数据</b>，通过数据冗余、集中资源管理等方式将大规模的硬件整合为高可靠的海量虚拟存储资源，用户可以轻松享受多媒体分享网站、网盘、个人企业数据备份等海量数据的云存储服务。
  2.  <b>低成本</b>，无需提供服务器，也不用关心冗余系统，不必关心购买安装和维护用于数据资源的硬件，用户完全可以省下这笔成本，借助于JSS平台，就可以轻松地创建和管理数据资源。
  3.  <b>安全性</b>，平台采用数据隔离、访问控制策略来保证数据安全性，使用严格的安全措施，比如使用经过证明的加密算法对用户进行身份验证，有效防止用户信息和用户数据资源泄露。
  4.  <b>高可用性</b>，通过软件智能调度实现自动故障恢复来保证系统的高可用性，同时采用群集系统，快速消除单点故障,在任何时候都能够保证系统正常使用，对外提供云存储服务。

##Maven

```xml
<dependency>
	<groupId>com.jcloud</groupId>
	<artifactId>jss-sdk-java</artifactId>
	<version>1.0.1-SNAPSHOT</version>
</dependency>
```
##使用  
  京东云存储服务Java SDK核心类为JingdongStorageService，开发者需通过该类提供的多种方法访问京东云存储服务。
###构建JingdongStorageService对象
  在使用Credential之前，需要创建Credential对象，该对象必须包含用户的AccessKeyId以及SecertAccessKeyId
```java
Credential credential = new Credential("AccessKeyId", "SecertAccessKeyId");
```
  通过Credential对象来创建JingdongStorageService
```java
JingdongStorageService service = new JingdongStorageService(credential);
```
JingdongStorageService对象内部维护一组HTTP连接池，在不使用该对象之前需要调用其shutdown方法关闭连接池，请注意，一旦调用shutdown方法，该对象就不能再次被使用，否则将会抛出异常。
```java
service.shutdown();
```
###列出所有的Bucket
```java
List<Bucket> list = service.listBucket();
for(Bucket b:list) {
   System.out.println(b.getName());
}
```
###Bucket相关操作
创建一个Bucket，请注意，京东云存储所有的Bucket都是全局唯一的，每个用户最多只能创建100个Bucket，每个Bucket在京东云存储系统中都是唯一的，不能创建2个相同名字的Bucket。
```java
service.bucket("bucketname").create();
```
删除Bucket,当Bucket中没有Object的时候，该Bucket才能被删除, 否则删除会失败。
```java
service.bucket("bucketname").delete();
```
###Object相关操作
上传数据
```java
service.bucket("bucketname").object("key").entity(new File("filename")).put();
```

下载数据
```java
service.bucket("bucketname").object("key").get().toFile(new File("/export/test.txt"));
```
断点下载数据(Range GET)
```java
service.bucket(bucketName).object(key).range(0, 3).get().toFile(new File("/export/test.key"));
service.bucket(bucketName).object(key).range(4).get().toFile(new File("/export/test.key"),true);
```
上传流对象
```java
File fp = new File("D:\\export\\test.txt");
InputStream inStream = new FileInputStream(fp);
service.bucket("bucketName").object("key").entity(fp.length(), inStream).put();//必须指定流的长度
inStream.close();
```
获取Object信息与Metadata(HEAD Object)
```java
StorageObject so = service.bucket("bucketname").object("key").head();
String bucketName = so.getBucket(); // 获取bucketName
long contentLength = so.getContentLength(); // 获取该key的大小
String contentoType = so.getContentType(); // 获取流类型
String md5 = so.getETag(); // 获取该key的MD5
String lastModified = so.getLastModified(); // 获取该key的最后修改时间  
Map<String, String> headers = so.getHeaders(); // 获取服务端返回的headers
```
获取 Object流对象
```java
InputStream in=service.bucket("bucketname").object("key").get().getInputStream();//使用完成后，流需要手动关闭
```
删除 Object
```java
service.bucket("bucketname").object("key").delete();
```
判断 Object 是否在 Bucket中
```java
boolean exist = service.bucket("bucketname").object("key").exist();
```
获取 Bucket 下 Object 列表(默认返回前1000个,按照key做字典排序)
```java
ObjectListing oResult = service.bucket("bucketname").listObject();
for (ObjectSummary okey : oResult.getObjectSummaries()) {
    System.out.println("keyName:" + okey.getKey());
}
```
获取 Bucket 下 Object 列表(返回前n个,按照key做字典排序)
```java
ObjectListing oResult = service.bucket("bucketname").maxKeys(n).listObject();
for (ObjectSummary okey : oResult.getObjectSummaries()) {
    System.out.println("keyName:" + okey.getKey());
}
```
获取 Bucket 下 Object 列表(判断是否还有下一页,按照key做字典排序)
```java
ObjectListing oResult = service.bucket("bucketname").listObject();
boolean hasNext = oResult.hasNext();
```
获取 Bucket 下 Object 列表(返回bucket下第587~20000的key,按照key做字典排序)
```java
ObjectListing oResult = service.bucket("bucketname").maxKeys(586).listObject();
List<ObjectSummary> list = oResult.getObjectSummaries();
if (oResult.hasNext()) {
  String marker = list.get(list.size()).getKey();
  List<ObjectSummary> listResult = service.bucket("bucketname") .marker(marker).maxKeys(2000 - 586).listObject().getObjectSummaries();
  //遍历listResult即可得到第587~2000的key信息
  }
```
获取 Bucket 下 Object 列表(返回该bucket下以“aaprefix”开头的key,按照key做字典排序)
```java
ObjectListing objects = jss.bucket("bucketname").prefix("appprefix").listObject();
for (ObjectSummary okey : objects.getObjectSummaries()) {
    System.out.println("keyName:" + okey.getKey());
}
```
创建带预签名的URI
京东云存储提供了一种基于查询字串(Query String)的认证方式，即通过预签名(Presigned)的方式，为要发布的Object生成一个带有认证信息的URI，并将它分发给第三方用户来实现公开访问。
SDK中提供了PresigendURIBuilder来构造预签名URI。
```java
URI signatureUrl = jss.bucket("bucketname").object("key").generatePresignedUrl(500000);
  // 产生一个链接,可以通过浏览器来下载该key，500000秒之后该链接就不能再次下载了
```
生成的URI如下：
```java
http://storage.jcloud.com/bucketname/key?Expires=1371947369&AccessKey=dfa51215af4a47c086cbf77d1479c07d&Signature=F4vmVeqveYJwqCpuR8NZO6%2FIU7s%3D
```
## Exception
京东云存储的Exception包含StoragerClientException和StorageServerException
###StoragerClientException
在访问云存储过程中，所有没有能够正常完成服务请求的操作，都会返回StoragerClientException该 Exception 是由 RuntimeException 派生而来，StoragerClientException的对象中，
并会得出以下由存储服务器获得到的错误返回的响应：error code， error message， error request id， error resource。例如在创建2次Bucket时候，我们执行的代码如下
```java
try {
	jss.bucket("bucketname").create();
	jss.bucket("bucketname").create();
    } catch (StorageServerException e) {
	e.printStackTrace();
    } catch (StorageClientException e) {
    System.out.println("httpcode:"+e.getError().getHttpStatusCode());//获取http返回错误码
	System.out.println("errormessage:"+e.getError().getMessage());//获取错误信息
	System.out.println("resource:"+e.getError().getResource());//获取请求资源
	System.out.println("requestId:"+e.getError().getRequestId());//获取请求ID	
    }

```
###StorageServerException
在访问云存储过程中，所有没有能够正常完成服务请求的操作，都会返回StoragerServerException该 Exception 是由 RuntimeException 派生而来，
StoragerServerException所捕获的异常信息是由于京东云存储服务异常而导致的，为了不影响你的服务，建议立即联系京东云存储服务。

