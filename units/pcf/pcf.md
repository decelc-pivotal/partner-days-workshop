---
pageTitle: Exercise #1 - Pushing an application to Pivotal Cloud Foundry
---

# Introduction

In this lab, we will use the CF CLI to push an app to Pivotal Cloud Foundry.

1. Install the CF CLI. Following the instructions for your platform here: https://github.com/cloudfoundry/cli#downloads. To see a list of all available commands, run
    ```bash
    cf help -a
    ```


2. Find your PCF credentials from the event slack channel


3. Log in to PCF. Using the url, username, and password from the site above, run this command:
    ```bash
    cf login -a [URL] -u admin -p PASSWORD --skip-ssl-validation
    ```  
Enter the username when prompted for email, and the password when prompted for password.


4. Target your org/space. Orgs and spaces are used for managing roles and permissions. Apps and services are space-scoped.
    cf target -o training-[Your first initial + last name] -s training


5. Get the sample app.
    ```bash
    git clone https://github.com/cf-platform-eng/spring-music
    cd spring-music
    ```


6. Build the sample app. Navigate to the app directory in a shell, and run:
    ```bash
    ./gradlew assemble
    ```


7. Push the app. In the app directory, run:
    ```bash
    cf push
    ```
Look at the output to see the staging process.


8. See you running app. See your application:
    ```bash
    cf apps
    ```
The URL of your app shows in the last output column (note: the url is randomized to avoid conflicts). Open the URL in a web browser to see the running app!


9. Look at the app’s logs. Run the command:
    ```bash
    cf logs spring-music
    ```
This will stream the app’s logs to your terminal. Refresh the app’s page in your browser and see the logs go by. Press Ctrl-C to stop streaming. You can view recent logs with:
    ```bash
    cf logs spring-music --recent
    ```


11. Make a data change in the UI then restart the app with:
    ```bash
    cf restart spring-music
    ```
Observe that, once restarted, the data reverts. This is a 12-factor app and we haven't added a backing data storage service yet.


10. Scale your app. Run the command:
    ```bash
    cf scale -i 5 spring-music
    ```
Then run
    ```bash
    cf apps
    ```
There are now 5 instances of your app running. Awesome.


12. Cleanup. Scale back down the app
    ```bash
    cf scale -i 1 spring-music
    ```
