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
$ shutdown.sh
```

---
### Build Docker images 

- Oracle Database 11g R2
- SGI Gateway Application on Tomcat 8.5 (2 Instances)
- Nginx for Load Balancer

```sh
$ vi docker-compose.yaml
$ docker-compose build
$ docker-compose up --scale sgigw_app=2
$ docker-compose down
```

---
### Install Oracle Database 11g xe

```sh
$ docker pull alexeiled/docker-oracle-xe-11g
$ vi oracle-dockerfile
$ docker build -t sgigw_db:1.0.1 -f docker/oracle-dockerfile .
$ docker run -d -p 
```

---
### Docker Container

```sh
$ vi docker/tomcat8-dockerfile
$ vi docker/ROOT.xml
$ docker build -t sgigw_app:1.0.1 -f docker/tomcat8-dockerfile .
$ docker run -d -p 8080:8080 --name sgigw sgigw_app:1.0.1
$ open http://localhost:8080
``` 

---
### Docker Composer

```sh
$ vi nginx-dockerfile
$ vi nginx.conf
$ vi docker-compose.yml
$ docker-compose up -d --scale sgigw_app=2
```

---
### Push Docker image

```sh
$ docker tag sgigw_app:1.0.1 credemol/sgigw_app:1.0.1
$ docker login
$ docker push credemol/sgigw_app:1.0.1
$ open https://hub.docker.com/r/credemol/sgigw_app/
```

---
### Kubernetes (Deployment)

```sh
$ minikube start
$ vi docker-compose-v3.yml
$ kompose convert -f docker-compose-v3.yml
$ vi sgigw-deployment.yaml
$ vi sgigw-service.yaml
$ kubectl get all
$ kubectl create -f sgigw-deployment.yaml
$ kubectl get all
```

---
### Kubernetes (Service)

```sh
$ kubectl create -f sgigw-service.yaml
$ kubectl get all
$ kubectl get svc 
$ kubectl describe svc sgigw
$ minikube service sgigw --url
```

---
### Kubernetes (Scale Out/In)

```sh
$ kubectl get deployments
$ kubectl scale --replicas=4 deployment sgigw
$ kubectl get deployments 
$ kubectl get pods
$ kubectl describe svc sgigw
```

---
### Kubernetes (Delete resources)

```sh
$ kubectl get all --show-labels
$ kubectl delete all --selector='io.kompose.service=sgigw'
$ kubectl delete svc sgigw
```

---
### Minikube Dashboard

```sh
$ minikube dashboard
```
> Update the value of replicas in Deployments 

```sh
$ kubectl get pods
$ kubectl deployments pods
```





