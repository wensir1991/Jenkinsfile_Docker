# Jenkinsfile构建java_tomcat项目
构建步骤
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
