:arrow_right: **Instalação manual**

Optei por subir uma máquina virtual com o Vagrant junto com o Virtualbox

```
vagrant init adavilag/ubuntu-server-22.04.1
```
```
root@victor-ubuntu:/home/victor/ubuntu-server-opencms# ls -lha
total 16K
drwxr-xr-x  3 root   root   4,0K out 19 11:49 .
drwxr-x--- 22 victor victor 4,0K out 19 11:47 ..
drwxr-xr-x  4 root   root   4,0K out 19 11:50 .vagrant
-rw-r--r--  1 root   root   3,4K out 19 11:49 Vagrantfile
```
Após a conclusão do processo, descomentei a seguinte linha para deixar a rede como bridged
```
  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network"
```
Após, executado o comando ```vagrant up``` para iniciar a máquina  

:warning: Requisitos do OpenCms

:arrow_right: OpenCms 18 is compatible with Java 21, 17, and 11.

:arrow_right: OpenCms supports Tomcat 9.x and 8.x.

Instalado versão 11 do OpenJDK
```
sudo apt install openjdk-11-jdk
```
```
root@vagrant:/opt/tomcat/logs# java --version
openjdk 11.0.24 2024-07-16
OpenJDK Runtime Environment (build 11.0.24+8-post-Ubuntu-1ubuntu320.04)
OpenJDK 64-Bit Server VM (build 11.0.24+8-post-Ubuntu-1ubuntu320.04, mixed mode, sharing)
```
Criado um usuário para o Tomcat, por questão de segurança
```
sudo useradd -m -d /opt/tomcat -U -s /bin/false tomcat
```
No diretório /tmp, efetuado download do Tomcat com wget
```
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.96/bin/apache-tomcat-9.0.96.tar.gz
```
```
root@vagrant:/tmp# ls -lha
total 13M
drwxrwxrwt 11 root root 4.0K Oct 19 14:57 .
drwxr-xr-x 19 root root 4.0K Oct 19 14:53 ..
-rw-r--r--  1 root root  13M Oct  3 20:27 apache-tomcat-9.0.96.tar.gz
drwxrwxrwt  2 root root 4.0K Oct 19 14:54 .font-unix
drwxr-xr-x  2 root root 4.0K Oct 19 14:56 hsperfdata_root
drwxrwxrwt  2 root root 4.0K Oct 19 14:54 .ICE-unix
drwx------  2 root root 4.0K Oct 19 14:54 netplan_eep5ooly
drwx------  3 root root 4.0K Oct 19 14:54 systemd-private-c0dd8f4b968a4b80b9f00ba531ea66c4-systemd-logind.service-9twWhf
drwx------  3 root root 4.0K Oct 19 14:54 systemd-private-c0dd8f4b968a4b80b9f00ba531ea66c4-systemd-resolved.service-1YcmBf
drwxrwxrwt  2 root root 4.0K Oct 19 14:54 .Test-unix
drwxrwxrwt  2 root root 4.0K Oct 19 14:54 .X11-unix
drwxrwxrwt  2 root root 4.0K Oct 19 14:54 .XIM-unix
```
Descompactado no diretório /opt/tomcat
```
sudo tar xzvf apache-tomcat-9*tar.gz -C /opt/tomcat --strip-components=1
```

Alterado o owner e o group do diretório para o usuário e grupo Tomcat
```
sudo chown -R tomcat:tomcat /opt/tomcat/
```

Liberado permissão de execução para o usuário Tomcat no /bin, onde contém os sripts essencias
```
sudo chmod -R u+x /opt/tomcat/bin no diretório /bin
```

