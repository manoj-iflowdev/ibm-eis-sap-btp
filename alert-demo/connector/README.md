# Part 2: Deploy API Connector

In this part of the tutorial you will deploy an API connector using SAP Integration Suite. This connector will act as a proxy between the EIS alerting system and SAP Event Mesh. IBM EIS will send alert messages to the connector, and the connector will forward the alerts to SAP Event Mesh.

The connector handles the OAuth2 authentication flow with SAP Event Mesh, which involves obtaining and renewing bearer tokens. As well, it allows us to configure the message headers needed for SAP Event Mesh. When configuring an endpoint for an IBM EIS alert, we need only to specify a URL and a static authorization string.

## Prerequisites

- You have deployed SAP Event Mesh as shown in Part 1, and have recorded the values for the SAP Event Mesh instance, including "clientid", "clientsecret", "tokenendpoint" and "uri".
- You have deployed SAP Integration Suite in SAP BTP. Here is a [tutorial](https://developers.sap.com/tutorials/cp-starter-isuite-onboard-subscribe.html) that shows how to do this with a trial account. 
- You have the project files available on your local environment.

## Steps

### Step 1: Start the SAP Open Connectors GUI

If you have not already done so, install the SAP Integration Suite. You can use this [tutorial](https://developers.sap.com/tutorials/cp-starter-isuite-onboard-subscribe.html) to install it with a trial account. Once installed, click on the Manage Capabilities tile. This opens a window in which you can enable the Open Connectors capability.

Click on Create Connectors to enter the Open Connectors GUI.

![image](https://media.github.ibm.com/user/24824/files/cbea67dc-dc53-4f43-8542-f4e8bc1fb9e0)

### Step 2: Import the SAP Event Mesh connector

From the SAP Open Connectors GUI click on Build New Connector.

![image](https://media.github.ibm.com/user/24824/files/02230cff-6800-4c65-aaf7-2e5241e37032)

Click Import.

![image](https://media.github.ibm.com/user/24824/files/2b8563d8-9e44-47d8-b4a2-43c9867ea7f2)

Click From computer. 

![image](https://media.github.ibm.com/user/24824/files/ecfb811e-ef7c-41f9-8cd2-7eabeee6c9a8)

Import the connector from the project files on your local machine. The file will be named:

```
eis-connector-for-sap/alert-demo/connector/SAP Event Mesh.json
```

Once completed you will be able to see a newly added private connector called SAP Event Mesh. 

![image](https://media.github.ibm.com/user/24824/files/7e47562c-2986-4f75-ba81-b779aed81a8f)

### Step 3: Create a connector instance

In order to use the connector, you must create an instance of the connector. This instance will be configured with the SAP Event Mesh URL, bearer token and queue name that were found in Part 1.

Click on the Instances tab, and then click Create Instance.

![image](https://media.github.ibm.com/user/24824/files/851e46c9-ce54-46cd-9a47-8bec65272ed0)

Search for the newly added connector to add. Fill in the Name and apiKey fields, and then click Create Instance.

Set fields in the form using values as follows:
- Name: Choose a name for the connector instance.
- OAuth API Key: "clientid" from the SAP Event Mesh instance.
- OAuth API Secret: "clientsecret" from the SAP Event Mesh instance.
- OAuth Token URL: "tokenendpoint" from the SAP Event Mesh instance followed by the string: "?grant_type=client_credentials&response_type=token".
- uri: "uri" from the SAP Event Mesh instance. 
- x-address: "topic:myorg/weather/eis/alerts". 

Note: Instead of "topic" at the beginning of the x-address field, you can use "queue" to send messages directly to a queue.

Click Create Instance.

![image](https://github.com/ibm-ecosystem-engineering/ibm-eis-sap-btp/assets/49033907/8e2eef8d-0d35-4f14-91ce-b32a6b095934)

Select to import the sendMessage API call, and click Import.

![image](https://media.github.ibm.com/user/24824/files/2598336c-1307-49c8-a5ef-73c3ae826b92)

You have now created an instance of the connector, which can be viewed from the Instances tab.

### Step 4: Test the API connector

Go to the Instances tab and click the API Docs button.

![image](https://media.github.ibm.com/user/24824/files/4bf4eae5-cd63-4ce2-bc3f-c3f3d3366681)

Try out the sendmessage API call. You can fill in the body of the message with some arbitrary JSON, e.g. 
```
"{'key':'value'}"
```

![image](https://media.github.ibm.com/user/24824/files/131e1b46-d116-485c-b96e-cf0a19fea7c8)

Press the Execute button to make the call. If all goes well, you will get a 200 response from the call, and the JSON structure will be sent to the queue in SAP Event Mesh. You can verify that this was sent by consuming a message from the queue as shown in Part 1 of this tutorial, either using the SAP Event Mesh GUI or using the curl command.

Make sure that you record the values of the following strings from the sendmessage call:

- Authorization
- Request URL

These are required when configuring the IBM EIS alerts in the next part of this tutorial.

## Next Steps

In this part of the tutorial, you have set up an API connector in SAP BTP using SAP Open Connectors. In Part 3, you will configure IBM EIS to send alerts to the sendmessage endpoint exposed by this connector. 

## Maintainer
This project is maintained by Manoj Gali, a Senior SAP CPI Consultant with over 5 years of experience in SAP integration technologies. Manoj specializes in SAP CPI, PI/PO, and hybrid integration landscapes, with deep expertise in Groovy Scripting, XSLT, and User-Defined Functions (UDFs). Manoj focuses on designing and optimizing secure, high-performance data exchange solutions across SAP and non-SAP systems.

For technical inquiries or support, please contact:
- Email: manoj.gali695@gmail.com