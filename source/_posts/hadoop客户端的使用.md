---
title: hadoop客户端的使用
date: 2020-03-13 17:41:57
categories: [hadoop,大数据]
tags: [大数据,工具]
---

​	使用java代码，将本地文件传递到hdfs中。需要自己放入依赖的jar包，这是一些实例代码，作为参考使用

<!--more-->

``` java
package com.king.bigdata;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import java.net.URI;
public class HdfsClientDemo {
    public static void main(String[] args) throws Exception {
        /**
         * Configuration参数对象的机制：
         *    构造时，会加载jar包中的默认配置 xx-default.xml
         *    再加载 用户配置xx-site.xml  ，覆盖掉默认参数
         *    构造完成之后，还可以conf.set("p","v")，会再次覆盖用户配置文件中的参数值
         */
        // new Configuration()会从项目的classpath中加载core-default.xml hdfs-default.xml core-site.xml hdfs-site.xml等文件
        Configuration conf = new Configuration();
        // 指定本客户端上传文件到hdfs时需要保存的副本数为：2
        conf.set("dfs.replication", "2");
        // 指定本客户端上传文件到hdfs时切块的规格大小：64M
        conf.set("dfs.blocksize", "64m");
        // 构造一个访问指定HDFS系统的客户端对象: 参数1:——HDFS系统的URI，参数2：——客户端要特别指定的参数，参数3：客户端的身份（用户名）
        FileSystem fs = FileSystem.get(new URI("hdfs://hdf-01:9000/"), conf, "root");
        // 上传一个文件到HDFS中
        fs.copyFromLocalFile(new Path("D:/jdk-8u241-linux-x64.tar.gz"), new Path("/aaa/"));
        fs.close();
    }
    
    FileSystem fs = null;
	@Before
	public void init() throws Exception{
		Configuration conf = new Configuration();
		conf.set("dfs.replication", "2");
		conf.set("dfs.blocksize", "64m");
		fs = FileSystem.get(new URI("hdfs://hdf-01:9000/"), conf, "root");
	}
	
	
	/**
	 * 从HDFS中下载文件到客户端本地磁盘
	 * @throws IOException 
	 * @throws IllegalArgumentException 
	 */
	@Test
	public void testGet() throws IllegalArgumentException, IOException{
		
		fs.copyToLocalFile(new Path("/hdp20-05.txt"), new Path("f:/"));
		fs.close();
		
	}
	
	
	/**
	 * 在hdfs内部移动文件\修改名称
	 */
	@Test
	public void testRename() throws Exception{
		fs.rename(new Path("/install.log"), new Path("/aaa/in.log"));
		fs.close();
		
	}
	
	/**
	 * 在hdfs中创建文件夹
	 */
	@Test
	public void testMkdir() throws Exception{
		fs.mkdirs(new Path("/xx/yy/zz")); // 如果使用命令行添加多个文件夹
		fs.close();
	}
	
	/**
	 * 在hdfs中删除文件或文件夹
	 */
	@Test
	public void testRm() throws Exception{
		fs.delete(new Path("/aaa"), true);
		fs.close();
	}
	
	/**
	 * 查询hdfs指定目录下的文件信息
	 */
	@Test
	public void testLs() throws Exception{
		// 只查询文件的信息,不返回文件夹的信息
		RemoteIterator<LocatedFileStatus> iter = fs.listFiles(new Path("/"), true);
		while(iter.hasNext()){
			LocatedFileStatus status = iter.next();
			System.out.println("文件全路径："+status.getPath());
			System.out.println("块大小："+status.getBlockSize());
			System.out.println("文件长度："+status.getLen());
			System.out.println("副本数量："+status.getReplication());
			System.out.println("块信息："+Arrays.toString(status.getBlockLocations()));
			System.out.println("--------------------------------");
		}
		fs.close();
	}
	
	/**
	 * 查询hdfs指定目录下的文件和文件夹信息(无递归)
	 */
	@Test
	public void testLs2() throws Exception{
		FileStatus[] listStatus = fs.listStatus(new Path("/"));
		for(FileStatus status:listStatus){
			System.out.println("文件全路径："+status.getPath());
			System.out.println(status.isDirectory()?"这是文件夹":"这是文件");
			System.out.println("块大小："+status.getBlockSize());
			System.out.println("文件长度："+status.getLen());
			System.out.println("副本数量："+status.getReplication());
			System.out.println("--------------------------------");
		}
		fs.close();
	}
    
    /**
	 * 读取hdfs中的文件的内容
	 * 
	 * @throws IOException
	 * @throws IllegalArgumentException
	 */
	@Test
	public void testReadData() throws IllegalArgumentException, IOException {

		FSDataInputStream in = fs.open(new Path("/test.txt"));

		BufferedReader br = new BufferedReader(new InputStreamReader(in, "utf-8"));

		String line = null;
		while ((line = br.readLine()) != null) {
			System.out.println(line);
		}

		br.close();
		in.close();
		fs.close();

	}

	/**
	 * 读取hdfs中文件的指定偏移量范围的内容
	 * 
	 * 
	 * 作业题：用本例中的知识，实现读取一个文本文件中的指定BLOCK块中的所有数据
	 * 
	 * @throws IOException
	 * @throws IllegalArgumentException
	 */
	@Test
	public void testRandomReadData() throws IllegalArgumentException, IOException {
		FSDataInputStream in = fs.open(new Path("/xx.dat"));
		// 将读取的起始位置进行指定
		in.seek(12);
		// 读16个字节
		byte[] buf = new byte[16];
		in.read(buf);
		System.out.println(new String(buf));
		in.close();
		fs.close();
	}

	/**
	 * 往hdfs中的文件写内容
	 * 
	 * @throws IOException
	 * @throws IllegalArgumentException
	 */

	@Test
	public void testWriteData() throws IllegalArgumentException, IOException {

		FSDataOutputStream out = fs.create(new Path("/zz.jpg"), false);
		// D:\images\006l0mbogy1fhehjb6ikoj30ku0ku76b.jpg
		FileInputStream in = new FileInputStream("D:/images/006l0mbogy1fhehjb6ikoj30ku0ku76b.jpg");
		byte[] buf = new byte[1024];
		int read = 0;
		while ((read = in.read(buf)) != -1) {
			out.write(buf,0,read);
		}
		in.close();
		out.close();
		fs.close();
	}
}
```
