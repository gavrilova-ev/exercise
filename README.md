# SAP Leonardo Machine Learning Foundation and the SAP S/4HANA Cloud SDK
Here, we provide the instructions to proceed with the code jam "SAP Leonardo Machine Learning and the SAP S/4HANA Cloud SDK". Below, you find the following information:
* [Technical prerequisites](#prerequisites): setup required to execute the steps described in this documentation. This information was provided before the workshop, so, we assume that those prerequisites are already fulfilled. Nevertheless, you can use this description to double check.
* [Task 0: Preparation steps](#task0)
* [Task 1: Retrieve SAP S/4HANA data using the SAP S/4HANA Cloud SDK virtual data model](#task1)
* [Task 2: Set up the Continuous Delivery Toolkit and the CI/CD pipeline of the S/4HANA Cloud SDK in GKE](#task2)
* [Task 3: Integrate SAP Leonardo Machine Learning service to provide translations](#task3)
* [Bonus, Task 4: Write data back to SAP S/4HANA using the SAP S/4HANA Cloud SDK virtual data model](#task4)

So, let us get started!

## <a name="prerequisites">Technical prerequisites</a>
You do not need any additional software installed locally for this code jam, as we will use the full stack WebIDE for development. All the steps of this code jam also are described considering that you re using the WebIDE.
```diff
- TODO: remove this part if OP WebIDE is used
```
### Set up Web IDE
To get access to WebIDE, you can use your SAP Cloud Platform, Neo trial account:
![Access SAP Cloud Platform, Neo account](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.55.20.png)
Then, navigate to Services and choose SAP Web IDE Full-Stack:
![SAP Web IDE Full-Stack service](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.56.46.png)
Make sure that the service is enabled or enable it, at the end you should see the status "Enabled":
![SAP Web IDE is enabled](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.57.13.png)
After that, choose "Go to Service" in the section "Take Action":
![Go to service Web IDE](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.57.36.png)
After the Web IDE loads, you should see the workspace, where we will create a new project in the next steps:
![Web IDE workspace](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/webIDE/Screenshot%202019-02-25%20at%2012.58.45.png)

### Set up SAP Cloud Platform account
We will deploy the application in SAP Cloud Platform, Cloud Foundry. For that purpose, you would require your own trial account. [Here](https://cloudplatform.sap.com/try.html), you can find information on how to get your trial account in SAP Cloud Platform, Cloud Foundry.
You can choose different regions and infrastructure provider for your subaccounts in the menu section "Regions" after you have created your trial account. In the Regions section choose US Central (IA) on the GCP infrastructure:
![SAP Cloud Platform account on Google Cloud Platform](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/SCPaccount/Screenshot%202019-02-25%20at%2013.15.43.png)
If you already have a trial account and want to reuse it, please make sure to have sufficient entitlements for the following services in the US Central (IA) region:
Application runtime: 3 or more
Destination: 1
![Manage entitlements](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/SCPaccount/Screenshot%202019-02-25%20at%2013.19.22.png)

### SAP S/4HANA
For this workshop, you can use the SAP S/4HANA system (you can ask the access information from your instructors) that we provide or the mock server available via the URL:
https://odata-mock-server-shiny-kudu.cfapps.us30.hana.ondemand.com/

### GitHub Account
We will share our source code via [GitHub](https://github.com/). So, you will require your own GitHub account for this CodeJam. If you do not already have one, please, make sure to Sign Up for GitHub, as proposed in the [initial page](https://github.com/).

## <a name="task0">Task 0: Preparation steps</a>
Before, we get started with the actual implementation, we need to perform some preparation steps and familiarize ourselves with the project structure.
* We provide you with the GitHub repository for this codejam that includes the skeleton of the project you will extend with the S/4HANA and Leonardo ML Foundation integration capabilities. All you need to do is to fork this repository. All the steps in this code jam will be executed in your own fork of this repository.

* Open and login into WebIDE
* Now, we will clone our new forked repository into our WebIDE workspace. In the WebIDE, choose the "Development" perspective, right click on "Workspace" -> Git -> Clone repository -> paste the URL of your fork of the code jam and choose "Clone". This will create a local copy of your project in WebIDE. After the request is completed successfully, we can start investigating our predefined project structure.
* Investigate your project structure:
  * **.che** is a technical folder created automatically by WebIDE and is required to correctly interpret the projecgt structure and to build and run project
  * Artifacts **cx-server**, **Jenkinsfile**, **pipeline_config.yml** help to set up and customize CI/CD server and the pipeline for your SDK based solutions. Those resources will be used to set up CI/CD pipeline for our application later. We also highly encourage you to check out [the related resources after the workshop](https://blogs.sap.com/2017/09/20/continuous-integration-and-delivery/)
  * **srv** folder and its artifacts: <br>
    **application** folder contains the business logic that we will extend in this code jam. It also contains the JS based frontend components in the **webapp** subfolder. We will only focus on backend components, though.
    **integration-tests** and **unit-tests** folders include integration and unit tests. We have already prepared the integration tests for your application, they do not pass yet, though, and therefore are ignored for now. <br>
    **pom.xml** is a [maven configuration file](https://maven.apache.org/pom.html)
  * **.gitignore** file is used to exclude certain files in your working directory from your Git history
  * **mta.yaml** is a build and deployment descriptor to be able to build and deploy the application in SAP Cloud Platform, Cloud Foundry.
  * **solutions** folder contains the solutions for the coding tasks of this code jam. Use it wisely :)


Before we get started with the development, let us familiarize ourselves on how to build and execute tests of the application in WebIDE without deploying it in SAP Cloud Platform. That will help us to quickly test changes in the application that we will perform in the next steps.

While building the application, we will execute integration tests. For the integration tests, you need to provide the URL and credentials of your SAP S/4HANA system.
* Open the file `srv/integration-tests/src/test/resources/systems.yml`.
As we will be using the parameters of the provided SAP S/4HANA system in this code jam, please make sure that the file contains the following content.
```
---
erp:
  default: "S4HANA"
  systems:
    - alias: "S4HANA"
      uri: "https://odata-mock-server-shiny-kudu.cfapps.us30.hana.ondemand.com/"

```
* In the same directory, create a `credentials.yml` file used during tests with the following content and make sure to put the correct name and password provided by the code jam instructor.
```
---
credentials:
- alias: "S4HANA"
  username: "(username)"
  password: "(password)"
```

After this, right clink on the "srv" folder and choose Build -> Build and Run Tests. Wait till the build finishes and make sure that you get "SUCCESS" for all the executed steps: Root, Application, Unit Tests, Integration Tests.

Please note, if you are using SAP WebIDE on SAP CLoud Platform, you might need to explicitly configure SAP Cloud Platform, Cloud Foundry environment. This can be done in Preferences -> Cloud Foundry. >ou can set up API Endpoint, Organization, and Space there and also install Build in your space, which is required for the deployment of applications from WebIDE.  

Now, after we got familiar with the local testing of the application, let us start with the first step: integrating SAP S/4HANA into this application using the SAP S/4HANA Cloud SDK.

## <a name="task1">Task 1: Retrieve SAP S/4HANA data using the SAP S/4HANA Cloud SDK virtual data model</a>
In this step, we will investigate two queries to SAP S/4HANA to retrieve business partner data. Firstly, we will retrieve the list of business partners for the list view in  the application. Secondly, we will take a look at the query retrieving detailed data of a single business partner by ID.

Start the development of queries by looking into the class BusinessPartnerServlet, which is the servlet exposing the business partner APIs. 
We could use any API framework here, such as JAX-RS or Spring. However, we use a servlet here for simplicity. Looking into the servlet, we can see that the main functionality is moved out into the commands GetAllBusinessPartnersCommand and GetSingleBusinessPartnerByIdCommand. Open and implement the command *GetAllBusinessPartnersCommand* as explained below.

The *GetAllBusinessPartnersCommand* should return a list of available business partners in the ERP system. The class was already created. We just have to implement the execute method:
* The instance of the class *BusinessPartnerService* already provides a method to retrieve all business partners. Type *service* to see a list of all available methods. Use the method *getAllBusinessPartner* to fetch multiple business partner entities.
* We only want to return the properties first name, last name and id. Thus, select only these properties by using the *select* method on the result from step 1. Luckily, we do not have to know the exact names of these properties in the public API of S/4HANA. They are codified as static member of the class *BusinessPartner*. We can select the business partner id by using *BusinessPartner.BUSINESS_PARTNER*. Please, do the same for the first name and last name.
* There are multiple categories of business partners. In this session, we only want to retrieve persons. The category is identified by a number, which is stored in the static class variable called *CATEGORY_PERSON*. The method to filter is called *filter* and can be executed on the result from the previous step.
The property *BusinessPartner.BUSINESS_PARTNER_CATEGORY* should equal *CATEGORY_PERSON*. To express that use the methods provided by the object *BusinessPartner.BUSINESS_PARTNER_CATEGORY*.
* All the previous steps did not execute any requests, but just defined the request. With the method *execute* you finally execute the query and retrieve the result.

Hint: Try to solve it on your own. However, the solution can also be found in the solution folder in the session material.

Now, also take a look at he command *GetSingleBusinessPartnerByIdCommand*. It was already implemented for you. Based on this source code, can you find out how the OData "expand" method can be implemented using the Virtual Data Model of the SAP S/4HANA Cloud SDK? Hint: addresses of business partners are retrieved using *expand*.

To check whether the queries are implemented correctly, go to the integration-tests folder and remove the *@Ignore* annotation for the following test: *BusinessPartnerServletTest.testGetAll()*.
Now, build and test the application as described in [Task 0](#task0) and make sure that the tests ran successfully. 

If the unignored test do not show errors, congratulations! You have successfully integrated SAP S/4HANA with your application.
In this case, let us push the local changes into our remote GitHub repository. 
For that, choose "Git Pane" on the right, choose the files that you have modified, stage and commit and push:
![Push to GitHub from Web IDE](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/git/Screenshot%202019-02-25%20at%2013.30.25.png)

Now, it is time to think about how we can continuously deliver this application.

## <a name="task2">Task 2: Set up the Continuous Delivery Toolkit and the CI/CD pipeline of the S/4HANA Cloud SDK in GKE</a>
Generally, you can use several ways to deploy your applications in SAP Cloud Platform. The recommended way for productive applications is to use the [Continuous Delivery Toolkit](https://github.com/SAP/cloud-s4-sdk-pipeline), which also ensures that our source code is properly tested and checked before being deployed. 

In this task, we will setup the Continuous Delivery Toolkit on GKE and SAP Cloud Platform account - everything we need to successfully deploy our application in an automated manner in SAP Cloud Platform account.

### Prepare SAP Cloud Platform Account
Here, we describe how to deploy your application manually using the SAP Cloud Platform, Cloud Foundry Cockpit. Necessary preliminary steps are also described here.
* Create service instances for S/4HANA connectivity
* Create destination endpoints

Firstly, create an instance of the destination service to connect to SAP S/4HANA (mock) system. For that, in the cloud platform cockpit on the level of your development space choose Services -> Service Marketplace and choose the destination service from the catalog.
Instantiate the service with all the default parameters. Give the name my-destination to your instance.
![Destination service in the Service Marketplace](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/destination.PNG)

Secondly, create an instance of the Authorization and Trust Management service. In the Service Marketplace, choose the Authorization and Trust Management service and instantiate it with the default parameters. Give the name my-xsuaa to your service instance.
![Authorization and Trust Management](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/uaa.PNG)

Next, we will create a destination endpoint to connect to the provided SAP S/4HANA system.
You can find the configuration of the destination endpoints on the level of your subaccount by choosing Connectivity -> Destinations.
Then, you can create a new destination endpoint by choosing "New Destination".

For the S/4HANA connectivity, create the destination with the following parameters. Make sure to put user credentials provided by the instructor. Those are the same credentials that you have used for testing in WebIDE in [Task 0](#task0) <br>
```diff
- TODO: 
add correct S4 URL
```
Name: ErpQueryEndpoint <br>
Type: HTTP <br>
URL: https://odata-mock-server-shy-sitatunga.cfapps.eu10.hana.ondemand.com/ <br>
Proxy type: Internet <br>
Authentication: BasicAuthentication <br>
User: (username) <br>
Password: (password) <br>

Now, we are ready to deploy our application in SAP Cloud Platform, Cloud Foundry. We will do it using the Continuous Delivery Toolkit that is also testing the application and ensures the code quality in the next step.

### Set up Continuous Delivery Toolkit in GKE
```diff
- TODO: 
1. Discuss with Google: if each participant will create its own cluster or we have a central cluster with the pre-installed Jenkins -> adapt the tutorial accordingly
2. Modify the linked Google next tutorial removing google next-specific points
3. Adding the Jenkins configurations: 
test credentials
deployment credentials
```
[GKE CI/CD Tutorial](https://github.com/SAPDocuments/Tutorials/blob/73cd62ac35d1ecd0df14c2b1cac60eccac20107c/tutorials/s4sdk-continuous-delivery-toolkit-setup/s4sdk-continuous-delivery-toolkit-setup.md)

If you still have time, continue with the next task.
In the next step, we will see how to integrate one of the SAP Leonardo Machine Learning services into your application.

## <a name="task3">Task 3: Integrate SAP Leonardo Machine Learning services</a>

The following step will explain how to quickly get an idea off the ground that requires integrating SAP Leonardo Machine Learning Foundation services found on SAP API Business Hub into an SAP Cloud Platform side-by-side extension using the SAP S/4HANA Cloud SDK. Please note that SAP Leonardo Machine Learning services in SAP API Hub are only intended for prototyping and not for production. For production, you would use use the service provisioning via SAP Cloud Platform service binding or the service keys, both leveraging the monitoring and security infrastructure of the platform. 
In our application, we will use the translation service for the example implementation. To implement the integration with ML services, we will leverage the SAP S/4HANA Cloud SDK component that simplifies the integration.

### Find the corresponding service in SAP API Hub
Firstly, go to SAP API Hub: https://api.sap.com/ and choose "Log On" on the right top corner of the screen.
Type "translation" in the search and choose "Inference service for machine translation" from the found services. The documentation for the service opens up and you can see the metadata, try the service out and even generate the code by clicking "Code snippet". By clicking "Show API Key", you can get the API key specific for your user to authenticate against the API and we will use it later in the configuration in SAP Cloud Platform.

### Create destination configuration in SAP Cloud Platform
Secondly, create a new destination configuration with the name sap_api_business_hub_ml in your SAP Cloud Platform account. This destination configuration should point to the sandbox system in SAP API hub and should include an additional parameter API_KEY with the value of your API key:
![API Hub destination](https://github.com/gavrilova-ev/codejam/blob/master/docs/pictures/SCPaccount/Screenshot%202019-03-06%20at%2018.33.35.png)

### Implement missing pieces
Thirdly, to implement the integration with the created API Hub desctination, find the package *machinelearning* in your project, where you will find the *TranslateServlet* class. This class contains the method *doGet*, which does the translation from german to english language for the provided text in the input parameter. This servlet will be called from the application frontend every time, when a user clicks on a term, which is shown in german language on the screen.

In this method, we already provide the implementation and leave some TODOs related to request building for you. In the code, you need to do the following steps to build HttpPost query: <br>
**TODO 1**: Get the Destination object from the SCP destination configuration using the S/4HANA Cloud SDK. Hint: you might use the class DestinationAccessor of the S/4HANA Cloud SDK to get any destination configured in SCP by its name. <br>
**TODO 2**: Using the Destination object construct the API endpoint for API sandbox by combining info from destination and TRANSLATION_PATH. Create HttpPost request using the constructed URL. Hint: you can get the URL configured in the destination using the destination object create in TODO 1 and its method getUri() <br>
**TODO 3**: Set additional postRequest headers: "Content-Type", "application/json" and "Accept", "application/json;charset=UTF-8" Hint: this can be done using the setHeader() method on the HttpPost query. <br>
**TODO 4**:  Using the Destination object retrieve APIKey and add it to the postReqwuest header. Hint: use the method getPropertiesByName() on the Desrtination object created in TODO 1 to retrieve corresponding API key. <br>

If you experience difficulties, you can compare you solution with the one provided in the [folder solutions](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/solutions/application/src/main/java/com/sap/cloud/s4hana/examples/addressmgr/machinelearning/commands/MlTranslationCommand.java).

Build and test this version in WebIDE. If everything compiles successfully, push your changes into GitHub repository, as was explained in [Task 1](#task1).

### Deploy the application using the Continuous Delivery Toolkit
Finally, we will deploy the application in your development space in SAP Cloud Platform, Cloud Foundry, as it was done in the previous task.

Push your changes into GitHub repository, as was explained in [Task 1](#task1).

Now, go to your Jenkins and start the build for your project again.
```diff
- TODO: 
add the screenshot
```

When the application is deployed, you can drill down into the application, choose the link for the application and append "/address-manager" to it. You should be able to see the business partner coming back from the mock server and you should be able to translate their professions by clicking on them.

![Result of the deployment](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/deploymentResult.PNG)

![Business Partner Address Manager with the integrated translation service](https://github.com/SAP/cloud-s4-sdk-book/blob/ml-codejam/docs/pictures/Translation.PNG)

Congratulations! You have just finished the main steps in this code jam:
* Firstly, we have integrated SAP S/4HANA Business Partner APIs to read the list of business partners and to read the detailed information
* Secondly, we have deployed the Continuous Delivery Toolkit of the S/4HANA Cloud SDK in our GKE cluster to continuously and automatically test, check, and deliver our applications.
* Thirdly, we have integrated SAP Leonardo Machine Learning functional service, using the Translation as an example.

Continue to the next steps in case you have time. 

## <a name="task4">Bonus, Task 4: Write data back to SAP S/4HANA using the SAP S/4HANA Cloud SDK virtual data model</a>
Here, we will further investigate the capabilities of the SAP S/4HANA Cloud SDK virtual data model to integrate SAP S/4HANA now also for update, and delete operations.

*	The class BusinessPartnerService already offers methods to create, update or delete address. The input values, such as the addresses or IDs to delete are member variables of the commands. They are passed into the command from the servlet.
*	Implement the run methods in the commands UpdateAddressCommand and DeleteAddressCommand. 
*	For the delete method we first have to create a business partner address instance, which has the IDs for the business partner and the address specified. The class BusinessPartnerAddress offers the method builder to create a builder and can be used as follows:

```
BusinessPartnerAddress addressToDelete = BusinessPartnerAddress.builder()
        .businessPartner(businessPartnerId)
        .addressID(addressId)
        .build();
```

*	Two commands expect an integer to be returned as result. These integers should correspond to the status code returned from SAP S/4HANA as result of the modification. You can simple call getHttpStatusCode to get the status code back.

Try to implement the queries by yourself. Feel free to check out the solution folder that we have prepared for you in case you are experiencing difficulties.

To test your logic, we have already prepared the tests. Go the the class AddressServletTest, which resides in the integration-tests module and remove all @Ignore annotations. Run the tests in this class and make sure that all tests are green. If not, get back to your commands and fix the issues.

If the tests are successful, you can now push your changes to GitHub again and start the Jenkins build and deployment
