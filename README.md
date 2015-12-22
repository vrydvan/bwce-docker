##Caution
Fon internal use only. **Must not be shared outside BW engineering**.

##Prerequiste
Install [Docker Engine](https://docs.docker.com/engine/installation) , [Docker Machine](https://docs.docker.com/machine/install-machine).

##Create BWCE base docker Image
1. Clone this repo
2. Download modified **bwce.zip** from https://drive.google.com/open?id=0B-tPKrxN5XKzRzlFTjJQcXYxQWM and copy it to _/resources/bwce-runtime_ folder. You need access permission to download it. Contact vnalawad@tibco.com for access permission.
2. Build docker image from repo folder e.g. 
 	_docker build -f [[Dockerfile-Ubuntu](Dockerfile-Ubuntu) or [Dockerfile](Dockerfile)] -t **tibco/bwce:v1.1.0** ._
3. BWCE base docker image size:
	* With Ubuntu : ~405 MB
	* With Alpine: 207.4 MB
4. Run BWCE application
	* In Local Environment: In local enviornment, run BWCE application by mapping volume containing ear file to / volume in the container
		e.g.  _docker run --name BWRESTAPP -d **-v /Users/vnalawad/docker-apps/testrest_1.0.0.ear:/testrest_1.0.0.ear** -p 18080:8080 -p 17777:7777 **tibco/bwce:v1.1.0**_. [See docker for more info](https://docs.docker.com/engine/userguide/dockervolumes)
	* Build Application docker image: To run application on docker based PAAS platforms, create application docker image. 
		* Build Image: Create application Dockerfile and ear [[Example](examples/HTTP)] and build image. _Ensure that application docker image is created using BWCE base image [[Example](examples/HTTP/Dockerfile)]._ e.g. **docker build -t docker.http.application:1.0 .**. 
		* Run application using application image e.g.  _docker run -e MESSAGE='Welcome to BWCE' --log-driver=syslog --log-opt syslog-address=udp://logs3.papertrailapp.com:39293 -d -P docker.http.application:1.0_

##Supported Features
* **Application configuration**: 
	* **Using Environment Variables** - BWCE application can be configured with env variables. Refer environment var using #ENV-VAR-NAME# token in the application profile. Only supported for default application profile (default.substvar). e.g. _docker run --name BWHTTPAPP **-e MESSAGE='BWCE Rocks on Docker'** -d -v /Users/vnalawad/docker-apps/docker.http.application_1.0.0.ear:/docker.http.application_1.0.0.ear -p 18081:8080 tibco/bwce:v1.1.0_
	*  **Consul Configuration Managment Service** -  BWCE applications can be configured with keys from the Consul. Refer key using #KEY-NAME# token in the application profile. e.g. for key /dev/MESSAGE use #MESSAGE#. Set **`APP_CONFIG_PROFILE_NAME`** env variable to configure profile or root folder name e.g **APP_CONFIG_PROFILE_NAME=dev**. 
		* Consul server running outside of docker - To use configurations from Consul server running outside of docker, set `CONSUL_SERVER_URI` env variable. e.g. -e CONSUL_SERVER_URI=http://10.97.201.21:8050
		* Consule server running as docker container - To use configurations from Consul server running as docker container, link consul docker container to your BWCE application container as --link `<Consule-container-Name>:consuleserver`. Note that alias name in the link must be **consuleserver**. The default port is set to 8050. To change consul server port, set `CONSUL_SERVER_PORT` env variable. e.g. _docker run --name Dev-BWHttpApp -e APP_CONFIG_PROFILE_NAME=dev --link consul-server:consulserver --log-driver=syslog --log-opt syslog-address=udp://logs3.papertrailapp.com:39293 -d -v /Users/vnalawad/docker-apps/docker.http.application_1.0.0.ear:/docker.http.application_1.0.0.ear -p 18081:8080 tibco/bwce-alpine_
* **Logging using Papertrail**: Run your BWCE application with  _--log-driver=syslog --log-opt syslog-address=udp://{your-papertrail-log-destination}_  e.g. docker run --name BWRESTAPP  **--log-driver=syslog --log-opt syslog-address=udp://logs3.papertrailapp.com:11111** -d -v /Users/vnalawad/docker-apps/testrest_1.0.0.ear:/testrest_1.0.0.ear -p 18080:8080 -p 17777:7777 tibco/bwce:v1.1.0. [More Options](http://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-docker)

