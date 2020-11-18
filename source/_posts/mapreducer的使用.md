---
title: mapreducer的使用
date: 2020-08-03 22:16:32
categories: [大数据]
tags: [大数据,linux]
---

## 1.mapruduce的理念

​	MapReduce是hadoop的计算框架。表现形式就是有个输入(input)和输出(output)。在运行一个mapreduce计算任务的时候，任务过程被分为两个阶段：map阶段和reduce阶段，每个阶段都是用键值对(key/value)作为输入(input)和输出(output)。

## 2.demo

​	概念什么的东西说的再多，不如两个demo来的实在。

<!--more-->

### 1.计算网站访问的Top 5

​	需要计算的数据样式

``` xml
2017/07/28 qq.com/a
2017/07/28 qq.com/bx
2017/07/28 qq.com/by
2017/07/28 qq.com/by3
2017/07/28 qq.com/news
2017/07/28 sina.com/news/socail
2017/07/28 163.com/ac
2017/07/28 sina.com/news/socail
2017/07/28 163.com/sport
2017/07/28 163.com/ac
2017/07/28 sina.com/play
2017/07/28 163.com/sport
2017/07/28 163.com/ac
2017/07/28 sina.com/movie
2017/07/28 sina.com/play
2017/07/28 sina.com/movie
2017/07/28 163.com/sport
2017/07/28 sina.com/movie
.......

```

mapreduce模式下对数据进行统计：

``` java
// map
import java.io.IOException;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class PageTopnMapper extends Mapper<LongWritable, Text, Text, IntWritable>{
	@Override
	protected void map(LongWritable key, Text value, Mapper<LongWritable, Text, Text, IntWritable>.Context context)
			throws IOException, InterruptedException {
		String line = value.toString();
		String[] split = line.split(" ");
		context.write(new Text(split[1]), new IntWritable(1));
	}
}
```

``` java
# reducer
import java.io.IOException;
import java.util.Map.Entry;
import java.util.Set;
import java.util.TreeMap;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

public class PageTopnReducer extends Reducer<Text, IntWritable, Text, IntWritable>{
	TreeMap<PageCount, Object> treeMap = new TreeMap<>();
	@Override
	protected void reduce(Text key, Iterable<IntWritable> values,
			Reducer<Text, IntWritable, Text, IntWritable>.Context context) throws IOException, InterruptedException {
		int count = 0;
		for (IntWritable value : values) {
			count += value.get();
		}
		PageCount pageCount = new PageCount();
		pageCount.set(key.toString(), count);
		treeMap.put(pageCount,null);
	}

	@Override
	protected void cleanup(Context context)
			throws IOException, InterruptedException {
		Configuration conf = context.getConfiguration();
		int topn = conf.getInt("top.n", 5);
		Set<Entry<PageCount, Object>> entrySet = treeMap.entrySet();
		int i= 0;
		for (Entry<PageCount, Object> entry : entrySet) {
			context.write(new Text(entry.getKey().getPage()), new IntWritable(entry.getKey().getCount()));
			i++;
			if(i==topn) return;
		}
	}
}

```

为了统计出top5使用了一个pagecount的类型，并且实现了Comparable的比较接口，为后续放入到treemap中方便排序做了准备

``` java
package cn.edu360.mr.page.topn;

public class PageCount implements Comparable<PageCount>{
	
	private String page;
	private int count;
	
	public void set(String page, int count) {
		this.page = page;
		this.count = count;
	}
	
	public String getPage() {
		return page;
	}
	public void setPage(String page) {
		this.page = page;
	}
	public int getCount() {
		return count;
	}
	public void setCount(int count) {
		this.count = count;
	}

	@Override
	public int compareTo(PageCount o) {	
         // 此处数值相等了，会按照字母的顺序进行排序
		return o.getCount()-this.count==0?this.page.compareTo(o.getPage()):o.getCount()-this.count;
	}
}

```

最后进行提交，因为这是demo所以只是在windows上进行单机的提交

``` java
package cn.edu360.mr.page.topn;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class JobSubmitter {

	public static void main(String[] args) throws Exception {

		/**
		 * 通过加载classpath下的*-site.xml文件解析参数
		 */
		Configuration conf = new Configuration();
		conf.addResource("xx-oo.xml");
		/**
		 * 通过代码设置参数
		 */
		//conf.setInt("top.n", 3);
		//conf.setInt("top.n", Integer.parseInt(args[0]));
		/**
		 * 通过属性配置文件获取参数
		 */
		/*Properties props = new Properties();
		props.load(JobSubmitter.class.getClassLoader().getResourceAsStream("topn.properties"));
		conf.setInt("top.n", Integer.parseInt(props.getProperty("top.n")));*/
		Job job = Job.getInstance(conf);
		job.setJarByClass(JobSubmitter.class);
		job.setMapperClass(PageTopnMapper.class);
		job.setReducerClass(PageTopnReducer.class);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(IntWritable.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(IntWritable.class);
		FileInputFormat.setInputPaths(job, new Path("F:\\mrdata\\url\\input"));
		FileOutputFormat.setOutputPath(job, new Path("F:\\mrdata\\url\\output")); // 必须不存在
		job.waitForCompletion(true);
	}
}

```

