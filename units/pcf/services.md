---
pageTitle: Exercise #2 - Binding an application to a service
---

# Introduction

In this lab, we will create service and bind to an application

1.	Review the documentation on managing service instances: https://docs.cloudfoundry.org/devguide/services/managing-services.html


2.	Review what services are available in the marketplace.
    ```bash
    cf marketplace
    ```
Note that the marketplace is a slight misnomer: it's completely local to a PCF
installation and the contents are controlled by the system operator.


3.	Create service instance
    ```bash
    cf create-service p-redis shared-vm mydb
    ```

4.	Bind service to your application
    ```bash
    cf bind-service spring-music mydb
    ```

5.	Restart your app
    ```bash
    cf restart spring-music
    ```

6.	Check vcap services
    ```bash
    cf env spring-music
    ```

7.	Create a user-provided service. ** Be sure to replace "N" with your last name **
    ```bash
    cf cups mongo-ups -p '{"uri": "mongodb://10.0.12.2/mongo-ups-N"}'
    ```

8.	Unbind the redis service:
    ```bash
    cf unbind-service spring-music mydb
    ```

9.	Bind the user-provided service
    ```bash
    cf bind-service spring-music mongo-ups
    ```

10.	Restart your app
    ```bash
    cf restart spring-music
    ```

11. View the recent logs (see cf help or the last lab). In the boot sequence, note the app choosing the Mongo profile.


12. Make a data change in the UI then restart the app. Observe that, once restarted, the data now persists.


13. Cleanup by unbinding and deleting the services.
    ```bash
    cf unbind-service spring-music mongo-ups
    cf delete-service mongo-ups
    cf delete-service mydb
    cf restart spring-music
    ```
