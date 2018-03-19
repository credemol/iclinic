# Demo for iClinic

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
1. docker-maven-plugin (build & push)

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
$ vi sgigw-app-deployment.yaml 
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

---
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

---
## Spring Boot & API First Strategy

---
### Setting up

```sh
$ vi docker-compose-mysqlonly.yaml
$ docker-compose -f docker-compose-mysqlonly.yaml build
$ docker image ls | grep iclinic
$ docker-compose -f docker-compose-mysqlonly.yaml up
```

---
### Create Spring Boot Application

project name: iclinic-demo
dependencies:
  - SQL/JPA
  - SQL/MySQL
  - Web/Web
  - Web/Rest Repositories


---
### Sample Entity class

Package Name: com.example.demo.entity
Class Name: Sample 

```java
	/** 아이디 */
	private String id;
	/** 이름 */
	private String name;
	/** 내용 */
	private String description;
	/** 사용여부 */
	private String useYn;
	/** 등록자 */
	private String regUser;  
```    

---
### SampleRepository interface

Package Name: com.example.demo.repo
Class Name: SampleRepository

```java
@RepositoryRestController
public interface SampleRepository extends CrudRepository<Sample, String>{

}
```

---
### Run REST API

```sh 
$ mvn clean package
$ mvn run spring-boot:run

```

---
### Swagger - pom.xml

```xml
		<dependency>
			<groupId>io.springfox</groupId>
			<artifactId>springfox-swagger2</artifactId>
			<version>2.6.1</version>
		</dependency>
    <dependency>
      <groupId>io.springfox</groupId>
      <artifactId>springfox-swagger-ui</artifactId>
      <version>2.6.1</version>
    </dependency>
```

---
### Swagger - Add EnableSwagger2

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import springfox.documentation.swagger2.annotations.EnableSwagger2;

@SpringBootApplication
@EnableSwagger2
public class IclinicDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(IclinicDemoApplication.class, args);
	}
}
```
@[9](Add EnableSwagger2 annotation)

---
### Swagger - SampleRestController class

Package: com.example.demo.rest;

```java
@RequestMapping("/rest/samples")
@RestController
public class SampleService {

	@Autowired
	SampleRepository repository;
	
	
}
``` 

---
### Swagger - GetAll

```java
	@RequestMapping(value="", method=RequestMethod.GET,
			produces= {MediaType.APPLICATION_JSON_VALUE})
	public ResponseEntity<List<Sample>> getAllSamples() {
		List<Sample> samples = new ArrayList<>();
		
		repository.findAll().iterator().forEachRemaining(samples::add);

		return new ResponseEntity<List<Sample>>(samples, HttpStatus.OK);
	}
```

---
### Swagger - AddSample

```java
	@RequestMapping(value="", method=RequestMethod.POST, 
			produces= {MediaType.APPLICATION_JSON_VALUE},
			consumes= {MediaType.APPLICATION_JSON_VALUE})
	public ResponseEntity<Sample> addSample(@RequestBody Sample sample) {
		if(repository.findById(sample.getId()).isPresent()) {
			return new ResponseEntity<Sample>(HttpStatus.CONFLICT);
		}
		
		repository.save(sample);
		
		return new ResponseEntity<Sample>(sample, HttpStatus.OK);
	}
```

---
### Swagger - GetById

```java
	@RequestMapping(value="{id}", method=RequestMethod.GET,
			produces= {MediaType.APPLICATION_JSON_VALUE})
	public ResponseEntity<Sample> getById(@PathVariable("id") String id) {
		Optional<Sample> sample = repository.findById(id);
		
		if(sample.isPresent()) {
			return new ResponseEntity<Sample>(sample.get(), HttpStatus.OK);
		}
		return new ResponseEntity<Sample>(HttpStatus.NOT_FOUND);
	}
```

---
### Run

```sh
$ mvn clean package
$ mvn spring-boot:run
$ open http://localhost:8080/swagger-ui.html
$ open http://localhost:8080/v2/api-docs
```

---
### Swagger Codegen(Client: 32) - Java

```sh
$ brew install swagger-codegen
$ mkdir ../codegen
$ cd ../codegen
$ curl -o swagger.json http://localhost:8080/v2/api-docs
$ swagger-codegen generate -v -i swagger.json -l "java" \
  -o "java-client" --group-id "com.iclinicemr.api" \
  --artifact-id "sampleapi" \
  --invoker-package "com.iclinicemr.client" \
  --api-package "com.iclinicemr.api" \
  --model-package "com.iclinicemr.model"

$ code java-client
```

---
### Test

- com.iclinicemr.client.ApiClient: line 115 https -> http
- com.iclinicemr.api.SampleRestControllerApiTest
  - //@Ignore
  - // comment addSample
  - System.out.println("all samples: " + response);
  - String id = "SAMPLE-00001";
  - System.out.println("sample: " + response);

```sh
$ mvn test
```

---
### Swagger Codegen(Server: 23): SpringBoot

```sh
$ swagger-codegen generate -v -i swagger.json -l "spring" \
 -o "springboot-api" --group-id "com.iclinicemr.api" \
 --artifact-id "sampleapi" --api-package "com.iclinicemr.api" \
 --model-package "com.iclinicemr.model" --model-name-suffix "VO"

$ code springboot-api
$ mvn spring-boot:run -Dserver.port=9090
$ open http://localhost:9090/swagger-ui.html
```

---
### API Transformation

```sh
$ open https://apimatic.io/
$ open https://apimatic.io/transformer
```

- Convert: swagger.json --> swagger.yaml 
- Convert: swagger.json --> Postman.json

---
### docker-maven-plugin

[https://github.com/spotify/docker-maven-plugin](https://github.com/spotify/docker-maven-plugin)

---
### ~/.m2/settings.xml
```sh
$ mvn --encrypt-password your-dockerhub-password
```
```xml
  <server>
    <id>docker-hub</id>
    <username>credemol</username>
    <password>{GSxBOfHdHTkHMlOpe/BhqTLMb1zLfz86xXoZxRmZng0=}</password>
    <configuration>
      <email>credemol@gmail.com</email>
    </configuration>
  </server>
```

---
### pom.xml

```xml
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.11</version>
                <configuration>
                
			      <serverId>docker-hub</serverId>
			      <registryUrl>https://index.docker.io/v1/</registryUrl>
                 
                    <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                    <dockerDirectory>src/main/docker</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>	
```
@[7-8](docker repository information-docker hub)
@[11](Dockerfile location)

---
### src/main/docker/Dockerfile

```dockerfile
FROM frolvlad/alpine-oraclejdk8:slim

ADD iclinic-demo-0.0.1-SNAPSHOT.jar app.jar
RUN sh -c 'touch /app.jar'
ENV JAVA_OPTS=""
ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]
```

---
### Build Docker Image and Push it to Docker hub

```sh
$ mvn clean package docker:build -DpushImage
$ docker image ls | grep credemol/iclinic-demo
$ open https://hub.docker.com/r/credemol/iclinic-demo/
```