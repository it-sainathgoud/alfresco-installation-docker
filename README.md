
# Docker installation

docker login quay.io -u="alfresco+acs_v6_trial"

Enter the following password when prompted (Copy and Paste):
MDF9RNGUJPKZ83KK8UVGUVWO9AYKUZ0VN6WG5VOOCUT6BX19JJLU5ZL0HKU7N20C

docker-compose up

# Creating a custom Alfresco repository image (Dockerfile)

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