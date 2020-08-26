#!/usr/bin/env groovy

def registry = "https://index.docker.io/v1/" 
def project = "welcome"
def app_name = "demo"
def image_name = "wenjusir1991/wenju:${Branch}-${BUILD_NUMBER}"  
def git_address = "https://github.com/wensir1991/tomcat-java-demo.git" 
def docker_registry_auth = "f517118b-f509-40e3-adbe-198d0f3e169c"
def git_auth = "320fa214-7b6f-4f05-9b97-a693c073aa85"                  

pipeline {
    agent any
    stages {
        stage('拉取代码'){
            steps {
              checkout([$class: 'GitSCM', branches: [[name: '${Branch}']], userRemoteConfigs: [[credentialsId: "${git_auth}", url: "${git_address}"]]])
            }
        }

        stage('代码编译'){
           steps {
             sh """
                JAVA_HOME=/usr/local/jdk
                PATH=$JAVA_HOME/bin:/usr/local/maven/bin:$PATH
                mvn clean package -Dmaven.test.skip=true
                """ 
           }
        }

        stage('构建镜像'){
           steps {
                withCredentials([usernamePassword(credentialsId: "${docker_registry_auth}", passwordVariable: 'password', usernameVariable: 'username')]) {
                sh """
                  echo '
                    FROM wenjusir1991/wenju:nginx_tomcat
                    LABEL maitainer sunwenju
                    RUN rm -rf /usr/local/tomcat/webapps/*
                    ADD target/*.war /usr/local/tomcat/webapps/ROOT.war
                  ' > Dockerfile
                  docker build -t ${image_name} .
                  docker login -u ${username} -p '${password}' ${registry}
                  docker push ${image_name}
                """
                }
           } 
        }

        stage('部署到Docker'){
           steps {
              sh """
              docker rm -f tomcat-java-demo |true
              docker container run -d --name tomcat-java-demo -p 8080:8080 ${image_name}
              """
            }
        }
    }
}

