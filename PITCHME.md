# Presentation for iClinic

---
# Agenda

- Using Docker & Kubernetes
- Spring Boot and API First Strategy

---
## Using Docker & Kubernetes

1. Samsung Genome Institute Demo
1. Setting up Environment (Oracle, Tomcat, Nginx)
1. Run Application locally
1. Docker Image and Container
1. Taking advantage of using Kubernetes

---
## Spring Boot and API First Strategy

1. Spring Boot Application
1. Set up Environment(MySQL, Spring Boot Application)
1. Build REST API & Swagger
1. Using Swagger Codegen & API Transformer

---
## Demo

- Samsung Medial Center
- Samsung Genome Institute
- SGI Gateway Service: 

- [Presentation: https://gitpitch.com/credemol/iclinic/master](https://gitpitch.com/credemol/iclinic/master)
- [SGI Gateway Application](https://sgigateway1-sgi.uscom-central-1.oraclecloud.com/)
- [Oracle Developer Cloud Service](https://developer.us2.oraclecloud.com/developer22241-sgi/)


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
$ vi docker/oracle-dockerfile
$ docker build -t sgigw_db:1.0.1 -f docker/oracle-dockerfile .
$ docker run -d -p 1521:1521 --name sgigw_db sgigw_db:1.0.1
```


---
### Run SGI Gateway Application locally.

```sh
$ mvn clean package
$ cp -f target/sgi_gateway-1.0.0.war $CATALINA_HOME/deploy
$ export SGI_JDBC_URL=jdbc:oracle:thin:@localhost:1521:xe
$ export SGI_DBUSER=system
$ export SGI_DBPW=oracle
$ startup.sh
$ open http://localhost:8080
$ shutdown.sh
```

---
### Docker Container

```sh
$ vi docker/tomcat8-dockerfile
$ vi docker/ROOT.xml
$ docker build -t sgigw_app:1.0.1 -f docker/tomcat8-dockerfile .
$ docker run -d -p 8080:8080 --name sgigw_app --link sgigw_db:sgigw_db \
-e "SGI_JDBC_URL=jdbc:oracle:thin:@sgigw_db:1521:xe" \
-e "SGI_DBUSER=system" -e "SGI_DBPW=oracle" sgigw_app:1.0.1

$ docker exec -it sgigw_app bash
$ open http://localhost:8080
$ docker container stop $(docker container ls -q)
$ docker container rm $(docker container ls -aq)
``` 

---
### Docker Composer

```sh
$ vi docker/nginx-dockerfile
$ vi docker/nginx.conf
$ vi docker-compose.yml
$ docker image ls | grep sgigw_
$ docker-compose build
$ docker-compose up -d --scale sgigw_app=2
$ docker exec -it sgigatewayv1_sgigw_app_1 bash # tail -f logs/localhost_access
$ docker exec -it sgigatewayv1_sgigw_app_2 bash # tail -f logs/localhost_access
$ docker-compose down 
```

---
### Push Docker image

```sh
$ docker tag sgigw_db:1.0.2 credemol/sgigw_db:1.0.2
$ docker tag sgigw_app:1.0.2 credemol/sgigw_app:1.0.2
$ docker login
$ docker push credemol/sgigw_db:1.0.2
$ docker push credemol/sgigw_app:1.0.2
$ open https://hub.docker.com/r/credemol/sgigw_app/
```

---
### Kubernetes 

```sh
$ minikube status
$ minikube start
$ vi docker-compose-v3.yml
$ kompose convert -f docker-compose-v3.yml
$ vi sgigw-deployment.yaml
$ vi sgigw-service.yaml
$ kubectl get all --show-labels
$ kubectl delete all --selector "io.kompose.service=sgigw-db"
$ kubectl delete all --selector "io.kompose.service=sgigw-app"

```

---
### Kubernetes DB (Deployment)

```sh
$ kubectl create -f sgigw-db-deployment.yaml
$ kubectl get all
$ kubectl create -f sgigw-db-service.yaml
$ kubectl get svc
$ kubectl describe svc sgigw-db
```

---
### Kubernetes (Deployment)

```sh
$ vi sgigw-app-deployment.yaml # modify SGI_JDBC_URL, minikube service sgigw-db --url
$ kubectl create -f sgigw-app-deployment.yaml
$ kubectl get all
```

---
### Kubernetes (Service)

```sh
$ kubectl create -f sgigw-app-service.yaml
$ kubectl get all
$ kubectl get svc 
$ kubectl describe svc sgigw-app
$ minikube service sgigw-app --url
$ open "$(minikube service sgigw-app --url)"
```

---
### Kubernetes (Scale Out/In)

```sh
$ kubectl get deployments
$ kubectl scale --replicas=4 deployment sgigw-app
$ kubectl get deployments 
$ kubectl get pods
$ kubectl describe svc sgigw-app
```

---
### Kubernetes (Delete resources)

```sh
$ kubectl get all --show-labels
$ kubectl delete all --selector='io.kompose.service=sgigw-app'
$ kubectl delete svc sgigw-app
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

### Kubernetes ConfigMap & Secret

```sh 
$ vi sgigw-configmap.yaml
$ kubectl get configmap
$ kubectl create -f sgigw-configmap.yaml
$ kubectl describe configmap sgigw-config
$ echo -n "oracle" | base64
$ vi sgigw-secret.yaml
$ kubectl create -f sgigw-secret.yaml
$ kubectl get secret
$ kubectl describe secret sgigw-secret
$ rm sgigw-secret.yaml
```

---
### Kubernetes apply Values form ConfigMap & Secret

```yaml
      - env:
        - name: SGI_DBPW
          valueFrom:
            secretKeyRef:
              name: sgigw-secret
              key: sgigw.dbpw
        - name: SGI_DBUSER
          valueFrom:
            configMapKeyRef:
              name: sgigw-config
              key: sgigw.dbuser 
        - name: SGI_JDBC_URL
          valueFrom:
            configMapKeyRef:
              name: sgigw-config
              key: sgigw.jdbcurl
```

---
### Redeploy sgigw-app

```sh
$ kubectl get all --show-labels
$ delete all --selector "io.kompose.service=sgigw-app"
# modify sgigw-app-deployment.yaml
$ kubectl create -f sgigw-app-deployment.yaml
$ open $(minikube service sgigw-app --url)
$ kubectl get pods
$ kubectl get pod $sgigw_app_pod_id
```

## Spring Boot & API First Strategy