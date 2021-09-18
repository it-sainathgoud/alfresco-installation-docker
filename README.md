# Install Alfresco Content Services using Docker Compose
Installing ACS using Docker Compose is best option available to do POC, Demos or to explore and deploy any integrated solutions into Alfresco such as custom SAML. Due to the limited capabilities of Docker Compose, this deployment method is recommended for development and test environments only. Lets see how easy it is to install ACS 7.0 solution with the supplied Docker Compose file. 

## Getting Started
* Install using Docker Compose
* Alfresco Cotent Service URL's
* Using Docker Compose
* Inspecting Docker Compose  
* Deploying additional addons
* Customizing the Docker Compose with Custom Port #
* Creating a Custom Alfresco Repository Image
* Creating a Custom Alfresco Share Image

## Install using Docker Compose
* Install Docker and start it
* To get the docker compose file, navigate to Alfresco website at https://www.alfresco.com/platform/content-services-ecm/trial/download and add your details to get Docker Compose.yml file. Once you received, execute the below steps. 
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

## Alfresco Cotent Service URL's

* ACS services will be available at http://localhost:8080/alfresco. Your default username/password is: admin/admin
* Alfresco Share will be available at http://localhost/share. 
* Alfresco REST API Explorer will be available at http://localhost/api-explorer. 
* Alfresco Search Services will be available at http://localhost/solr.  

## Using Docker Compose
Once the files have been generated, review that configuration is what you expected and add or modify any other settings. After that, just start Docker Compose.

```
# List the images and additional details:
docker-compose images

# List the running containers:
docker-compose ps

# Connect to docker image => alfresco-repo
docker exec -u 0 -t -i docker_alfresco_1 /bin/bash
docker exec -u 0 -t -i docker_share_1 /bin/bash

# View the log files for each service <service-name>, or container <container-name>:
docker-compose logs share
docker container logs docker-compose_share_1

docker-compose logs --tail=25 share
docker container logs --tail=25 docker-compose_share_1

# Check for a success message:
Successfully retrieved license information from Alfresco.

# Stop all the running containers:
docker-compose stop

# Restart the containers (after using the stop command):
docker-compose restart

# Starts the containers that were started with docker-compose up:
docker-compose start

# Stop all running containers, and remove them and the network:
docker-compose down [--rmi all]

The --rmi all option also removes the images created by docker-compose up, and the images used by any service. You can use this, for example, if any containers fail and you need to remove them:
 Stopping docker-compose_transform-router_1 ... done
 ...
 Removing docker-compose_transform-router_1 ... done
 ...
 Removing network docker-compose_default
 Removing image alfresco/alfresco-content-repository:6.1.1
 Removing image ...

# Remove Files from docker container
rm -rf [filename]

#Retrieve Logs
docker logs docker_alfresco_1 > docker_alfresco_1
docker logs docker_share_1 > docker_share_1
```

## Inspecting Docker Compose  
We have seen how easy it is to deploy and run the ACS system with Docker Compose, next we will look at the Compose YAML file that made it happen.