Criado um serviço do Tomcat
```
sudo nano /etc/systemd/system/tomcat.service
```
```
[Unit]
Description=Tomcat
After=network.target

[Service]
Type=forking

User=tomcat
Group=tomcat

Environment="JAVA_HOME=/usr/lib/jvm/java-1.11.0-openjdk-amd64"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"
Environment="CATALINA_BASE=/opt/tomcat"
Environment="CATALINA_HOME=/opt/tomcat"
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"


ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/opt/tomcat/bin/shutdown.sh

RestartSec=10
Restart=always

[Install]
WantedBy=multi-user.target
```
Reiniciado o daemon, iniciado o Tomcat e habilitado na inicialização do sistema
```
systemctl daemon-reload
 sudo systemctl start tomcat
ystemctl status tomcat
● tomcat.service - Tomcat
     Loaded: loaded (/etc/systemd/system/tomcat.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-10-19 14:59:10 UTC; 14s ago
   Main PID: 3564 (java)
      Tasks: 28 (limit: 9515)
     Memory: 95.6M
     CGroup: /system.slice/tomcat.service
             └─3564 /usr/lib/jvm/java-1.11.0-openjdk-amd64/bin/java -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoad>

Oct 19 14:59:10 vagrant systemd[1]: Starting Tomcat...
Oct 19 14:59:10 vagrant startup.sh[3542]: Tomcat started.
Oct 19 14:59:10 vagrant systemd[1]: Started Tomcat.
```
Instalado o postgresql
```
sudo apt install postgresql
```
```
systemctl status postgresql
● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Sat 2024-10-19 12:01:02 -03; 17s ago
   Main PID: 4693 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 9515)
     Memory: 0B
     CGroup: /system.slice/postgresql.service

Oct 19 12:01:02 vagrant systemd[1]: Starting PostgreSQL RDBMS...
Oct 19 12:01:02 vagrant systemd[1]: Finished PostgreSQL RDBMS.
root@vagrant:/tmp# 
```
Verificado a versão do postgresql
```
psql --version

psql (PostgreSQL) 12.20 (Ubuntu 12.20-0ubuntu0.20.04.1)
```
No diretório /opt efetuado o download o opencms com wget
```
wget http://www.opencms.org/downloads/opencms/opencms-18.0.zip
```
Extraido o opencms 
```
unzip opencms-18.0.zip
```

Movido para o diretório dos apps do tomcat
```
mv opencms.war tomcat/webapps/
```
Reiniciado o tomcat
```
systemctl restart tomcat
```
Ajustado permissão no diretório do opencms do Tomcat
```
chown -R tomcat:tomcat /opt/tomcat/webapps/opencms
sudo chmod -R 755 /opt/tomcat/webapps/opencms
```
Opencms iniciado