### 2.自定义Partitioner

​	对电话归属地进行分块，不同的区号，生成不同的块

数据样式

``` xml
1363157985066 	13726230503	00-FD-07-A4-72-B8:CMCC	120.196.100.82	i02.c.aliimg.com		24	27	2481	24681	200
1363157995052 	13826544101	5C-0E-8B-C7-F1-E0:CMCC	120.197.40.4			4	0	264	0	200
1363157991076 	13926435656	20-10-7A-28-CC-0A:CMCC	120.196.100.99			2	4	132	1512	200
1363154400022 	13926251106	5C-0E-8B-8B-B1-50:CMCC	120.197.40.4			4	0	240	0	200
1363157993044 	18211575961	94-71-AC-CD-E6-18:CMCC-EASY	120.196.100.99	iface.qiyi.com	视频网站	15	12	1527	2106	200
1363157995074 	84138413	5C-0E-8B-8C-E8-20:7DaysInn	120.197.40.4	122.72.52.12		20	16	4116	1432	200
1363157993055 	13560439658	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			18	15	1116	954	200
1363157995033 	15920133257	5C-0E-8B-C7-BA-20:CMCC	120.197.40.4	sug.so.360.cn	信息安全	20	20	3156	2936	200
1363157983019 	13719199419	68-A1-B7-03-07-B1:CMCC-EASY	120.196.100.82			4	0	240	0	200
1363157984041 	13660577991	5C-0E-8B-92-5C-20:CMCC-EASY	120.197.40.4	s19.cnzz.com	站点统计	24	9	6960	690	200
1363157973098 	15013685858	5C-0E-8B-C7-F7-90:CMCC	120.197.40.4	rank.ie.sogou.com	搜索引擎	28	27	3659	3538	200
1363157986029 	15989002119	E8-99-C4-4E-93-E0:CMCC-EASY	120.196.100.99	www.umeng.com	站点统计	3	3	1938	180	200
1363157992093 	13560439658	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			15	9	918	4938	200
1363157986041 	13480253104	5C-0E-8B-C7-FC-80:CMCC-EASY	120.197.40.4			3	3	180	180	200
1363157984040 	13602846565	5C-0E-8B-8B-B6-00:CMCC	120.197.40.4	2052.flash2-http.qq.com	综合门户	15	12	1938	2910	200
1363157995093 	13922314466	00-FD-07-A2-EC-BA:CMCC	120.196.100.82	img.qfc.cn		12	12	3008	3720	200
1363157982040 	13502468823	5C-0A-5B-6A-0B-D4:CMCC-EASY	120.196.100.99	y0.ifengimg.com	综合门户	57	102	7335	110349	200
1363157986072 	18320173382	84-25-DB-4F-10-1A:CMCC-EASY	120.196.100.99	input.shouji.sogou.com	搜索引擎	21	18	9531	2412	200
1363157990043 	13925057413	00-1F-64-E1-E6-9A:CMCC	120.196.100.55	t3.baidu.com	搜索引擎	69	63	11058	48243	200
1363157988072 	13760778710	00-FD-07-A4-7B-08:CMCC	120.196.100.82			2	2	120	120	200
1363157985066 	13726238888	00-FD-07-A4-72-B8:CMCC	120.196.100.82	i02.c.aliimg.com		24	27	2481	24681	200
1363157993055 	13560436666	C4-17-FE-BA-DE-D9:CMCC	120.196.100.99			18	15	1116	954	200
```

为了方便进行统计，设计的统计类：

