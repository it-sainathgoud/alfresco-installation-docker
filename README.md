# Install Alfresco Content Services using Docker Compose
Installing ACS using Docker Compose is best option available to do POC, Demos or to explore and deploy any integrated solutions into Alfresco such as custom SAML. Due to the limited capabilities of Docker Compose, this deployment method is recommended for development and test environments only. Lets see how easy it is to install ACS 7.0 solution with the supplied Docker Compose file. 

## Installation
To get the Docker Compose file, navigate to Alfresco website at https://www.alfresco.com/platform/content-services-ecm/trial/download and add your details to get temporary Docker Compose.yml file.  

* Install Docker and start it
* Download the Alfresco docker-compose file
* Open a command prompt or terminal window
* Navigate to the folder where you saved the docker-compose.yml file
* Issue the following three commands:

```
docker login quay.io -u="alfresco+acs_v6_trial"

Enter the following password when prompted (Copy and Paste):
MDF9RNGUJPKZ83KK8UVGUVWO9AYKUZ0VN6WG5VOOCUT6BX19JJLU5ZL0HKU7N20C

docker-compose up
```

* ACS services will be available at http://localhost:8080/alfresco. Your default username/password is: admin/admin
* Alfresco Share will be available at http://localhost/share. 
* Alfresco REST API Explorer will be available at http://localhost/api-explorer. 
* Alfresco Search Services will be available at http://localhost/solr.  

## Deploying additional addons
If you want to deploy additional addons, use deployment folders for Alfresco and Share services.

**Alfresco**
```
├── alfresco
│   ├── modules             > Deployment directory for addons
│   │   ├── amps            > Repository addons with AMP format
│   │   └── jars            > Repository addons with JAR format
```

**Share**
```
└── share                   
    └── modules             > Deployment directory for addons
        ├── amps            > Share addons with AMP format
        └── jars            > Share addons with JAR format
```

## Using Docker Compose
Once the files have been generated, review that configuration is what you expected and add or modify any other settings. After that, just start Docker Compose.

```
$ docker-compose up --build --force-recreate -d

You can shutdown it at any moment using following command.
$ docker-compose down


Alternatively if you choose to apply the start script, you can start the deployment with
./start.sh

It will wait until alfresco is reachable and shutdown with
./start.sh -d


More options are available with.
./start.sh -h
```


## Creating a custom Alfresco repository image (Dockerfile)

```
FROM quay.io/alfresco/alfresco-content-repository:7.0.0

ARG TOMCAT_DIR=/usr/local/tomcat

USER root

## Customize
RUN mkdir ${TOMCAT_DIR}/saml-keystore
RUN chown -R root:Alfresco ${TOMCAT_DIR}/saml-keystore
ADD --chown=root:Alfresco ./saml-keystore ${TOMCAT_DIR}/saml-keystore/
COPY --chown=root:Alfresco alfresco-global.properties ${TOMCAT_DIR}/shared/classes/

## Apply Alfresco SAML customization 
ADD ./saml/alfresco-saml-repo-1.2.1.amp ${TOMCAT_DIR}/amps/

RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
     ${TOMCAT_DIR}/amps ${TOMCAT_DIR}/webapps/alfresco -directory -nobackup -verbose

USER alfresco

## Build
docker build whopper-alfresco -t alfresco/whopper-alfresco:7.0.0 -t alfresco/whopper-alfresco:latest
```

## Creating a custom Alfresco Share image

```
#Dockerfile - alfresco
FROM quay.io/alfresco/alfresco-share:7.0.0

ARG TOMCAT_DIR=/usr/local/tomcat

# Apply Share SAML customization
ADD ./saml/alfresco-saml-share-1.2.1.amp ${TOMCAT_DIR}/amps_share/

RUN java -jar ${TOMCAT_DIR}/alfresco-mmt/alfresco-mmt*.jar install \
     ${TOMCAT_DIR}/amps_share ${TOMCAT_DIR}/webapps/share -directory -nobackup -verbose

#Build
docker build whopper-share -t alfresco/whopper-share:7.0.0 -t alfresco/whopper-share:latest

```