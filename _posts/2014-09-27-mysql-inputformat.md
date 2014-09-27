---
layout: post
title: mysql inputformat简单实现
category: Hadoop
date: 2014-09-27
---
	import java.io.IOException;
	import java.sql.Connection;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.sql.Statement;
	import java.util.ArrayList;
	import java.util.List;
	
	import org.apache.hadoop.io.LongWritable;
	import org.apache.hadoop.mapreduce.InputFormat;
	import org.apache.hadoop.mapreduce.InputSplit;
	import org.apache.hadoop.mapreduce.JobContext;
	import org.apache.hadoop.mapreduce.RecordReader;
	import org.apache.hadoop.mapreduce.TaskAttemptContext;
	
	public class MySqlInputFormat extends InputFormat<LongWritable, Person> {
	
	@Override
	public List<InputSplit> getSplits(JobContext context) throws IOException,
	InterruptedException {
	String connString = "jdbc:mysql://172.20.29.29:3306/MapReduceInputTest";
	String sql = "select count(*) as num from person";
	try {
	Class.forName("com.mysql.jdbc.Driver");
	} catch (ClassNotFoundException e) {
	e.printStackTrace();
	}
	
	Connection conn = null;
	Statement stmt = null;
	ResultSet rs = null;
	try {
	conn = java.sql.DriverManager.getConnection(connString
	+ "?useUnicode=true&characterEncoding=utf8", "admin",
	"admin");
	stmt = conn.createStatement();
	rs = stmt.executeQuery(sql);
	long count = 0;
	if (rs.next()) {
	count = rs.getLong("num");
	}
	System.out.println(count);
	List<InputSplit> splits = new ArrayList<InputSplit>();
	
	long mapNum = count % 3 == 0 ? (count) / 3 : (count / 3 + 1);
	
	for (long i = 0; i < mapNum; i++) {
	long start = i * 3;
	long end = (i + 1) * 3;
	MysqlInputSplit mysqlInputSplit = new MysqlInputSplit(start,
	end);
	splits.add(mysqlInputSplit);
	}
	return splits;
	} catch (Exception ex) {
	ex.printStackTrace();
	try {
	rs.close();
	stmt.close();
	conn.close();
	} catch (SQLException e) {
	e.printStackTrace();
	}
	}
	
	return null;
	}
	
	@Override
	public RecordReader<LongWritable, Person> createRecordReader(
	InputSplit split, TaskAttemptContext context) throws IOException,
	InterruptedException {
	return new MySqlRecordReader((MysqlInputSplit) split);
	}
	
	}
	
=====================================================================================================
	
	import java.io.DataInput;
	import java.io.DataOutput;
	import java.io.IOException;
	
	import org.apache.hadoop.io.Writable;
	import org.apache.hadoop.mapreduce.InputSplit;
	
	public class MysqlInputSplit extends InputSplit implements Writable {
	private long end = 0;
	private long start = 0;
	
	public MysqlInputSplit(long start, long end) {
	this.start = start;
	this.end = end;
	}
	
	public MysqlInputSplit() {
	}
	
	public void readFields(DataInput dataInput) throws IOException {
	this.start = dataInput.readLong();
	this.end = dataInput.readLong();
	}
	
	public void write(DataOutput dataOutput) throws IOException {
	dataOutput.writeLong(start);
	dataOutput.writeLong(end);
	}
	
	@Override
	public long getLength() throws IOException {
	return end - start;
	}
	
	@Override
	public String[] getLocations() throws IOException {
	return new String[] {};
	}
	
	public long getStart() {
	return start;
	}
	
	public long getEnd() {
	return end;
	}
	
	}
	
=====================================================================================================
	
	import java.io.IOException;
	import java.sql.Connection;
	import java.sql.PreparedStatement;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	
	import org.apache.hadoop.io.LongWritable;
	import org.apache.hadoop.mapreduce.InputSplit;
	import org.apache.hadoop.mapreduce.RecordReader;
	import org.apache.hadoop.mapreduce.TaskAttemptContext;
	
	public class MySqlRecordReader extends RecordReader<LongWritable, Person> {
	
	private ResultSet results = null;
	
	@Override
	public void initialize(InputSplit split, TaskAttemptContext context)
	throws IOException, InterruptedException {
	// do nothing
	
	}
	
	private final MysqlInputSplit split;
	private long pos = 0;
	
	private LongWritable key = null;
	
	private Person value = null;
	
	private Connection conn;
	
	protected PreparedStatement statement;
	
	public MySqlRecordReader(InputSplit split) {
	this.split = (MysqlInputSplit) split;
	}
	
	@Override
	public boolean nextKeyValue() throws IOException, InterruptedException {
	//System.out.println("ddd");
	try {
	if (key == null) {
	key = new LongWritable();
	}
	if (value == null) {
	value = new Person();
	}
	if (null == this.results) {
	String connString = "jdbc:mysql://172.20.29.29:3306/MapReduceInputTest";
	String sql = "select * from person LIMIT " + split.getLength()
	+ " OFFSET " + split.getStart();
	Class.forName("com.mysql.jdbc.Driver");
	conn = java.sql.DriverManager.getConnection(connString
	+ "?useUnicode=true&characterEncoding=utf8", "admin",
	"admin");
	statement = conn.prepareStatement(sql);
	this.results = statement.executeQuery();
	}
	
	if (!results.next()) {
	return false;
	}
	
	// Set the key field value as the output key value
	
	key.set(pos + split.getStart());
	value.setAge(results.getInt("age"));
	value.setName(results.getString("name"));
	pos++;
	} catch (Exception e) {
	e.printStackTrace();
	throw new IOException("SQLException in nextKeyValue", e);
	}
	return true;
	}
	
	@Override
	public LongWritable getCurrentKey() throws IOException,
	InterruptedException {
	return key;
	}
	
	@Override
	public Person getCurrentValue() throws IOException, InterruptedException {
	return value;
	}
	
	@Override
	public float getProgress() throws IOException, InterruptedException {
	// TODO Auto-generated method stub
	return 0;
	}
	
	@Override
	public void close() throws IOException {
	try {
	if (null != results) {
	results.close();
	}
	if (null != statement) {
	statement.close();
	}
	if (null != conn) {
	conn.close();
	}
	} catch (SQLException e) {
	throw new IOException(e.getMessage());
	}
	
	}
	
	}
	
	 

 