``` java
import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.Writable;

/**
 * 本案例的功能：演示自定义数据类型如何实现hadoop的序列化接口
 * 1、该类一定要保留空参构造函数
 * 2、write方法中输出字段二进制数据的顺序  要与  readFields方法读取数据的顺序一致
 * 
 * @author ThinkPad
 *
 */
public class FlowBean implements Writable { // hadoop的序列化接口

	private int upFlow;
	private int dFlow;
	private String phone;
	private int amountFlow;

	public FlowBean(){}
	
	public FlowBean(String phone, int upFlow, int dFlow) {
		this.phone = phone;
		this.upFlow = upFlow;
		this.dFlow = dFlow;
		this.amountFlow = upFlow + dFlow;
	}

	public String getPhone() {
		return phone;
	}

	public void setPhone(String phone) {
		this.phone = phone;
	}

	public int getUpFlow() {
		return upFlow;
	}

	public void setUpFlow(int upFlow) {
		this.upFlow = upFlow;
	}

	public int getdFlow() {
		return dFlow;
	}

	public void setdFlow(int dFlow) {
		this.dFlow = dFlow;
	}

	public int getAmountFlow() {
		return amountFlow;
	}

	public void setAmountFlow(int amountFlow) {
		this.amountFlow = amountFlow;
	}

	/**
	 * hadoop系统在序列化该类的对象时要调用的方法
	 */
	@Override
	public void write(DataOutput out) throws IOException {

		out.writeInt(upFlow);
		out.writeUTF(phone);
		out.writeInt(dFlow);
		out.writeInt(amountFlow);

	}

	/**
	 * hadoop系统在反序列化该类的对象时要调用的方法
	 */
	@Override
	public void readFields(DataInput in) throws IOException {
		this.upFlow = in.readInt();
		this.phone = in.readUTF();
		this.dFlow = in.readInt();
		this.amountFlow = in.readInt();
	}

	@Override
	public String toString() {
		 
		return this.phone + ","+this.upFlow +","+ this.dFlow +"," + this.amountFlow;
	}
	
}

```

map

``` java
import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class FlowCountMapper extends Mapper<LongWritable, Text, Text, FlowBean>{
	@Override
	protected void map(LongWritable key, Text value, Context context)
			throws IOException, InterruptedException {
		String line = value.toString();
		String[] fields = line.split("\t");
		String phone = fields[1];
		int upFlow = Integer.parseInt(fields[fields.length-3]);
		int dFlow = Integer.parseInt(fields[fields.length-2]);
		context.write(new Text(phone), new FlowBean(phone, upFlow, dFlow));
	}
}
```

reduce

``` java
import java.io.IOException;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
public class FlowCountReducer extends Reducer<Text, FlowBean, Text, FlowBean>{
	/**
	 *  key：是某个手机号
	 *  values：是这个手机号所产生的所有访问记录中的流量数据
	 *  
	 *  <135,flowBean1><135,flowBean2><135,flowBean3><135,flowBean4>
	 */
	@Override
	protected void reduce(Text key, Iterable<FlowBean> values, Reducer<Text, FlowBean, Text, FlowBean>.Context context)
			throws IOException, InterruptedException {
		int upSum = 0;
		int dSum = 0;
		for(FlowBean value:values){
			upSum += value.getUpFlow();
			dSum += value.getdFlow();
		}
		context.write(key, new FlowBean(key.toString(), upSum, dSum));
	}
}
```

Partitioner的实现，根据区号进行划分不同的输出

``` java
import java.util.HashMap;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Partitioner;

/**
 * 本类是提供给MapTask用的
 * MapTask通过这个类的getPartition方法，来计算它所产生的每一对kv数据该分发给哪一个reduce task
 * @author ThinkPad
 *
 */
public class ProvincePartitioner extends Partitioner<Text, FlowBean>{
	static HashMap<String,Integer> codeMap = new HashMap<>();
	static{
		codeMap.put("135", 0);
		codeMap.put("136", 1);
		codeMap.put("137", 2);
		codeMap.put("138", 3);
		codeMap.put("139", 4);
	}
	@Override
	public int getPartition(Text key, FlowBean value, int numPartitions) {
		
		Integer code = codeMap.get(key.toString().substring(0, 3));
		return code==null?5:code;
	}

}

```

submit

``` java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
public class JobSubmitter {
	public static void main(String[] args) throws Exception {
		Configuration conf = new Configuration();
		Job job = Job.getInstance(conf);
		job.setJarByClass(JobSubmitter.class);
		job.setMapperClass(FlowCountMapper.class);
		job.setReducerClass(FlowCountReducer.class);
		// 设置参数：maptask在做数据分区时，用哪个分区逻辑类  （如果不指定，它会用默认的HashPartitioner）
		job.setPartitionerClass(ProvincePartitioner.class);
		// 由于我们的ProvincePartitioner可能会产生6种分区号，所以，需要有6个reduce task来接收
		job.setNumReduceTasks(6);
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(FlowBean.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(FlowBean.class);
		FileInputFormat.setInputPaths(job, new Path("F:\\mrdata\\flow\\input"));
		FileOutputFormat.setOutputPath(job, new Path("F:\\mrdata\\flow\\province-output"));
		job.waitForCompletion(true);
	}
}
```

最后的输出结果：

![](jg.PNG)

有6个文件对应不同的结果，打开悄悄

![](jg1.PNG)





