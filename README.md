##MAVEN 安装文档

###将apache-maven-3.2.1-bin.zip解压放到E:\MAVEN\下

###修改E:\MAVEN\apache-maven-3.2.1\conf\settings.xml文件

#####搜索[localRepository]在它下面追加一行[d:/MAVEN/m2/repository是你本地仓库地址]

	<localRepository>d:/MAVEN/m2/repository</localRepository>

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
