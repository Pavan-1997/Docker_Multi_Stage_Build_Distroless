# Docker Multi Stage Build Distroless   

The Docker Multi-Stage buld is implemented by Splitting the docker file into two parts - 
```
STAGE1:		FROM Ubuntu
			RUN ---

STAGE2: 	(Copying the Artifact binary file from Stage1 to Stage2) 
			(In Stage2 choosing a very minimal image like python runtime or JRE or distroless image)
			CMD [---]
```
We exclude the build we require only run time by dividing into stages

---
# Consider a 3-tier application (Frontend - React, Backend - Java, DB - MySQL)

* We can't choose run time in Stage1 because later it may cause issues in installing other dependencies

BEFORE MULTISTAGE DOCKER BUILD (Final docker image size goes upto 1.5GB) :
```
FROM Ubuntu ---> 400MB
Install dependencies for Java ---> 50MB
Install dependencies for React ---> 100MB
Install dependencies for MySQL ---> 100MB
Build JAVA 
Build Frontend
Combined Entrypoint [/app.ear]			
```

AFTER MULTISTAGE DOCKER BUILD:
```
STAGE1:		FROM Ubuntu as Build ---> 400MB
			Install dependencies for Java ---> 50MB
			Install dependencies for React ---> 100MB
			Install dependencies for MySQL ---> 100MB
			Build JAVA 
			Build Frontend

STAGE2: 	FROM openjdk11 ---> 100MB
			copy --from Build --> 50MB
			Entrypoint [/app.ear] 		

* So the final image size would be 150MB

* There an be countess stages in Docker MultiStage file, but there will be only one final stage which will be a minimal image
```

---

# DISTROLESS DOCKER IMAGE:

- "Distroless" images contain only your application and its runtime dependencies. They do not contain package managers, shells or any other programs you would expect to find in a standard Linux distribution.

- Its ia minimal image that will hardly have any packages

- Golang is a statically typed application which don't require a go run time to execute a Go application, which eventually reduces the docker image to 15MB of size. This image may not allow us to perform basic shell commands (find, wget, curl, ls)

- Distroless images provides highest security by making the image less vulnerable

- OpenJDK is a distroless image which lead to 100MB 

---
# Implementation 

1. Now goto EC2 from AWS Console -> Click on Launch instance

	Give a name 
	
	Use Ubuntu as an image
	
	Instance type as t2.micro
	
	Create a key pair if already present use existing one
	
	Click on Launch instance
	
	Connect to the instance


2. Install Docker
```
sudo apt update -y

sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

apt-cache policy docker-ce

sudo apt install docker-ce

sudo usermod -aG docker $USER

sudo systemctl status docker 
```


3. Install Go
```
sudo apt update -y

sudo apt install golang
```


4. Clone my repository
```
git clone https://github.com/Pavan-1997/Docker_Multi_Stage_Build_Distroless.git
```


5. Run the calculator application using Go by going inside the repo folder Golang_Docker_MSB_Distroless
```
go run calculator.go
```


6. Go inside the repo folder dockerfile-without-multistage and create a docker image without Docker Multi Stage Build
```
sudo docker build -t simplecalculator .
```
  * ubi-minimal is lighter than Ubuntu or you can use a golang image but doesn't suffice the need for Docker MultiStage


7. Check the docker image size that is created which is of size `863MB`
```
sudo docker image ls
```
![image](https://github.com/Pavan-1997/Docker_Multi_Stage_Build_Distroless/assets/32020205/13e5bff7-fdfa-4757-a77c-55093b3b06d9)


8. Go outside the repo folder dockerfile-without-multistage and create a docker image with Docker Multi Stage Build
```
sudo docker build -t simplecalculator_dmss .
```
  * Scratch is a minimal distroless image, Golang runs on this image since it doesn't require a run time but Python will fail because it requires a run time - eventually we have install python on top of this image


9. Check the docker image size that is created which is of size `1.83MB`, which has significantly reduced the size
```
sudo docker image ls
```
![image](https://github.com/Pavan-1997/Docker_Multi_Stage_Build_Distroless/assets/32020205/3ee48ba0-5d7e-4b0f-8dfd-c3ed2b8b097b)
