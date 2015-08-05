#add mongo service
```cf create-service mongodb 100 mongodb01-kyle```


### 8.4.2014 docker containers

## Installation Setup

1.  upgarde docker to 1.6 from my version 1.5.0  ```boot2docker upgrade``` now brings me to version 1.7.1
2. upgrade cf
>cf version 6.10.0-b78bf10-2015-02-11T22:25:45+00:00
3. Went to [github](https://github.com/cloudfoundry/cli/releases) and just grabed the binaries
4. Install the container pluging ```cf install-plugin https://static-ice.ng.bluemix.net/ibm-containers-mac```  you should now see the ic pluggin from ```cf help``` under the ***INSTALLED PLUGIN COMMANDS***  section
5.  ```cf ic login``` now shows my new private bluemix namespace
```
** Authenticating with registry at registry.ng.bluemix.net
Successfully authenticated with registry
Your private Bluemix repository is registry.ng.bluemix.net/bowerman
```
The cf ic login also gives some details on how you can use native docker and just change your registery or use the cf ic plugin

```
There are two ways to use the CLI with IBM Containers:

Option 1) This option allows you to use cf ic for managing containers on IBM Containers while still using the docker CLI directly to manage your local docker host.
	Leverage this Cloud Foundry IBM Containers plugin without affecting the local docker environment:


	Example Usage:
	cf ic ps
	cf ic images

Option 2) Leverage the docker CLI directly. In this shell, override local docker environment to connect to IBM Containers by setting these variables, copy and paste the following:
	Notice: only commands with an asterisk are supported within this option

 	export DOCKER_HOST=tcp://containers-api.ng.bluemix.net:8443
 	export DOCKER_CERT_PATH=/Users/kylebowerman/.ice/certs
 	export DOCKER_TLS_VERIFY=1

	Example Usage:
	docker ps
	docker images

```
## Attempts to build a deploy a custom mongo container to Bluemix

6. Build a mongo image from a local docker file:  
   ```docker build -t ktb_mongo .```
7. Tag the image with your private namespace in the IBM Containers registry.
   ```docker tag ktb_mongo  registry.ng.bluemix.net/bowerman/ktb_mongo:v0.1``` this command basicly clones the image but to a different repo and adds a version via the tag appended to the image name.
8. Now we need to push this image to the IBM Containers registry: ```docker push  registry.ng.bluemix.net/bowerman/ktb_mongo:v0.1 ```  I had to run cf ic login first. but then it worked.
9.  You can create a container from this image in the Bluemix Catalog, or with the following command:  cf ic run --name container_name registry.ng.bluemix.net/bowerman/image_name:image_tag
I ran ```  cf ic run --name ktb_mongo registry.ng.bluemix.net/bowerman/ktb_mongo:v0.1```  crashed after a few minutes, why lets check and see if I had an entry point in my dockerfile ... Yep I did not have an entry point lets try it again.  But first I need to remove the container: ```cf ic  rm eb5197f8-209```  So now lets pass the container entry point into the run command,   we will forward the ports as well as turn on the http service with mongo with the following options

```cf ic run -d -p 27017:21017 -p 28017:28017  --name ktb_mongo_c2 registry.ng.bluemix.net/bowerman/ktb_mongo:v0.1 mongod --rest --httpinterface``` returns the following container id ***527d1655-988*** and I confirmed that it is staying up.

  * -p forwards the default mongo port (27017) and the web interface runs on 28017
  * **mongod** is the executable or 'entry point we are running'
  * -rest turns on the rest api for mongo
  * --httpinterface turns on the mongo web tool

10. Now lest inspect the container by running ```cf ic inspect 527d1655-988```  Ah shit it crashed again.  The run command works localy so I am not sure why it is crashing on bluemix.

 - tried:  ``` cf ic  run   --name ktb_mongo_c2 registry.ng.bluemix.net/bowerman/ktb_mongo:v0.1 mongod```  ***CRASHED***

 - tried: rebuilt Dockerfile with valid entry point.  This might work!
  * ran the build for the new Dockerfile with the entry point and built that image
 ** first I create the new image as mongodwithentrypoint ```docker build -t mongodwithentrypoint .```
 ** now I tag it version 2 ``` docker tag mongodwithentrypoint registry.ng.bluemix.net/bowerman/ktb_mongo:v0.2```
 ** next I push it up to bluemix ``` docker push registry.ng.bluemix.net/bowerman/ktb_mongo:v0.2``` ***CRASHED***

 -tried with ports:
  -- ``` cf ic run -d -p 27017:21017 -p 28017:28017 --name ktb_mongoV0.2.2  registry.ng.bluemix.net/bowerman/ktb_mongo:v0.2``` ***CRASHED***
