# Produce/Consume Event hub message with Managed Identity

This sample shows how to produce/consume Event Hub with Managed Identity in `Azure Spring Cloud`.

## Prerequisite

* [JDK 8](https://docs.microsoft.com/en-us/azure/java/jdk/java-jdk-install)
* [Maven 3.0 and above](http://maven.apache.org/install.html)
* [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) or [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview)

## How to run 

1. Run `mvn clean package`.
2. Install Azure CLI extension for Azure Spring Cloud by running below command.
    ```
    az extension add -y --source https://azureclitemp.blob.core.windows.net/spring-cloud/spring_cloud-0.1.0-py2.py3-none-any.whl
    ```
3. Create an instance of Azure Spring Cloud.
    ```
    az spring-cloud create -n <resource name> -g <resource group name>
    ```
4. Create an app with public domain assigned.
    ```
    az spring-cloud app create -n <app name> -s <resource name> -g <resource group name> --is-public true 
    ```
5. Enable system assigned Managed Identity.
   ```
   az spring-cloud app identity assign -n <app name> -s <resource name> -g <resource group name>
   ```
6. Create role assignment and to the system-assigned Managed Identity
    ```
    az role assignment create --role "Azure Event Hubs Data owner" --scope $id --assignee-object-id <principal-id-you-got-in-step5>
    ```
7. Deploy app with jar
    ```
    az spring-cloud app deploy -n <app name> -s <resource name> -g <resource group name> --jar-path ./target/asc-managed-identity-eventhub-binder-sample-0.1.0.jar
    ```
8.  Verify app is running. Instances should have status `RUNNING` and discoveryStatus `UP`. 
    ```
    az spring-cloud app show -n <app name> -s <resource name> -g <resource group name>
    ```
9. Verify sample is working. The url is fetched from previous step.
    ```
    # Produce a message to Event hub
    curl -X PUT {url}/message?value={value}

    # Consume a message from event hub
    curl {url}/message
    ```