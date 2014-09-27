---
layout: post
title: eclipse hadoop job
category: Hadoop
date: 2014-09-27
---

使用EJob提交jar
在run中添加：

         conf.set("fs.default.name", "hdfs://192.168.241.7:9000");
          conf.set("mapred.job.tracker", "192.168.241.7:9001");
          Job job = new Job(conf, "....... my  job ......");
          try {
            File jarFile = EJob.createTempJar("bin");
            ((JobConf) job.getConfiguration()).setJar(jarFile.toString());
        } catch (IOException e) {
            throw new IOException(e);
        }
         
修改hadoop的hdfs-site.xml 关闭权限校验

     <property> 
          <name>dfs.permissions</name> 
          <value>false</value> 
     </property> 



**其他解决办法：**

hdfs的权限判断十分简单，就是拿发出指令的user name和文件的user name 做比较

  	private void check(INode inode, FsAction access
      ) throws AccessControlException {
    if (inode == null) {
      return;
    }
    FsPermission mode = inode.getFsPermission();

    if (user.equals(inode.getUserName())) { //user class
      if (mode.getUserAction().implies(access)) { return; }
    }
    else if (groups.contains(inode.getGroupName())) { //group class
      if (mode.getGroupAction().implies(access)) { return; }
    }
    else { //other class
      if (mode.getOtherAction().implies(access)) { return; }
    }
    throw new AccessControlException("Permission denied: user=" + user
        + ", access=" + access + ", inode=" + inode);
 	 }
	}

在多用户提交任务时遇到Permission denied, 的原因和任务提交的过程有关。

1. 首先提交任务的客户端会把任务相关文件打包放在hadoop.tmp.dir中，这是本地目录，需要通过本地系统权限验证。由于是临时目录直接设置成为777就行.
2.  客户端会将任务文件打包写入hdfs的	

		mapreduce.jobtracker.staging.root.dir + "/" + user + "/.staging" 目录，需要经过hdfs权限验证。通常可以选择两种方法解决。

		1) 保持mapreduce.jobtracker.staging.root.dir为默认，将此目录chmod 777
		2) 在hdfs的/user目录下建立用户目录，并且chown为该用户，相当于在hdfs下创建一个用户。
		然后设置mapreduce.jobtracker.staging.root.dir为/user,  这是官方推荐的，可以看到这个属性的描述是这样写的：
		The root of the staging area for users' job files In practice, this should be the directory where users' home directories are located (usually /user)。
		3）有人说可以在启动任务时加入hadoop.job.ugi属性指定用户和群组。 我验证这样不行。看源码里也是直接获取
		ugi = UserGroupInformation.getCurrentUser();不过看hdfs权限策略这操行应该可以设置。