```
19-Oct-2024 01:11:15.979 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/9.0.96
19-Oct-2024 01:11:15.989 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Oct 3 2024 19:44:30 UTC
19-Oct-2024 01:11:15.989 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version number: 9.0.96.0
19-Oct-2024 01:11:15.989 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
19-Oct-2024 01:11:15.989 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            5.4.0-99-generic
19-Oct-2024 01:11:15.992 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
19-Oct-2024 01:11:15.992 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /usr/lib/jvm/java-11-openjdk-amd64
19-Oct-2024 01:11:15.992 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           11.0.24+8-post-Ubuntu-1ubuntu320.04
19-Oct-2024 01:11:15.992 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor:            Ubuntu
19-Oct-2024 01:11:15.992 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:         /opt/tomcat
19-Oct-2024 01:11:15.992 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:         /opt/tomcat
19-Oct-2024 01:11:16.030 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.lang=ALL-UNNAMED
19-Oct-2024 01:11:16.030 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.io=ALL-UNNAMED
19-Oct-2024 01:11:16.030 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.util=ALL-UNNAMED
19-Oct-2024 01:11:16.030 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.util.concurrent=ALL-UNNAMED
19-Oct-2024 01:11:16.034 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
19-Oct-2024 01:11:16.034 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties
19-Oct-2024 01:11:16.034 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
19-Oct-2024 01:11:16.034 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.security.egd=file:///dev/urandom
19-Oct-2024 01:11:16.034 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djdk.tls.ephemeralDHKeySize=2048
19-Oct-2024 01:11:16.034 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources
19-Oct-2024 01:11:16.035 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027
19-Oct-2024 01:11:16.035 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Xms512M
19-Oct-2024 01:11:16.035 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Xmx1024M
19-Oct-2024 01:11:16.035 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -XX:+UseParallelGC
19-Oct-2024 01:11:16.035 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dignore.endorsed.dirs=
19-Oct-2024 01:11:16.035 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.base=/opt/tomcat
19-Oct-2024 01:11:16.035 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.home=/opt/tomcat
19-Oct-2024 01:11:16.035 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/opt/tomcat/temp
19-Oct-2024 01:11:16.042 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent The Apache Tomcat Native library which allows using OpenSSL was not found on the java.library.path: [/usr/java/packages/lib:/usr/lib/x86_64-linux-gnu/jni:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu:/usr/lib/jni:/lib:/usr/lib]
19-Oct-2024 01:11:16.905 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
19-Oct-2024 01:11:16.982 INFO [main] org.apache.catalina.startup.Catalina.load Server initialization in [1566] milliseconds
19-Oct-2024 01:11:17.137 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
19-Oct-2024 01:11:17.138 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet engine: [Apache Tomcat/9.0.96]
19-Oct-2024 01:11:17.172 INFO [main] org.apache.catalina.startup.HostConfig.deployWAR Deploying web application archive [/opt/tomcat/webapps/opencms.war]
NOTE: Picked up JDK_JAVA_OPTIONS:  --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
19-Oct-2024 01:11:42.331 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/9.0.96
19-Oct-2024 01:11:42.340 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Oct 3 2024 19:44:30 UTC
19-Oct-2024 01:11:42.341 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version number: 9.0.96.0
19-Oct-2024 01:11:42.341 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
19-Oct-2024 01:11:42.341 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            5.4.0-99-generic
19-Oct-2024 01:11:42.341 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
19-Oct-2024 01:11:42.341 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /usr/lib/jvm/java-11-openjdk-amd64
19-Oct-2024 01:11:42.346 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           11.0.24+8-post-Ubuntu-1ubuntu320.04
19-Oct-2024 01:11:42.346 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor:            Ubuntu
19-Oct-2024 01:11:42.346 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:         /opt/tomcat
19-Oct-2024 01:11:42.346 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:         /opt/tomcat
19-Oct-2024 01:11:42.398 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.lang=ALL-UNNAMED
19-Oct-2024 01:11:42.398 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.io=ALL-UNNAMED
19-Oct-2024 01:11:42.399 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.util=ALL-UNNAMED
19-Oct-2024 01:11:42.399 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.util.concurrent=ALL-UNNAMED
19-Oct-2024 01:11:42.399 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
19-Oct-2024 01:11:42.399 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.config.file=/opt/tomcat/conf/logging.properties
19-Oct-2024 01:11:42.399 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
19-Oct-2024 01:11:42.399 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.security.egd=file:///dev/urandom
19-Oct-2024 01:11:42.399 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djdk.tls.ephemeralDHKeySize=2048
19-Oct-2024 01:11:42.400 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources
19-Oct-2024 01:11:42.400 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027
19-Oct-2024 01:11:42.400 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dignore.endorsed.dirs=
19-Oct-2024 01:11:42.400 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.base=/opt/tomcat
19-Oct-2024 01:11:42.400 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.home=/opt/tomcat
19-Oct-2024 01:11:42.400 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/opt/tomcat/temp
19-Oct-2024 01:11:42.414 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent The Apache Tomcat Native library which allows using OpenSSL was not found on the java.library.path: [/usr/java/packages/lib:/usr/lib/x86_64-linux-gnu/jni:/lib/x86_64-linux-gnu:/usr/lib/x86_64-linux-gnu:/usr/lib/jni:/lib:/usr/lib]
19-Oct-2024 01:11:43.343 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
19-Oct-2024 01:11:43.414 INFO [main] org.apache.catalina.startup.Catalina.load Server initialization in [1716] milliseconds
19-Oct-2024 01:11:43.569 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
19-Oct-2024 01:11:43.569 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet engine: [Apache Tomcat/9.0.96]
19-Oct-2024 01:11:43.600 INFO [main] org.apache.catalina.startup.HostConfig.deployWAR Deploying web application archive [/opt/tomcat/webapps/opencms.war]
19-Oct-2024 01:11:57.753 INFO [main] org.apache.jasper.servlet.TldScanner.scanJars At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
19-Oct-2024 01:11:59.067 INFO [main] org.apache.catalina.startup.HostConfig.deployWAR Deployment of web application archive [/opt/tomcat/webapps/opencms.war] has finished in [15,466] ms
19-Oct-2024 01:11:59.071 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/opt/tomcat/webapps/examples]
19-Oct-2024 01:11:59.419 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/opt/tomcat/webapps/examples] has finished in [348] ms
19-Oct-2024 01:11:59.421 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/opt/tomcat/webapps/host-manager]
19-Oct-2024 01:11:59.470 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/opt/tomcat/webapps/host-manager] has finished in [49] ms
19-Oct-2024 01:11:59.478 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/opt/tomcat/webapps/ROOT]
19-Oct-2024 01:11:59.511 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/opt/tomcat/webapps/ROOT] has finished in [32] ms
19-Oct-2024 01:11:59.512 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/opt/tomcat/webapps/manager]
19-Oct-2024 01:11:59.562 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/opt/tomcat/webapps/manager] has finished in [50] ms
19-Oct-2024 01:11:59.563 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/opt/tomcat/webapps/docs]
19-Oct-2024 01:11:59.601 INFO [main] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/opt/tomcat/webapps/docs] has finished in [38] ms
19-Oct-2024 01:11:59.616 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
19-Oct-2024 01:11:59.656 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [16238] milliseconds
Setup-Bean initialized successfully.
Check runtime connection....
Check runtime connection - COMPLETED
Check setup connection....
Check setup connection - COMPLETED
DB connection tested successfully.
Database created successfully.
Tables created successfully.
Database setup was successful.
19-Oct-2024 12:17:05.009 WARNING [OpenCms: Indexing 'OpenCms-is-amazing.pdf'] org.apache.tika.config.InitializableProblemHandler$3.handleInitializableProblem org.xerial's sqlite-jdbc is not loaded.
Please provide the jar on your classpath to parse sqlite files.
See tika-parsers/pom.xml for the correct version.
Starting OpenCms, version 18.0 in web application "opencms"

Copyright (c) 2002 - 2024 Alkacon Software GmbH & Co. KG
OpenCms comes with ABSOLUTELY NO WARRANTY
This is free software, and you are welcome to
redistribute it under certain conditions.
Please see the GNU Lesser General Public Licence for
further details.


Not starting JLAN server because no config file was found at /opt/tomcat/webapps/opencms/WEB-INF/config/jlanConfig.xml

```
Configuração do usuário no banco para rodar o setup do OpenCMS
```
sudo -u post postgres

CREATE ROLE opencmssetup WITH LOGIN PASSWORD 'opencms';

ALTER ROLE opencmssetup SUPERUSER;

Para sair da linha de comando

\q
```

