---
layout: post
title: 使用Hadoop进行全排序
category : Hadoop
tags : [hadoop]
---
{% include JB/setup %}

`Hadoop Terasort`  
又一片碎文

## 基本思路
Hadoop框架能够保证每个reduce的输入都是按键值排序的，因此如果只有一个reduce时最终输出就是全局排序的，但是在大数据情况下只设定一个reduce无疑是不可行的。
如果能够保证reduce之间的顺序，即第i个reduce中的数据都小于第i+1个reduce中数据，则最终结果也就全局有序。  
那么就要先制定区间，各区间和reduce依次对应，用partitioner把数据依次归到对应的reduce中，hadoop中已有相印实现TotalOrderPartitioner。  
Hadoop全局排序的基本思路：  

+ 先对数据进行抽样，对抽样数据进行排序获得区间
+ Map处理后直接输出数据
+ Partitioner根据数据所属区间将数据发到对应的reduce
+ Reducer获得数据直接输出

## 抽样实现
`org.apache.hadoop.mapreduce.lib.partition.InputSampler`封装了三种抽样逻辑：SplitSampler、RandomSampler和IntervalSampler，具体介绍参见引用。代码示例：

	job.setPartitionerClass(TotalOrderPartitioner.class);
	Path pfile = new Path(input, "_partitions");
	TotalOrderPartitioner.setPartitionFile(job.getConfiguration(), pfile);
	InputSampler.Sampler<LongWritable, NullWritable> sampler = new InputSampler.RandomSampler<LongWritable, NullWritable>(
			0.1, 10000);
	InputSampler.writePartitionFile(job, sampler);


## 全排实现
代码示例：

	public class Terasort extends Configured implements Tool {
	
		/*
		 * (non-Javadoc)
		 * 
		 * @see org.apache.hadoop.util.Tool#run(java.lang.String[])
		 */
		@Override
		public int run(String[] args) throws Exception {
			Job job = new Job(getConf(), "terasort");
			job.setJarByClass(getClass());
	
			Path input = new Path(args[0]);
			FileInputFormat.setInputPaths(job, input);
			job.setInputFormatClass(KeyValueTextInputFormat.class);
			FileOutputFormat.setOutputPath(job, new Path(args[1]));
			job.setOutputFormatClass(TextOutputFormat.class);
			job.setOutputKeyClass(LongWritable.class);
			job.setOutputValueClass(NullWritable.class);
	
			job.setMapperClass(MyMapper.class);
			job.setReducerClass(MyReducer.class);
	
			// sampler
			job.setPartitionerClass(TotalOrderPartitioner.class);
			Path pfile = new Path(input, "_partitions");
			TotalOrderPartitioner.setPartitionFile(job.getConfiguration(), pfile);
			InputSampler.Sampler<LongWritable, NullWritable> sampler = new InputSampler.RandomSampler<LongWritable, NullWritable>(
					0.1, 10000);
			InputSampler.writePartitionFile(job, sampler);
			URI partitionUri = new URI(pfile.toString() + "#_partitions");
			DistributedCache.addCacheFile(partitionUri, getConf());
			DistributedCache.createSymlink(getConf());
	
			return job.waitForCompletion(true) ? 0 : 1;
		}
	
		public static class MyMapper extends Mapper<Text, Text, LongWritable, NullWritable> {
			protected void map(Text key, Text value,
					org.apache.hadoop.mapreduce.Mapper<Text, Text, LongWritable, NullWritable>.Context context)
					throws java.io.IOException, InterruptedException {
				try {
					LongWritable lw = new LongWritable(Long.valueOf(key.toString()));
					context.write(lw, NullWritable.get());
				} catch (Exception e) {
					// TODO: handle exception
				}
			};
		}
	
		public static class MyReducer extends Reducer<LongWritable, NullWritable, LongWritable, NullWritable> {
			protected void reduce(LongWritable arg0, java.lang.Iterable<NullWritable> arg1,
					org.apache.hadoop.mapreduce.Reducer<LongWritable, NullWritable, LongWritable, NullWritable>.Context arg2)
					throws java.io.IOException, InterruptedException {
				arg2.write(arg0, NullWritable.get());
			};
		}
	
		/**
		 * @param args
		 */
		public static void main(String[] args) throws Exception {
			int res = ToolRunner.run(new Configuration(), new Terasort(), args);
			System.exit(res);
		}
	
	}

## 问题记录
**File \_partition.lst does not exist**  
在google桑翻阅了各种网页，无果。后来发现setPartitionFile没有成功，最初使用`TotalOrderPartitioner.setPartitionFile(getConf(), pfile);`进行设置，
但`new Job(getConf(), "terasort")`时Job并非直接持有getConf返回的引用，而是又new了一个JobConf，坑啊！所以setPartitionFile得使用`job.getConfiguration()`作为参数。


## references
+ [Newsmth的讨论](http://www.newsmth.net/nForum/#!article/Java/301680?p=1)
+ [使用hadoop进行大规模数据的全局排序 ](http://stblog.baidu-tech.com/?p=397)
+ [Hadoop中TeraSort算法分析](http://dongxicheng.org/mapreduce/hadoop-terasort-analyse/)
+ [关于Hadoop中的采样器](http://www.cnblogs.com/xuxm2007/archive/2012/03/04/2379143.html)
+ [Mapreduce-Partition分析](http://blog.oddfoo.net/2011/04/17/mapreduce-partition%E5%88%86%E6%9E%90-2/)
