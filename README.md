# Jenkinsfile构建java_tomcat项目(基于Docker)
1.安装部署Jenkins
Jenkins依赖java环境
1.1.配置JDK和Maven环境变量
#wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"
# tar zxvf jdk-8u141-linux-x64.tar.gz
# mv jdk1.8.0_141/ /usr/local/jdk
1.2配置jdk环境变量
# vim /etc/profile
JAVA_HOME=/usr/local/jdk
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
MAVEN_HOME=/usr/local/maven
PATH=$JAVA_HOME/bin:$MAVEN_HOME/bin:$PATH
export JAVA_HOME CLASSPATH MAVEN_HOME PATH
#source /etc/profile
1.3部署maven环境
# wget http://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
# tar zxf apache-maven-3.6.3-bin.tar.gz
# mv apache-maven-3.6.3 /usr/local/maven
#  wget http://archive.apache.org/dist/tomcat/tomcat-8/v8.5.31/bin/apache-tomcat-8.5.31.tar.gz
# tar zxf apache-tomcat-8.5.31.tar.gz
# mv apache-tomcat-8.5.31 /usr/local/tomcat
非root用户运行Tomcat，需在[wenjusir@githarbor ~]$ sudo vim /usr/local/tomcat/bin/setclasspath.sh 加入环境变量

export JAVA_HOME=/usr/local/jdk
export JRE_HOME=/usr/local/jdk/jre

1.4.修改Maven源
# vim /usr/local/maven/conf/settings.xml，在<mirrors>字段下增加字段
 <mirrors>
    
    <mirror>     
      <id>central</id>     
      <mirrorOf>central</mirrorOf>     
      <name>aliyun maven</name>
      <url>https://maven.aliyun.com/repository/public</url>     
    </mirror>
    
    </mirrors>
1.5.Docker部署Jenkins
#docker run -d --name jenkins -p 80:8080 -p 50000:50000 -u root  \
   -v /opt/jenkins_home:/var/jenkins_home \
   -v /var/run/docker.sock:/var/run/docker.sock   \
   -v /usr/bin/docker:/usr/bin/docker \
   -v /usr/local/maven:/usr/local/maven \
   -v /usr/local/jdk:/usr/local/jdk \
   -v /etc/localtime:/etc/localtime \
   --restart=always \
   --name jenkins jenkins/jenkins:lts

IP+80端口，进入初始化页面，插件可以后面安装
1.6.更改Jenkins源(解决下载插件慢的问题)
更改持续化数据目录下的文件(通过docker部署安装的Jenkins)
#vim /opt/jenkins_home/updates/default.json
搜索字段jenkins-ci.org
更改源
#sed -i 's/https:\/\/updates.jenkins.io\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /opt/jenkins_home/updates/default.json
# sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /opt/jenkins_home/updates/default.json
重启容器使配置文件生效
# docker restart jenkins

1.7.编写 Pipeline 流水线脚本
Jenkinsfile具体内容请查看https://github.com/wensir1991/Jenkinsfile

2.构建步骤
首先创建一个pipeline(流水线)项目，接着:
1.项目配置个Parameter参数(参数化构建)
路径:新建的pipeline项目下， Configure-->勾选This project is parameterized，选择String Parameter,Name:Branch; Default value:master;
Description:随便填写描述信息，例如:要发布的代码分支
2.Jenkins配置登录harbor镜像仓库用户名和密码，以及登录git仓库的用户名和密码
路径: Manage Jenkins-->Mange Credentials
填写完后，会生成一个凭据ID，将这个ID复制到Jenkinsfile文件代码中，复制到Jenkinsfile文件对应的docker_registry_auth字段和git_auth 字段后面
3.可以使用pipeline模板生成拉取代码凭据代码
路径:新建的pipeline项目下， Configure-->拉倒最下面的Pipeline Syntax，
    3.1 生成拉取私有代码仓库的代码凭据语法
    Sample Step选择:check out from version control;Repository URL添加代码仓库地址，例如:https://github.com/wensir1991/tomcat-java-demo.git;
    Credtentials 选择拉取代码的凭据
    3.2 生成拉取私有镜像仓库的镜像凭据语法
    Sample Step选择:withCredentials:Bind credentials to variables,然后点击add,keystore Variable:username,Password Variable:password,
    接着讲相应的语法复制到Jenkinsfile文件中，并将里面的值改成变量值
4.开始构建项目
将生成的代码复制到Jenkinsfile文件中，然后将Jenkins文件所有的内容复制到pipeline项目里的pipeline下的Script里，保存
5.接着开始构建项目，并查看控制台输出信息,可以看到构建是否报错
路径:新建的pipeline项目下，Build with Parameters