```
services:
    alfresco:
        image: quay.io/alfresco/alfresco-content-repository:7.0.0
        mem_limit: 1700m
        environment:
            JAVA_TOOL_OPTIONS: " -Dencryption.keystore.type=JCEKS -Dencryption.cipherAlgorithm=DESede/CBC/PKCS5Padding -Dencryption.keyAlgorithm=DESede -Dencryption.keystore.location=/usr/local/tomcat/shared/classes/alfresco/extension/keystore/keystore -Dmetadata-keystore.password=mp6yc0UD9e -Dmetadata-keystore.aliases=metadata -Dmetadata-keystore.metadata.password=oKIWzVdEdA -Dmetadata-keystore.metadata.algorithm=DESede "
            JAVA_OPTS: "
                -Ddb.driver=org.postgresql.Driver
                -Ddb.username=alfresco
                -Ddb.password=alfresco
                -Ddb.url=jdbc:postgresql://postgres:5432/alfresco
                -Dsolr.host=solr6
                -Dsolr.port=8983
                -Dsolr.secureComms=none
                -Dsolr.base.url=/solr
                -Dindex.subsystem.name=solr6
                -Dshare.host=127.0.0.1
                -Dshare.port=8080
                -Dalfresco.host=localhost
                -Dalfresco.port=8080
                -Daos.baseUrlOverwrite=http://localhost:8080/alfresco/aos
                -Dmessaging.broker.url=\"failover:(nio://activemq:61616)?timeout=3000&jms.useCompression=true\"
                -Ddeployment.method=DOCKER_COMPOSE
                -Dtransform.service.enabled=true
                -Dtransform.service.url=http://transform-router:8095
                -Dsfs.url=http://shared-file-store:8099/
                -DlocalTransform.core-aio.url=http://transform-core-aio:8090/
                -Dcsrf.filter.enabled=false
                -Ddsync.service.uris=http://localhost:9090/alfresco
                -DtrialUid=id18104004
                -XX:MinRAMPercentage=50
                -XX:MaxRAMPercentage=80
                "

    transform-router:
        mem_limit: 512m
        image: quay.io/alfresco/alfresco-transform-router:1.3.2
        environment:
            JAVA_OPTS: " -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80"
            ACTIVEMQ_URL: "nio://activemq:61616"

            CORE_AIO_URL: "http://transform-core-aio:8090"

            FILE_STORE_URL: "http://shared-file-store:8099/alfresco/api/-default-/private/sfs/versions/1/file"
        ports:
            - 8095:8095
        links:
            - activemq

    transform-core-aio:
        image: alfresco/alfresco-transform-core-aio:2.3.10
        mem_limit: 1536m
        environment:
            JAVA_OPTS: " -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80"
            ACTIVEMQ_URL: "nio://activemq:61616"
            FILE_STORE_URL: "http://shared-file-store:8099/alfresco/api/-default-/private/sfs/versions/1/file"
        ports:
            - 8090:8090
        links:
            - activemq

    shared-file-store:
        image: quay.io/alfresco/alfresco-shared-file-store:0.13.0
        mem_limit: 512m
        environment:
            JAVA_OPTS: " -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80"
            scheduler.content.age.millis: 86400000
            scheduler.cleanup.interval: 86400000
        ports:
            - 8099:8099
        volumes:
            - shared-file-store-volume:/tmp/Alfresco/sfs

    share:
        image: quay.io/alfresco/alfresco-share:7.0.0
        mem_limit: 1g
        environment:
            REPO_HOST: "alfresco"
            REPO_PORT: "8080"
            JAVA_OPTS: " -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80 -Dalfresco.host=localhost -Dalfresco.port=8080 -Dalfresco.context=alfresco -Dalfresco.protocol=http "

    postgres:
        image: postgres:13.1
        mem_limit: 512m
        environment:
            - POSTGRES_PASSWORD=alfresco
            - POSTGRES_USER=alfresco
            - POSTGRES_DB=alfresco
        command: postgres -c max_connections=300 -c log_min_messages=LOG
        ports:
            - 5432:5432

    solr6:
        image: alfresco/alfresco-search-services:2.0.1
        mem_limit: 2g
        environment:
            #Solr needs to know how to register itself with Alfresco
                - SOLR_ALFRESCO_HOST=alfresco
                - SOLR_ALFRESCO_PORT=8080
            #Alfresco needs to know how to call solr
                - SOLR_SOLR_HOST=solr6
                - SOLR_SOLR_PORT=8983
            #Create the default alfresco and archive cores
                - SOLR_CREATE_ALFRESCO_DEFAULTS=alfresco,archive
            #HTTP by default
                - ALFRESCO_SECURE_COMMS=none
        ports:
            - 8083:8983 #Browser port

    activemq:
        image: alfresco/alfresco-activemq:5.16.1
        mem_limit: 1g
        ports:
            - 8161:8161 # Web Console
            - 5672:5672 # AMQP
            - 61616:61616 # OpenWire
            - 61613:61613 # STOMP

    digital-workspace:
        image: quay.io/alfresco/alfresco-digital-workspace:2.1.0-adw
        mem_limit: 128m
        environment:
            APP_CONFIG_AUTH_TYPE: "BASIC"
            BASE_PATH: ./

    proxy:
        image: alfresco/alfresco-acs-nginx:3.1.1
        mem_limit: 128m
        depends_on:
            - alfresco
            - digital-workspace
        ports:
            - 8080:8080
        links:
            - digital-workspace
            - alfresco
            - share

    sync-service:
        image: quay.io/alfresco/service-sync:3.4.0
        mem_limit: 1g
        environment:
            JAVA_OPTS: " -Dsql.db.driver=org.postgresql.Driver -Dsql.db.url=jdbc:postgresql://postgres:5432/alfresco -Dsql.db.username=alfresco -Dsql.db.password=alfresco -Dmessaging.broker.host=activemq -Drepo.hostname=alfresco -Drepo.port=8080 -Ddw.server.applicationConnectors[0].type=http -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80 "

        ports:
            - 9090:9090

volumes:
    shared-file-store-volume:
        driver_opts:
            type: tmpfs
            device: tmpfs

```

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