<img src="final-instalacao.png">

Instalação do Nginx
```
apt-get install nginx
```
Criado  um arquivo de configuração para o OpenCMS

```server_name opencms.local; ``` define o domínio local que será utilizado para acessar o OpenCms.

```proxy_pass http://192.168.1.13:8080; ``` as requisições serão redirecionadas para o Tomcat.

```
nano /etc/nginx/sites-available/opencms
```
```
server {
    listen 80;
    server_name opencms.local;

    location / {
        proxy_pass http://192.168.1.13:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

<img src="final-nginx.png">

Utils:

```
/tomcat/webapps/opencms/WEB-INF/logs/opencms.log
```
```
/tomcat/logs/catalina.out
```
```
/var/log/nginx/access.log
/var/log/nginx/error.log
```

:arrow_right: **Subindo o serviço com Docker**

Instalação do Docker

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
Add the repository to Apt sources:
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Criado um diretório /home/victor/opencsm para armazenas o compose, dockerfiles e arquivos de config

Diretório /home/victor/opencsm, com o docker-compose.yml
```
.:
total 28K
drwxr-xr-x  6 root   root   4,0K out 20 21:34 .
drwxr-x--- 24 victor victor 4,0K out 20 21:34 ..
-rw-r--r--  1 root   root    901 out 20 17:12 docker-compose.yml
drwxr-xr-x  2 root   root   4,0K out 20 17:16 nginx
drwxr-xr-x  6 root   root   4,0K out 20 16:07 opencms
drwxr-xr-x  2 root   root   4,0K out 20 11:11 tomcat
```
```
version: '3'

