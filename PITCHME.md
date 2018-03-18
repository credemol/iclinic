# Hi iClinic System

---
## Agenda
>1. Demo
>2. Run Application locally
>3. Docker Image and Container

---
## Demo

> Samsung Medial Center
> Samsung Genome Institute
> SGI Gateway Service: 

[SGI Gateway Application](https://sgigateway1-sgi.uscom-central-1.oraclecloud.com/)
[Oracle Developer Cloud Service](https://developer.us2.oraclecloud.com/developer22241-sgi/)
---
### Run SGI Gateway Application locally.

```sh
$ mvn clean package
$ cp -f target/sgi_gateway-1.0.0.war $CATALINA_HOME/deploy
$ startup.sh
$ open http://localhost:8080
```

---
### Docker Container

```sh
$ vi tomcat-dockerfile
$ vi 
$ docker build -t sgi_gateway_tomcat85:1.0.0 -f tomcat-dockerfile .
$ docker run -d -p 18080:8080 --name sgigw sgi_gateway_tomcat85:1.0.0
$ open http://localhost:18080
``` 

---
### Docker Composer

```sh
$ vi nginx-dockerfile
```