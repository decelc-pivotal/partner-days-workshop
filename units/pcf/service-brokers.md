---
pageTitle: Exercise #3 - Service Broker Lab
---

# Introduction

In this exercise, we get hands-on with creating a service broker and binding it to an application.


# Part 1: Catalog / Marketplace

1. Get the code. Download and extract the zip file containing the service broker (code and build) from our slack channel. This broker is implemented with Spring Boot and leverages the Spring Cloud Service Broker project.


2. Catalog (GET v2/catalog). Spring Cloud Service Broker has mapped the /v2/catalog endpoint for us.To see the implementation, open:
    src/main/java/org/springframework/cloud/servicebroker/mongodb/config/CatalogConfig.java
In particular, notice the SERVICE_ID, SERVICE_NAME, and PLAN_ID environment variables are used in the catalog configuration.


3. Build and deploy your service broker. Deploy the service broker as a Cloud Foundry application with the following commands. Be sure to sub in your name for LASTNAME in the commands.
    ```bash
    cd <path-to>/cloudfoundry-mongodb-service-broker
    ./gradlew assemble
    cf push mongodb-service-broker -p build/libs/cloudfoundry-mongodb-service-broker.jar -m 768M --random-route --no-start
    cf set-env mongodb-service-broker SERVICE_ID mongodb-service-broker-LASTNAME
    cf set-env mongodb-service-broker SERVICE_NAME MongoDB-LASTNAME
    cf set-env mongodb-service-broker PLAN_ID mongo-plan-LASTNAME
    cf set-env mongodb-service-broker MONGODB_HOST 10.0.12.2
    cf start mongodb-service-broker
    ```


4. View the catalog endpoint. Run:
    ```bash
    cf apps
    ```
to verify your service broker app running. Grab its URL from the last column. Append /v2/catalog to the URL, so the full url looks like:
    https://mongodb-service-<random-words>.apps.colfax.cf-app.com/v2/catalog

The username is "pivotal" and the password is "keepitsimple". You should see the JSON representation of your service broker’s catalog.

Verify the service is running by viewing the catalog via curl
    ```bash
    curl -s -k -u pivotal:keepitsimple https://mongodb-service-broker-<random-words>.apps.colfax.cf-app.com/v2/catalog | jq .
    ```

5. Register your service broker. We will register the service broker as space-scoped, which keeps it private to your space. Register your service broker with:
    ```bash
    cf create-service-broker mongodb-service-broker-LASTNAME pivotal keepitsimple https://mongodb-service-broker-<random-words>.apps.colfax.cf-app.com --space-scoped
    ```

View the service broker with the command:
    ```bash
    cf service-brokers
    ```

View the service access with:
    ```bash
    cf service-access
    ```

Note the service access is set to "none" since it is a space-scoped service broker. Verify that your service is listed in the marketplace:
    ```bash
    cf marketplace
    ```



# Part 2: Provisioning and Binding
1. Provision/Deprovision (/v2/service_instances/:instance_id). View the file:
    src/main/java/org/springframework/cloud/servicebroker/mongodb/service/MongoServiceInstanceService.java

The function createServiceInstance provisions a MongoDB database. The function deleteServiceInstance deprovisions a MongoDB database. The service broker tracks the provisioned MongoDB databases in its own repository.


2. Bind/Unbind (/v2/service_instances/:instance_id/service_bindings/:binding_id). View the file:
    src/main/java/org/springframework/cloud/servicebroker/mongodb/service/MongoServiceInstanceBindingService.java

The function createServiceInstanceBinding creates a unique set of credentials for the binding. The function deleteServiceInstanceBinding deletes the credentials for the binding. The service broker tracks the binding in its own repository.


3. Use MongoDB in spring-music. We will now create an instance of our MongoDB service and bind it to the spring-music app we have used in our other labs. To create an instance of our service, run:
    ```bash
    cf create-service MongoDB-N standard mongo-service
    cf bind-service spring-music mongo-service
    cf restart spring-music
    ```

Open your spring-music app, and click the  icon in the top-right corner. You should see it is now connected to your MongoDB database.


4. Cleanup by unbinding the services and deleting the service broker
    ```bash
    cf unbind-service spring-music mongo-service
    cf restart spring-music
    cf delete-service mongo-service
    cf delete-service-broker mongodb-service-broker-LASTNAME
    ```