services:
  
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: opencmssetup
      POSTGRES_PASSWORD: opencmspass
      POSTGRES_DB: opencms
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - opencms-network

  tomcat:
    build:
      context: ./tomcat
    depends_on:
      - postgres
    ports:
      - "8080:8080"
    networks:
      - opencms-network
    environment:
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_NAME=opencms
      - DB_USER=opencmssetup
      - DB_PASS=opencmspass
    volumes:
      - ./opencms:/usr/local/tomcat/webapps/opencms

  nginx:
    build:
      context: ./nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - tomcat
    networks:
      - opencms-network


volumes:
  postgres-data:

networks:
  opencms-network:
```

Diretório opencsm/nginx, contendo o Dockerfile e mapeamento de volumes
```
./nginx:
total 20K
drwxr-xr-x 2 root root 4,0K out 20 17:16 .
drwxr-xr-x 6 root root 4,0K out 20 21:34 ..
-rw-r--r-- 1 root root  423 out 20 09:43 Dockerfile
-rw-r--r-- 1 root root  875 out 20 17:16 nginx.conf
-rw-r--r-- 1 root root  376 out 20 16:55 opencms
```
```
# Imagem do Nginx
FROM nginx:latest

# Criar diretório dos sites se não existirem
RUN mkdir -p /etc/nginx/sites-available /etc/nginx/sites-enabled

# Copiar a configuração do site para o container
COPY opencms /etc/nginx/sites-available/opencms

# Lik simbolico para halitar o site
RUN ln -s /etc/nginx/sites-available/opencms /etc/nginx/sites-enabled/opencms

# Exponha a porta 80
EXPOSE 80
```

```
server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://opencms-tomcat-1:8080/;  # Direciona as requisições para o Tomcat
        proxy_set_header Host $host;              # Passa o host original
        proxy_set_header X-Real-IP $remote_addr;  # Passa o IP do cliente
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # Adiciona o cabeçalho para proxies
        proxy_set_header X-Forwarded-Proto $scheme;  # Adiciona o esquema (http ou https)
    }

    error_page 404 /404.html;  # Páginas de erro personalizadas
    location = /404.html {
        root /usr/share/nginx/html;  # Local onde está o arquivo 404.html
    }

    error_page 500 502 503 504 /50x.html;  # Páginas de erro personalizadas
    location = /50x.html {
        root /usr/share/nginx/html;  # Local onde está o arquivo 50x.html
    }
}
```

Diretório opencsm/tomcat, contendo o Dockerfile
```
./tomcat: 
total 238M
drwxr-xr-x 2 root root 4,0K out 20 11:11 .
drwxr-xr-x 6 root root 4,0K out 20 21:34 ..
-rwxr-xr-x 1 root root  587 out 20 11:11 Dockerfile
-rw-r--r-- 1 root root 238M out  7 10:54 opencms.war
```
```
# Usando a imagem base do Tomcat 9
FROM tomcat:9.0

