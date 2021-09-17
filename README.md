# Install Alfresco Content Services using Docker Compose
Installing ACS using Docker Compose is best option available to do POC, Demos or to explore and deploy any integrated solutions into Alfresco such as custom SAML. Due to the limited capabilities of Docker Compose, this deployment method is recommended for development and test environments only. Lets see how easy it is to install ACS 7.0 solution with the supplied Docker Compose file. 

To get the Docker Compose file, navigate to Alfresco website at [Alfresco] (https://www.alfresco.com/platform/content-services-ecm/trial/download) and add your details to get temporary Docker Compose.yml file with instructions.  

* Install Docker and start it
* Download the Alfresco docker-compose file
* Follow the detailed instructions below


## Docker Images

* [alfresco-content-repository-community:7.0.0](https://hub.docker.com/r/alfresco/alfresco-content-repository-community)

```
docker login quay.io -u="alfresco+acs_v6_trial"

Enter the following password when prompted (Copy and Paste):
MDF9RNGUJPKZ83KK8UVGUVWO9AYKUZ0VN6WG5VOOCUT6BX19JJLU5ZL0HKU7N20C

docker-compose up
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