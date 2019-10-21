---
title: JW-FastDFS图片服务器
date: 2019.04.11 14:49:27
tags: JW
categories: JW
---


FastDFS是C语言开发的分布式文件系统。提供冗余备份，负载均衡，线性扩容等机制，并注重高可用高性能指标。使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务。

![FastDFS](/images/JW-FastDFS图片服务器/FastDFS.jpg)
Client：客户端
Tracker：管理，记录
Storage：同组不同成员存储相同信息，不同组存储不同信息

文件上传：
![FastDFS1](/images/JW-FastDFS图片服务器/FastDFS1.jpg)

文件下载：
![FastDFS2](/images/JW-FastDFS图片服务器/FastDFS2.jpg)
索引信息示例：group1/M00/02/44/gr4jot534yt43r3543

**客户端**
下载FastDFS-Java客户端作为工程引用
pom.xml
```
		<dependency>
			<groupId>org.csource</groupId>
			<artifactId>fastdfs-client-java</artifactId>
			<version>1.27-SNAPSHOT</version>
		</dependency>
```

```
配置文件, a.conf：
tracker_server=192.168.25.133:22122

ClientGlobal.init("绝对路径/a.conf");
TrackClient trackClient = new TrackClient();
TrackerServer trackerServer = trackClient.getConnection();
StorageServer StorageServer = null;
StorageClient storageClient = new StorageClient(trackerServer, StorageServer);
//使用storageClient上传文件
String[] result = storageClient.upload_file(...);
// result返回数据库存储路径
```

**FastDFSClient工具类方便使用**
```
//  FastDFSClient
public class FastDFSClient {

	private TrackerClient trackerClient = null;
	private TrackerServer trackerServer = null;
	private StorageServer storageServer = null;
	private StorageClient1 storageClient = null;

	public FastDFSClient(String conf) throws Exception {
		if (conf.contains("classpath:")) {
			conf = conf.replace("classpath:", this.getClass().getResource("/").getPath());
		}
		ClientGlobal.init(conf);
		trackerClient = new TrackerClient();
		trackerServer = trackerClient.getConnection();
		storageServer = null;
		storageClient = new StorageClient1(trackerServer, storageServer);
	}

	/**
	 * 上传文件方法
	 * <p>Title: uploadFile</p>
	 * <p>Description: </p>
	 * @param fileName 文件全路径
	 * @param extName 文件扩展名，不包含（.）
	 * @param metas 文件扩展信息
	 * @return
	 * @throws Exception
	 */
	public String uploadFile(String fileName, String extName, NameValuePair[] metas) throws Exception {
		String result = storageClient.upload_file1(fileName, extName, metas);
		return result;
	}

	public String uploadFile(String fileName) throws Exception {
		return uploadFile(fileName, null, null);
	}

	public String uploadFile(String fileName, String extName) throws Exception {
		return uploadFile(fileName, extName, null);
	}

	/**
	 * 上传文件方法
	 * <p>Title: uploadFile</p>
	 * <p>Description: </p>
	 * @param fileContent 文件的内容，字节数组
	 * @param extName 文件扩展名
	 * @param metas 文件扩展信息
	 * @return
	 * @throws Exception
	 */
	public String uploadFile(byte[] fileContent, String extName, NameValuePair[] metas) throws Exception {

		String result = storageClient.upload_file1(fileContent, extName, metas);
		return result;
	}

	public String uploadFile(byte[] fileContent) throws Exception {
		return uploadFile(fileContent, null, null);
	}

	public String uploadFile(byte[] fileContent, String extName) throws Exception {
		return uploadFile(fileContent, extName, null);
	}
}
```

可配合nginx实现文件服务器..

前端可使用KingEdit富文本编辑器插件，富文本编辑器是纯js插件
![FastDFS3](/images/JW-FastDFS图片服务器/FastDFS3.jpg)

common.js中KingEdit文件上传所需参数
![FastDFS4](/images/JW-FastDFS图片服务器/FastDFS4.jpg)

pom添加文件上传组件依赖
![FastDFS5](/images/JW-FastDFS图片服务器/FastDFS5.jpg)

springmvc配置文件上传解析器
![FastDFS6](/images/JW-FastDFS图片服务器/FastDFS6.jpg)

controller配置，使用FastDFS上传
![FastDFS7](/images/JW-FastDFS图片服务器/FastDFS7.jpg)


兼容性问题
使用map加@requestbody作为浏览器返回值，content type为 application/json，部分浏览器KingEditor受影响。
改为使用json字符串作为浏览器返回值，content type则为text/plain。


e.g. 本文仅供个人笔记使用，借鉴部分网上资料。