# Diretório de trabalho
WORKDIR /usr/local/tomcat

# Copiar o OpenCms WAR para o diretório webapps do Tomcat
COPY opencms.war /usr/local/tomcat/webapps/

# Ajustar as permissões para o usuário tomcat
RUN useradd -m -d /opt/tomcat -U -s /bin/false tomcat
RUN chmod -R u+x /usr/local/tomcat/bin

# Expõe a porta padrão do Tomcat
EXPOSE 8080

# Inicia o Tomcat
CMD ["catalina.sh", "run"]
```

Diretório opencsm/opencsm, contendo mapeamento de volume
```
./opencms:
total 238M
drwxr-xr-x  6 root root 4,0K out 20 16:07 .
drwxr-xr-x  6 root root 4,0K out 20 21:34 ..
drwxr-x---  4 root root 4,0K out 20 16:07 export
drwxr-xr-x  2 root root 4,0K out  7 06:37 META-INF
-rw-r--r--  1 root root 238M out 20 11:16 opencms.war
drwxr-xr-x  9 root root 4,0K out 20 15:39 resources
drwxr-xr-x 17 root root 4,0K out 20 16:07 WEB-INF
```

Usei o comando ```docker-compose up --build``` para subir os containers

```
docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED        STATUS       PORTS                                       NAMES
fe42299eda2d   opencms-nginx    "/docker-entrypoint.…"   5 hours ago    Up 5 hours   0.0.0.0:80->80/tcp, :::80->80/tcp           opencms-nginx-1
b43e8c9becb7   opencms-tomcat   "catalina.sh run"        11 hours ago   Up 5 hours   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   opencms-tomcat-1
6d49dd060d6d   postgres:13      "docker-entrypoint.s…"   13 hours ago   Up 5 hours   5432/tcp                                    opencms-postgres-1
```
Precisei efetuar a extração manual do opencms.war no /usr/local/tomcat/webapps/

```
docker exec -it  b43e8c9becb7 /bin/bash
jar -xvf opencms.war -C /usr/local/tomcat/webapps/opencms
```
<img src="inicial-docker.png">

Durante a configuração deu alguns erros em relação ao usuário do banco e também com conexão já ativa

```
postgres-1  | 2024-10-20 14:56:19.988 UTC [98] FATAL:  database "opencms" does not exist
postgres-1  | 2024-10-20 14:56:25.004 UTC [99] ERROR:  source database "template1" is being accessed by other users
postgres-1  | 2024-10-20 14:56:25.004 UTC [99] DETAIL:  There is 1 other session using the database.

postgres-1  | 2024-10-20 14:56:19.988 UTC [98] FATAL:  database "opencms" does not exist
postgres-1  | 2024-10-20 14:56:25.004 UTC [99] ERROR:  source database "template1" is being accessed by other users
postgres-1  | 2024-10-20 14:56:25.004 UTC [99] DETAIL:  There is 1 other session using the database.
```

Corrigi reiniciado os containers, com isso encerrando a sessão ativa

Após o final da configuração, efetuado a configuração do Nginx

Nessa parte, o nginx estava pegando a rota padrão, mesmo já tendo o arquivo de config do opencms no sites-enable e no sites-available

```
nginx-1     | 2024/10/20 19:17:43 [error] 23#23: *1 "/usr/share/nginx/html/opencms/system/login/index.html" is not found (2: No such file or directory), client: 172.18.0.1, server: localhost, request: "GET /opencms/system/login/ HTTP/1.1", host: "opencms.local"
```
A solução que eu encontrei foi mapear a config que eu aplique no sites-enabled e no site-available para o default.conf

```
volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
```

<img src="final-docker.png">
