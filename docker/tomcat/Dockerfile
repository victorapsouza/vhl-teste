# Usando a imagem base do Tomcat 9
FROM tomcat:9.0

RUN apt-get update && apt-get install -y openjdk-11-jdk

RUN update-alternatives --set java /usr/lib/jvm/java-11-openjdk-amd64/bin/java

# Defina o diretório de trabalho
WORKDIR /usr/local/tomcat

# Copie o OpenCms WAR para o diretório webapps do Tomcat
COPY opencms.war /usr/local/tomcat/webapps/

# Ajuste as permissões para o usuário tomcat
RUN useradd -m -d /opt/tomcat -U -s /bin/false tomcat
RUN chmod -R u+x /usr/local/tomcat/bin

# Expõe a porta padrão do Tomcat
EXPOSE 8080

# Inicia o Tomcat
CMD ["catalina.sh", "run"]
