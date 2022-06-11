FROM ubuntu

LABEL \
    author="yuki<2487575080@qq.com>" \
    build-date="2022-06-11 15:05:03"

COPY readme.txt /usr/local/readme.txt

ENV MYPATH /usr/local/

WORKDIR $MYPATH

# 添加文件 ADD会自动解压文件
ADD jdk-8u301-linux-x64.tar.gz $MYPATH

ADD apache-tomcat-9.0.64.tar.gz $MYPATH

# RUN apt update

# RUN apt install vim

# 添加环境变量
ENV JAVA_HOME $MYPATH/jdk1.8.0_301

ENV CLASSPATH $JAVA_HOME/lib

ENV CATALINA_HOME $MYPATH/apache-tomcat-9.0.64

ENV CATALINA_BASH $MYPATH/apache-tomcat-9.0.64

# :拼接环境变量
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/bin:$CATALINA_HOME/lib

EXPOSE 8080

CMD $CATALINA_HOME/bin/startup.sh && tail -F $CATALINA_HOME/logs/catalina.out