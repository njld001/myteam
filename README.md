##MAVEN 安装文档
###安装JDK是1.6版本
	环境变量设置JAVA_HOME
###将 nexus-2.5.1.zip解压放到E:\MAVEN\下

####双击 install-nexus.bat
	E:\MAVEN\nexus-2.5.1\nexus-2.5.1-01\bin\jsw\windows-x86-64\install-nexus.bat
	windows-x86-64是你的系统32位系统选windows-x86-32
	在运行下输入services.msc 弹出服务窗口将nexus服务改成自动
	正常启动起来输入网址http://localhost:8081/nexus 用户名admin密码admin123
	错误情况请参照E:\MAVEN\nexus-2.5.1\nexus-2.5.1-01\logs\wrapper.log文件的内容进行 修改


###将apache-maven-3.2.1-bin.zip解压放到E:\MAVEN\下

####修改E:\MAVEN\apache-maven-3.2.1\conf\settings.xml文件

#####搜索[localRepository]在它下面追加一行[E:/MAVEN/m2/repository是你本地仓库地址]

	<localRepository>E:/MAVEN/m2/repository</localRepository>

- - -

#####搜索[profile]在它下面[http://localhost:8081/nexus是你的私服地址]

	 <profile>
	   <id>dev</id>
	   <repositories>
	        <repository>
	            <id>nexus</id>               
	            <url>http://localhost:8081/nexus/content/groups/public/</url>
	            <releases>
	                <enabled>true</enabled>
	            </releases>
	            <snapshots>
	                <enabled>true</enabled>
	            </snapshots>
	         </repository>
	    </repositories>          
	    <pluginRepositories>
	          <pluginRepository>
	              <id>nexus</id>
	              <url>http://localhost:8081/nexus/content/groups/public</url>
	              <releases>
	                  <enabled>true</enabled>
	              </releases>
	              <snapshots>
	                  <enabled>true</enabled>
	               </snapshots>
	           </pluginRepository>
	     </pluginRepositories>
	</profile>