## Customizing the Docker Compose with Custom Port #
If you want to change the default port to custom port then you would need to bring down your existing docker alfresco followed by duplicating the docker-compose.yml with custom changes such as below with port # you wish to change followed by navigate to the folder where you saved the custom-docker-compose.yml and enter command docker compose up. 

```
services:
    alfresco:
        image: alfresco/whopper-alfresco:7.0.0
        mem_limit: 1700m
        environment:
            JAVA_TOOL_OPTIONS: " -Dencryption.keystore.type=JCEKS -Dencryption.cipherAlgorithm=DESede/CBC/PKCS5Padding -Dencryption.keyAlgorithm=DESede -Dencryption.keystore.location=/usr/local/tomcat/shared/classes/alfresco/extension/keystore/keystore -Dmetadata-keystore.password=mp6yc0UD9e -Dmetadata-keystore.aliases=metadata -Dmetadata-keystore.metadata.password=oKIWzVdEdA -Dmetadata-keystore.metadata.algorithm=DESede "
            JAVA_OPTS: "
                -Ddb.driver=org.postgresql.Driver
                -Ddb.username=alfresco
                -Ddb.password=alfresco
                -Ddb.url=jdbc:postgresql://postgres:5432/alfresco
                -Dsolr.host=solr6
                -Dsolr.port=8983
                -Dsolr.secureComms=none
                -Dsolr.base.url=/solr
                -Dindex.subsystem.name=solr6
                -Dshare.host=127.0.0.1
                -Dshare.port=8080
                -Dalfresco.host=localhost
                -Dalfresco.port=8080
                -Daos.baseUrlOverwrite=http://localhost:8080/alfresco/aos
                -Dmessaging.broker.url=\"failover:(nio://activemq:61616)?timeout=3000&jms.useCompression=true\"
                -Ddeployment.method=DOCKER_COMPOSE
                -Dtransform.service.enabled=true
                -Dtransform.service.url=http://transform-router:8095
                -Dsfs.url=http://shared-file-store:8099/
                -DlocalTransform.core-aio.url=http://transform-core-aio:8090/
                -Dcsrf.filter.enabled=false
                -Ddsync.service.uris=http://localhost:9090/alfresco
                -DtrialUid=id18104004
                -XX:MinRAMPercentage=50
                -XX:MaxRAMPercentage=80
                "

    transform-router:
        mem_limit: 512m
        image: quay.io/alfresco/alfresco-transform-router:1.3.2
        environment:
            JAVA_OPTS: " -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80"
            ACTIVEMQ_URL: "nio://activemq:61616"

            CORE_AIO_URL: "http://transform-core-aio:8090"

            FILE_STORE_URL: "http://shared-file-store:8099/alfresco/api/-default-/private/sfs/versions/1/file"
        ports:
            - 8095:8095
        links:
            - activemq

    transform-core-aio:
        image: alfresco/alfresco-transform-core-aio:2.3.10
        mem_limit: 1536m
        environment:
            JAVA_OPTS: " -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80"
            ACTIVEMQ_URL: "nio://activemq:61616"
            FILE_STORE_URL: "http://shared-file-store:8099/alfresco/api/-default-/private/sfs/versions/1/file"
        ports:
            - 8090:8090
        links:
            - activemq

    shared-file-store:
        image: quay.io/alfresco/alfresco-shared-file-store:0.13.0
        mem_limit: 512m
        environment:
            JAVA_OPTS: " -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80"
            scheduler.content.age.millis: 86400000
            scheduler.cleanup.interval: 86400000
        ports:
            - 8099:8099
        volumes:
            - shared-file-store-volume:/tmp/Alfresco/sfs

    share:
        image: alfresco/whopper-share:7.0.0 
        mem_limit: 1g
        environment:
            REPO_HOST: "alfresco"
            REPO_PORT: "8080"
            JAVA_OPTS: " -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80 -Dalfresco.host=localhost -Dalfresco.port=8080 -Dalfresco.context=alfresco -Dalfresco.protocol=http "

    postgres:
        image: postgres:13.1
        mem_limit: 512m
        environment:
            - POSTGRES_PASSWORD=alfresco
            - POSTGRES_USER=alfresco
            - POSTGRES_DB=alfresco
        command: postgres -c max_connections=300 -c log_min_messages=LOG
        ports:
            - 5432:5432

    solr6:
        image: alfresco/alfresco-search-services:2.0.1
        mem_limit: 2g
        environment:
            #Solr needs to know how to register itself with Alfresco
                - SOLR_ALFRESCO_HOST=alfresco
                - SOLR_ALFRESCO_PORT=8080
            #Alfresco needs to know how to call solr
                - SOLR_SOLR_HOST=solr6
                - SOLR_SOLR_PORT=8983
            #Create the default alfresco and archive cores
                - SOLR_CREATE_ALFRESCO_DEFAULTS=alfresco,archive
            #HTTP by default
                - ALFRESCO_SECURE_COMMS=none
        ports:
            - 8083:8983 #Browser port

    activemq:
        image: alfresco/alfresco-activemq:5.16.1
        mem_limit: 1g
        ports:
            - 8161:8161 # Web Console
            - 5672:5672 # AMQP
            - 61616:61616 # OpenWire
            - 61613:61613 # STOMP

    digital-workspace:
        image: quay.io/alfresco/alfresco-digital-workspace:2.1.0-adw
        mem_limit: 128m
        environment:
            APP_CONFIG_AUTH_TYPE: "BASIC"
            BASE_PATH: ./

    proxy:
        image: alfresco/alfresco-acs-nginx:3.1.1
        mem_limit: 128m
        depends_on:
            - alfresco
            - digital-workspace
        ports:
            - 8080:8080
        links:
            - digital-workspace
            - alfresco
            - share

    sync-service:
        image: quay.io/alfresco/service-sync:3.4.0
        mem_limit: 1g
        environment:
            JAVA_OPTS: " -Dsql.db.driver=org.postgresql.Driver -Dsql.db.url=jdbc:postgresql://postgres:5432/alfresco -Dsql.db.username=alfresco -Dsql.db.password=alfresco -Dmessaging.broker.host=activemq -Drepo.hostname=alfresco -Drepo.port=8080 -Ddw.server.applicationConnectors[0].type=http -XX:MinRAMPercentage=50 -XX:MaxRAMPercentage=80 "

        ports:
            - 9090:9090

volumes:
    shared-file-store-volume:
        driver_opts:
            type: tmpfs
            device: tmpfs

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