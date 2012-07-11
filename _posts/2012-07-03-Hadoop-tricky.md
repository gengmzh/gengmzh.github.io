---
layout: post
title: Hadoop Tricky
category : hadoop
tags : [hadoop]
---
{% include JB/setup %}

`Hadoop Tricky`  

## issue 1: 如何多次遍历reducer的Iterable？
测试版本是0.20，reducer的输入values类型是Iteratable，更早的版本里应该是Iterator。  
那么如何多次遍历Iteratable/Iterator，最初没想到这里有什么特殊，代码示例如下：

	@Override
	protected void reduce(MappingKey arg0, java.lang.Iterable<MappingValue> arg1,
			Reducer<MappingKey, MappingValue, NullWritable, Text>.Context context) throws java.io.IOException,
			InterruptedException {
		// user
		UserMapping user = null;
		for (Iterator<MappingValue> iter = arg1.iterator(); iter.hasNext();) {
			MappingValue input = iter.next();
			if (input.isMapping()) {
				UserMapping m = new UserMapping().parseLineStr(input.getValue());
				if (user == null || user.getTime() > m.getTime()) {
					user = m;
				}
			} else if (input.isDaily()) {
				value.set(input.getValue());
				out.write("d", NullWritable.get(), value); //这里用到了MultipleOutputs
			}
		}
		// mapping
		for (Iterator<MappingValue> iter = arg1.iterator(); iter.hasNext();) {
			MappingValue input = iter.next();
			if(input.isWeekly() || input.isMonthly()){
				if (user == null) {
					value.set(input.getValue());
				} else {
					UserBase ub = new UserBase().parseLineStr(input.getValue());
					if (!ub.isRealUser()) {
						ub.setUserId(user.getUser()).setUserType(UserBase.USERTYPE_USER);
					}
					value.set(ub.formatLineStr());
				}
				out.write((input.isWeekly() ? "w" : "m"), NullWritable.get(), value); //这里用到了MultipleOutputs
			}
		}
	}

发现除了d-之外，并无w-或m-的输出，但应该有这样的数据。始发现对values的遍历并非那么随意，参考[Iterate twice on values](http://stackoverflow.com/questions/6111248/iterate-twice-on-values)
一文，将代码修改如下：

	@Override
	protected void reduce(MappingKey arg0, java.lang.Iterable<MappingValue> arg1,
			Reducer<MappingKey, MappingValue, NullWritable, Text>.Context context) throws java.io.IOException,
			InterruptedException {
		// user
		UserMapping user = null;
		List<MappingValue> values = new ArrayList<MappingValue>();
		for (Iterator<MappingValue> iter = arg1.iterator(); iter.hasNext();) {
			MappingValue input = iter.next();
			if (input.isMapping()) {
				UserMapping m = new UserMapping().parseLineStr(input.getValue());
				if (user == null || user.getTime() > m.getTime()) {
					user = m;
				}
			} else if (input.isDaily()) {
				value.set(input.getValue());
				out.write("d", NullWritable.get(), value);
			} else {
				values.add(input.getValue());
			}
		}
		// mapping
		for (MappingValue input : values) {
			if (user == null) {
				value.set(input.getValue());
			} else {
				UserBase ub = new UserBase().parseLineStr(input.getValue());
				if (!ub.isRealUser()) {
					ub.setUserId(user.getUser()).setUserType(UserBase.USERTYPE_USER);
				}
				value.set(ub.formatLineStr());
			}
			out.write((input.isWeekly() ? "w" : "m"), NullWritable.get(), value);
		}
	}

经过多次测试发现每次输出都不一样，d-中记录数不变，w-和m-中的记录数每次都不一样，why？  
最终发现遍历values读取数据还有特殊性，那就是某些情况下会重用每次next的返回对象，也就`values.add(input.getValue());`里已经add过的MappingValue还有可能变化，这一行代码须修改如下：

	// 每次重新new一个MappingValue
	values.add(new MappingValue(input.getType(), input.getValue()));

至此对values的遍历算是完事了。

## issue 2: 如何自定义Mapper的OutputKey？
自定义MapOutputKey只要继承WritableComparable即可，最初代码示例如下：

	public class MappingKey implements WritableComparable<MappingKey> {
	
		static final String seperator = "#";
		private String siteId, userId;
	
		public MappingKey() {
			this(null, null);
		}
	
		public MappingKey(String siteId, String userId) {
			super();
			this.siteId = siteId;
			this.userId = userId;
		}
	
		@Override
		public void readFields(DataInput arg0) throws IOException {
			String text = Text.readString(arg0);
			String[] args = text.split(seperator, 2);
			siteId = args[0];
			userId = args[1];
		}
	
		@Override
		public void write(DataOutput arg0) throws IOException {
			Text.writeString(arg0, siteId + seperator + userId);
		}
	
		@Override
		public int compareTo(MappingKey other) {
			if (this == other) {
				return 0;
			}
			int res = compare(this.siteId, other.siteId);
			if (res == 0) {
				res = compare(this.userId, other.userId);
			}
			return res;
		}
		
		int compare(String l, String r) {
			if (l == null) {
				return r == null ? 0 : -1;
			}
			return r != null ? l.compareTo(r) : 1;
		}
	
		public String getSiteId() {
			return siteId;
		}
	
		public String getUserId() {
			return userId;
		}
	
	}

测试几次后发现最终输出也是每次都不一样，修改MappingKey如下：

	public class MappingKey implements WritableComparable<MappingKey> {
	
		static final String seperator = "#";
		private String siteId, userId;
	
		public MappingKey() {
			this(null, null);
		}
	
		public MappingKey(String siteId, String userId) {
			super();
			this.siteId = siteId;
			this.userId = userId;
		}
	
		@Override
		public void readFields(DataInput arg0) throws IOException {
			String text = Text.readString(arg0);
	//		if (text != null && !text.isEmpty()) {
				String[] args = text.split(seperator, 2);
				siteId = /*(args.length > 0 ? */args[0]/* : null)*/;
				userId = /*(args.length > 1 ? */args[1] /*: null)*/;
	//		}
		}
	
		@Override
		public void write(DataOutput arg0) throws IOException {
			Text.writeString(arg0, siteId + seperator + userId);
		}
	
		@Override
		public int compareTo(MappingKey other) {
	//		if (this == other) {
	//			return 0;
	//		}
			return (siteId + seperator + userId).compareTo( other.siteId+seperator+other.userId);
	//		int res = compare(this.siteId, other.siteId);
	//		if (res == 0) {
	//			res = compare(this.userId, other.userId);
	//		}
	//		return res;
		}
		
		@Override
		public boolean equals(Object obj) { 
			if(!(obj instanceof MappingKey)){
				return false;
			}
			MappingKey other=(MappingKey)obj; 
			return (siteId + seperator + userId).equals( other.siteId+seperator+other.userId);
		}
		
		@Override
		public int hashCode() { 
			int res=17;
			res=res*37+siteId.hashCode();
			res=res*37+userId.hashCode();
			return res;
		}
	
	//	int compare(String l, String r) {
	//		if (l == null) {
	//			return r == null ? 0 : -1;
	//		}
	//		return r != null ? l.compareTo(r) : 1;
	//	}
	
		public String getSiteId() {
			return siteId;
		}
	
		public String getUserId() {
			return userId;
		}
	
	}

主要修改了compareTo，添加了equals和hashCode，之后运行正常，具体是哪个修改起作用了待有时间再研究。



## references
+ [Iterate twice on values](http://stackoverflow.com/questions/6111248/iterate-twice-on-values)
