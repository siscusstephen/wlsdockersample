#last edited 26/03/2020

# Sample Wls Docker domain creation using wlst.
There are few ways to package a image, either using wlst or using wdt.

In this example we are using wlst, as wlst has been in wls for quite a long time and all features in console can be script in wlst.
We can even record the changes in weblogic console and then use back the recording script (https://docs.oracle.com/middleware/1212/wls/WLACH/taskhelp/console/RecordWLSTScripts.html)
(Refer to references for more details,)

Wls operator Kubernetes make use of dynamic cluster to scale, so in the wlst we are configure dynamic cluster.

## Steps tested on below environment.
- Running windows 10 64 bit.
- Using docker desktop https://docs.docker.com/docker-for-windows/install/
- Do take note that Hyper-V and Containers Windows features must be enabled.
- Go to Start -> Turn windows features on -> Select Hyper-V and Containers.
- Apache Maven 3.6.1 is install to build a sample app.

## Build simple app
- mvn clean package

## Pull weblogic docker image
- Login to docker hub
- Search for weblogic
- Agree to the agreement and test if you could pull down the docker image
- Run command
> docker pull store/oracle/weblogic:12.2.1.4-191221
- Check if image is pull by running
> docker images

## Docker create
- Look at the contents of Dockerfile.create.
- It is pulling the image from docker store, copy the war file and running a wlst script to create a domain.
- We can also include os packages such as vim in the image so you run vi command, or you want to copy certain files into the image etc.
- Examine the contents of model.py wlst, it is creating a domain and deploy certain resources.
- In real life, your wls config will be ideally scripted in wlst. Although you can go into the console make changes and commit to a next version which we will explain below.
- Run command 
> docker build --file Dockerfile.create --force-rm=true -t mywls:v1 .
- Run command. You will see the docker image being created.
> docker images  
- If we do not want to pull the offical docker image from docker hub, we can install using image tool and build and patch your wls install from scrath. The same pricniple will apply and instead of pulling from the store, pull the weblogic binary from your own image that you have build, look at Dockerfile.create line 3.
- Refer to https://github.com/oracle/weblogic-image-tool

## [Optional] Docker update
- If image has been create and just want to update the app run the below command and update the tag to mywls:v2
> docker build --file Dockerfile.update --build-arg SOURCEIMAGE=mywls:v1 --force-rm=true -t mywls:v2 .
                    
## Start the admin server

- Create a file called domain.properties. Refer to domain.properties
- In docker create a shared directory, Eg in windows i place this domain.properties in D:\\1_dockershare\\docker-run
- Refer to https://github.com/wenjian80/wlsdockersample/blob/master/shared_directory.JPG on shared directory in docker dekstop.
- Run command 
> docker run -d -p 7001:7001 -p 9002:9002 --name=wlsadmin --hostname wlsadmin -v D:\\1_dockershare\\docker-run:/u01/oracle/properties -e ADMINISTRATION_PORT_ENABLED=true mywls:v1 /u01/oracle/user_projects/domains/sample-domain1/bin/startWebLogic.sh
- Run command to see if the it is running.
> docker ps -a 
- Access the console http://localhost:7001/console and logic via weblogic/welcome1 as defined in domain.properties
- Run command, example "docker exec -it 37dc159d29f7 bash" to go into the container. 
> docker exec -it [container id when you have run docker ps] bash
- You can make changes to the files system and do a commit to a different version. Example command below.
- For example you can go into the console make some changes and then commit to a next version. This is not recommended as the changes should be done in wlst so every things is run as a script when making a docker container.
> docker commit 37dc159d29f7 mywls:v2
- Stop the container 
> docker stop 37dc159d29f7
- See log of container 
> docker container logs 37dc159d29f7
- Remove container 
>docker container rm  37dc159d29f7


## Start the managed server
- Make sure your admin server is already up and running. Refer to previous steps.
- Create a file boot.properties. Refer to boot.properties
- In docker create a shared directory, Eg in window i place this boot.properties in D:\\1_dockershare\\docker-run\\ms1
- Run command 
> docker run -d --name=managed-server1 --hostname managed-server1 --link wlsadmin:wlsadmin -p 8001:8001 -v D:\\1_dockershare\\docker-run\\ms1:/u01/oracle/user_projects/domains/sample-domain1/servers/managed-server1/security  mywls:v1 /u01/oracle/user_projects/domains/sample-domain1/bin/startManagedWebLogic.sh managed-server1 http://wlsadmin:7001
- Go to weblogic console. http://localhost:7001/console. You will see the managed server is up.
- Try accessing http://localhost:8001/opdemo/ to see the app

## WLS operator [Steps created in July 2019]
- This image can then be used in weblogic operator.
- Refer to https://oracle.github.io/weblogic-kubernetes-operator/ for details.
- Scripts can be found in https://github.com/oracle/weblogic-kubernetes-operator/tree/master/kubernetes/samples/scripts/create-weblogic-domain to either choose domain in image model or pvc.
- Previous labs in https://github.com/wenjian80/kubelab
- Labs are on oracle cloud, using vanilla linux, with kubernetes installed via linux yum package.
- Labs are using domain in image model and th files are in shared directory using oracle shared file system, that can be change to use domain in image pvc.

## References

Weblogic docker image
- https://hub.docker.com/_/oracle-weblogic-server-12c
- https://container-registry.oracle.com
- https://github.com/oracle/docker-images/tree/master/OracleWebLogic

Weblogic operator
- https://oracle.github.io/weblogic-kubernetes-operator/
- https://github.com/oracle/weblogic-kubernetes-operator
- https://github.com/oracle/weblogic-kubernetes-operator/tree/master/kubernetes/samples/scripts/

Monitoring exporter
- https://github.com/oracle/weblogic-monitoring-exporter

Logging exporter
- https://github.com/oracle/weblogic-logging-exporter

Weblogic WDT
- https://github.com/oracle/weblogic-deploy-tooling

Weblogic WLST
- https://docs.oracle.com/middleware/1212/wls/WLACH/taskhelp/console/RecordWLSTScripts.html
- https://docs.oracle.com/middleware/12213/cross/wlsttasks.htm

Weblogic image tool, docker images
- https://github.com/oracle/weblogic-image-tool

White Paper steps
- https://www.oracle.com/a/ocom/docs/engineered-systems/oracle-webLogic-server-on-pca.pdf

Weblogic Tooling
- https://github.com/oracle/weblogic-image-tool
- https://github.com/oracle/weblogic-deploy-tooling

Operator
- https://oracle.github.io/weblogic-kubernetes-operator/userguide/introduction/
- https://github.com/oracle/weblogic-kubernetes-operator

Weblogic exporter for promethues
- https://github.com/oracle/weblogic-monitoring-exporter

Slack support
- oracle-weblogic.slack.com
- https://weblogic-slack-inviter.herokuapp.com/



