---
layout: post
title: Mysql outputformat简单实现
category: Hadoop
date: 2014-09-27
---
================================OutputFormat=========

	import java.io.IOException;
	import java.sql.Connection;
	import java.sql.Statement;
	import java.util.List;
	
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.io.NullWritable;
	import org.apache.hadoop.mapreduce.Job;
	import org.apache.hadoop.mapreduce.JobContext;
	import org.apache.hadoop.mapreduce.OutputCommitter;
	import org.apache.hadoop.mapreduce.OutputFormat;
	import org.apache.hadoop.mapreduce.RecordWriter;
	import org.apache.hadoop.mapreduce.TaskAttemptContext;
	import org.apache.hadoop.mapreduce.TaskInputOutputContext;
	import org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter;
	import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
	
	import com.augmentum.rob.input.Person;
	
	public class MySqlOutputFormat extends OutputFormat<NullWritable, Person> {
	
	@Override
	public RecordWriter<NullWritable, Person> getRecordWriter(
	TaskAttemptContext context) throws IOException,
	InterruptedException {
	MysqlRecordWriter mysqlRecordWriter = null;
	try {
	String connString = "jdbc:mysql://172.20.29.29:3306/MapReduceInputTest";
	Class.forName("com.mysql.jdbc.Driver");
	Connection connection = java.sql.DriverManager.getConnection(
	connString + "?useUnicode=true&characterEncoding=utf8",
	"admin", "admin");
	Statement statement = connection.createStatement();
	mysqlRecordWriter = new MysqlRecordWriter(connection, statement);
	
	} catch (Exception ex) {
	throw new IOException(ex.getMessage());
	}
	
	return mysqlRecordWriter;
	}
	
	@Override
	public void checkOutputSpecs(JobContext context) throws IOException,
	InterruptedException {
	// TODO Auto-generated method stub
	// 判断输出表是否设置
	
	// 判断输出表是否存在
	
	}
	
	@Override
	public OutputCommitter getOutputCommitter(TaskAttemptContext context)
	throws IOException, InterruptedException {
	return new FileOutputCommitter(
	FileOutputFormat.getOutputPath(context),
	context);
	}
	
	}
	
==========================RecordWriter=====================
	
	import java.io.IOException;
	import java.sql.Connection;
	import java.sql.SQLException;
	import java.sql.Statement;
	import java.util.List;
	
	import org.apache.commons.logging.Log;
	import org.apache.commons.logging.LogFactory;
	import org.apache.hadoop.io.NullWritable;
	import org.apache.hadoop.mapreduce.RecordWriter;
	import org.apache.hadoop.mapreduce.TaskAttemptContext;
	import org.apache.hadoop.util.StringUtils;
	
	import com.augmentum.rob.input.Person;
	public class MysqlRecordWriter extends RecordWriter<NullWritable, Person>{
	
	private Connection connection;
	private Statement statement;
	private static final Log LOG = LogFactory.getLog(MysqlRecordWriter.class);
	
	public MysqlRecordWriter() throws SQLException {}
	
	public MysqlRecordWriter(Connection connection, Statement statement) throws SQLException {
	this.connection = connection;
	this.statement = statement;
	this.connection.setAutoCommit(false);
	}
	
	public Connection getConnection() {
	return connection;
	}
	
	public Statement getStatement() {
	return statement;
	}
	
	@Override
	public void write(NullWritable key, Person person) throws IOException, InterruptedException {
	String sql = "INSERT person2(name, age) VALUES('"+person.getName()+"', "+person.getAge()+")";
	//可以根据key和value情况写多条INSERT SQL
	// List<String> sqlList = buildInsertSql();
	try {
	// if (sqlList != null && sqlList.size() > 0) {
	// for (String sql : sqlList) {
	// statement.addBatch(sql);
	// }
	// }
	statement.addBatch(sql);
	} catch (SQLException e) {
	e.printStackTrace();
	}
	}
	
	// //更具逻辑具体实现插入sql。
	// public abstract List<String> buildInsertSql();
	
	@Override
	public void close(TaskAttemptContext context) throws IOException, InterruptedException {
	try {
	statement.executeBatch();
	connection.commit();
	} catch (SQLException e) {
	try {
	connection.rollback();
	}
	catch (SQLException ex) {
	LOG.warn(StringUtils.stringifyException(ex));
	}
	throw new IOException(e.getMessage());
	} finally {
	try {
	statement.close();
	connection.close();
	}
	catch (SQLException ex) {
	throw new IOException(ex.getMessage());
	}
	}
	
	}
	
	}
	
======================================================================================================
	
	import java.io.IOException;
	import java.sql.Connection;
	import java.sql.PreparedStatement;
	
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.mapreduce.Job;
	import org.apache.hadoop.mapreduce.JobContext;
	import org.apache.hadoop.mapreduce.RecordWriter;
	import org.apache.hadoop.mapreduce.TaskAttemptContext;
	import org.apache.hadoop.mapreduce.TaskInputOutputContext;
	import org.apache.hadoop.mapreduce.lib.db.DBConfiguration;
	import org.apache.hadoop.mapreduce.lib.db.DBOutputFormat;
	import org.apache.hadoop.mapreduce.lib.db.DBWritable;
	import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
	
	public class RobOut<K extends DBWritable, V> extends DBOutputFormat<K, V>{
	
	/** {@inheritDoc} */
	public RecordWriter<K, V> getRecordWriter(TaskAttemptContext context)
	throws IOException {
	DBConfiguration dbConf = new DBConfiguration(context.getConfiguration());
	
	String tableName = dbConf.getOutputTableName();
	String[] fieldNames = dbConf.getOutputFieldNames();
	
	if(fieldNames == null) {
	fieldNames = new String[dbConf.getOutputFieldCount()];
	}
	
	try {
	Connection connection = dbConf.getConnection();
	PreparedStatement statement = null;
	
	statement = connection.prepareStatement(
	constructQuery(tableName, fieldNames));
	return new DBRecordWriter(connection, statement);
	} catch (Exception ex) {
	throw new IOException(ex.getMessage());
	}
	}
	
	/**
	* Set the {@link Path} of the output directory for the map-reduce job.
	*
	* @param job The job to modify
	* @param outputDir the {@link Path} of the output directory for
	* the map-reduce job.
	*/
	public static void setOutputTable(Job job, String tableName) {
	job.getConfiguration().set("output.mysql.table" + tableName, tableName);
	}
	
	/**
	* Get the {@link Path} to the output directory for the map-reduce job.
	*
	* @return the {@link Path} to the output directory for the map-reduce job.
	* @see FileOutputFormat#getWorkOutputPath(TaskInputOutputContext)
	*/
	public static Path getOutputTable(JobContext job) {
	String name = job.getConfiguration().get("mapred.output.table");
	return name == null ? null: new Path(name);
	}
	
	}