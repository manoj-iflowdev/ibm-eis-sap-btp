# Part 1: Deploy SAP Event Mesh

SAP Event Mesh allows applications to communicate through asynchronous events.

In this part of the tutorial, you will set up a message queue using the SAP Event Mesh service in SAP Business Technology Platform (BTP). For a better understanding of SAP Event Mesh, go through the tutorial [Learn about SAP Event Mesh](https://developers.sap.com/tutorials/cp-enterprisemessaging-learn-messaging-concepts.html).

## Prerequisites

- You have an enterprise account with SAP BTP in order to set up SAP Event Mesh. Note that at the time of writing a trial account cannot currently be used for this.
- [Set up SAP Event Mesh in BTP Cockpit](https://help.sap.com/docs/SAP_EM/bf82e6b26456494cbdd197057c09979f/3ef34ffcbbe94d3e8fff0f9ea2d5911d.html).
- [Set up the Event Mesh User Interface](https://help.sap.com/docs/SAP_EM/bf82e6b26456494cbdd197057c09979f/83777b586ec54a01b5e807620f5c4660.html).

## Steps

### Step 1: Create an SAP Event Mesh Service Instance

Follow the directions [here](https://help.sap.com/docs/SAP_EM/bf82e6b26456494cbdd197057c09979f/d0483a9e38434f23a4579d6fcc72654b.html) to create a SAP Event Mesh Service Instance. 

When you create it, name the instance "eis-alerts". Fill the following JSON in the form to create the service instance.

```
{
  "emname": "eis-alerts",
  "namespace": "myorg/weather/eis",
  "version": "1.1.0",
  "options": {
    "management": true,
    "messagingrest": true,
    "messaging": true
  },
  "rules": {
    "queueRules": {
        "publishFilter": [
            "${namespace}/*"
        ],
        "subscribeFilter": [
            "${namespace}/*"
        ]
    },
    "topicRules": {
        "publishFilter": [
            "${namespace}/*"
        ],
        "subscribeFilter": [
            "${namespace}/*"
        ]
    }
  }
}
```

### Step 2: Create a queue

In this step you will create a queue called "alerts" in SAP Event Mesh.

Click on the SAP Event Mesh service in the SAP Cockpit to open the SAP Event Mesh GUI. When the GUI opens, click on the "eis-alerts" tile.

![image](https://media.github.ibm.com/user/24824/files/54b0f961-ae41-4aad-b1ee-11234ed5013a)

Go to the Queues tab, and click on Create Queue.

![image](https://media.github.ibm.com/user/24824/files/57be7430-f4dd-4ce7-b8ca-ee686a5a7661)

Name your queue "alerts" and click Create.

![image](https://media.github.ibm.com/user/24824/files/67365594-706c-4ee5-aadf-d01d98d357ad)

Now you will be able to see the newly created queue under the Queues tab. 

![image](https://media.github.ibm.com/user/24824/files/7be858bf-3279-4f88-9c58-a8a688857cdf)

You can view the number of messages in the queue from the above window. This will come in handy when you start sending EIS alerts to the queue.

Try sending and receiving messages under the Test tab.

![image](https://media.github.ibm.com/user/24824/files/1a4b6260-c0df-455e-b840-4ab169894de0)

This functionality is also useful when you want to see and consume the EIS alert messages.

### Step 3: Subscribe to a topic

Messages can be posted either to queues directly, or posted to a topic. When a message is posted to a topic, the message will appear on every queue that is subscribed to that topic. Note that while queues persist message until they are consumed, topics will not persist messages.

Topics can be useful in context of weather alerts. You could potentially create multiple topics for different classes of alerts. Queues may correspond to applications. The queue for any particular application can be subscribed to topics of interest for that application.

For this tutorial, you will subscribe your queue to a single topic. Under the Queues tab in the GUI, click on the Actions menu button. Under this menu, click Queue subscriptions.

In the form, add a queue subscription named:
```
myorg/weather/eis/alerts
```

Later, you will configure IBM EIS to send alerts to this topic.

### Step 4: Create a service key

In this step, you will create a Service Key for the SAP Event Mesh instance. This will allow a client program to send messages to and receive messages from the SAP Event Mesh instance. Follow the directions [here](https://help.sap.com/docs/service-manager/sap-service-manager/creating-service-keys-in-cloud-foundry) to create the service key.

Once created, click on the Service Key in SAP BTP Cockpit. This opens a window that gives you the connection information in JSON format. 

Scroll down in this window until you find the "httprest" protocol section. Make note of the values in this section, as they will be used in the next part of this tutorial. 

![image](https://media.github.ibm.com/user/24824/files/a3f97208-deaa-4ffb-9988-9c05b5ddf152)

Scroll down until you find the "httprest" protocol section. Make note of the values for "clientid", "clientsecret", "tokenendpoint" and "uri" as these will be used in the next part of this tutorial. 

### Step 5: Test SAP Event Mesh from the Command Line (Optional)

In this step you will validate that the token is working by using "curl" to send and receive messages. You can potentially use this approach to have an application consume alert messages from a queue.

Set the values you have found in the previous step as environment variables.

```
export EM_TOKEN_ENDPOINT='<tokenendpoint>'
```
```
export EM_CLIENT_ID='<clientid>'
```
```
export EM_CLIENT_SECRET='<clientsecret>'
```
```
export EM_URL='<uri>'
```

Run the following command to get an access token that will be used to send and receive messages from SAP Event Mesh.

```
export EM_ACCESS_TOKEN=`curl "${EM_TOKEN_ENDPOINT}?grant_type=client_credentials&response_type=token" -H "Content-Type: application/x-www-form-urlencoded" --user ${EM_CLIENT_ID}:${EM_CLIENT_SECRET} | jq -r '.access_token'`
```

Next, run the following command to set an environment variable to the queue name, including the namespace. 
```
export EM_QUEUE_NAME="myorg%2Fweather%2Feis%2Falerts"
```
Note that in this string, the "/" characters of the path are replaced with "%2F".


#### Send a message

Run the following command to send a message to the queue. The JSON message is in the body of this POST.
```
curl -X POST -H "Authorization: Bearer ${EM_ACCESS_TOKEN}" -H "x-qos: 0" -H "Content-Type: application/json" "${EM_URL}/messagingrest/v1/queues/${EM_QUEUE_NAME}/messages" -d "{'property':'value'}"
```

#### Consume a message

Run the following to consume a message from the queue.
```
curl -X POST -H "Authorization: Bearer ${EM_ACCESS_TOKEN}" -H "x-qos: 0" -H "Content-Type: application/json" "${EM_URL}/messagingrest/v1/queues/${EM_QUEUE_NAME}/messages/consumption"
```
This call will return a message from the queue, consuming it in the process. The body of the return message will be empty when no more messages reside in the queue.

## Next steps

In Part 2 of this tutorial, you will use SAP Open Connectors to provide an API proxy to SAP Event Mesh. Instead of calling SAP Event Mesh directly, EIS alerting will instead use the proxy. This allows us to add necessary headers when the alert is passed on to SAP Event Mesh.

## Maintainer

This project is maintained by Manoj Gali, a Senior SAP CPI Consultant with over 5 years of experience in SAP integration technologies. Manoj specializes in designing, developing, and optimizing scalable integration solutions across SAP CPI, PI/PO, and hybrid landscapes.

For questions or contributions regarding this implementation, please reach out via:
- Email: manoj.gali695@gmail.com
- Expertise: Groovy Scripting, XSLT, and SAP BTP Integration Suite