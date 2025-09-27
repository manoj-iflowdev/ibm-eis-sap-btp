# Part 3: Configure EIS Alerts

Alerting in EIS provides you with access to weather alerts for your organization. For the list of available weather alerts, see the Catalog of weather alerts. You can create alert rules to send notifications for specific weather events to recipients by email, to an API endpoint, and to the Action center.

In this part of the tutorial, you will set up EIS alerts from a special HEARTBEAT criteria to post messages to the API Connector. 

## Prerequisites

- You will need an EIS account. There's a free 30-day trial available at the [product home page](https://www.ibm.com/products/environmental-intelligence-suite).
- The Request URL and Authorization String of the SAP Open Connector instance created in Part 2.

## Steps

### Step 1: Before you begin

If it hasn't be done already, create an IAM API key in your IBM Cloud account, by following the IBM Cloud documentation [here](https://cloud.ibm.com/docs/account?topic=account-userapikey&interface=ui#create_user_key). Then, go to the [EIS information page](https://www.app.ibm.com/environmental-intelligence/dashboard?cuiURL=static/settings/information) to get the account information.

Create some environment variables:

```sh
export MY_IAM_API_KEY="<IAM API Key>"
export MY_ORG_ID="<Organization ID>"
export MY_TENANT_ID="<Tenant ID>"
export MY_CONNECTOR_REQUEST_URL="<Request URL>"
export MY_CONNECTOR_AUTHORIZATION_STRING="<Authorization String>"
```

**NOTE**:
- For a Client ID such as saascore-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx, saascore is the service name, and xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx is the Tenant ID.
- The Authorization String should look like User xxxxxx, Organization yyyyyy, Element zzzzzz.

### Step 2: Create authentication tokens

Data and metadata service APIs use a JSON Web Token (JWT) for authentication and authorization. When you call an API with an HTTP Authorization request header, you must provide an unexpired token with a proper scope and signed by an approved JSON Web Key (JWK).

Generate an IAM Token and store it in an environment variable:

```sh
export MY_IAM_TOKEN=$(curl -X POST --location 'https://iam.cloud.ibm.com/identity/token' \
  --data-urlencode 'grant_type=urn:ibm:params:oauth:grant-type:apikey' \
  --data-urlencode "apikey=${MY_IAM_API_KEY}" | jq -r .access_token)
```

Generate a JSON Web Token and store it in an environment variable:

```sh
export JWT=$(curl -X GET --location "https://api.ibm.com/saascore/run/authentication-retrieve?orgId=${MY_ORG_ID}" \
  --header "X-IBM-Client-Id: saascore-${MY_TENANT_ID}" \
  --header "Authorization: Bearer ${MY_IAM_TOKEN}")
```

**NOTE**: More details about the authentication tokens can be found [here](https://www.ibm.com/docs/en/environmental-intel-suite?topic=apis-creating-authentication-tokens).

### Step 3: Store the Authorization String as a Secret

A Secret object can be created in EIS to store the Authentication String of the SAP Open Connector instance you've created in Part 2.

Create a JSON file for the Secret:

```sh
cat <<EOF > mywebhooksecretkey.json
{  
  "value" : "${MY_CONNECTOR_AUTHORIZATION_STRING}"
}
EOF
```

Make an API call to create the Secret named mywebhooksecretkey:

```sh
curl -v -X PUT --location "https://api.ibm.com/infohub/run/metadata/api/v1/na/tenants/${MY_TENANT_ID}/webhook/mywebhooksecretkey" \
  --header "X-IBM-Client-Id: infohub-${MY_TENANT_ID}" \
  --header 'Accept: application/json' \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer ${JWT}" \
  -d @mywebhooksecretkey.json
```

**NOTE**:
- The na in the URL corresponds to the North America region. Other regions will use a different two-letter string.
- Ensure that this command runs successfully, and you get a 201 response status code.

### Step 4: Create an Alert Endpoint

A notification message for a weather alert can be sent as a JSON payload to an HTTP endpoint.

Create a JSON file for the Alert Endpoint. For example:

```sh
cat <<EOF > myalertendpoint.json
{  
  "endpointType": "AlertEndpoint",
  "logicalName": "myalertendpoint",
  "apiKeyName": "mywebhooksecretkey",
  "webhookPayloadUrl": "${MY_CONNECTOR_REQUEST_URL}",
  "allowedOperations": [
      "create",
      "update",
      "delete"
  ],
  "isMetadataOnly": true,
  "tenantId": "${MY_TENANT_ID}",
  "name": "myalertendpoint",
  "authType": "custom",
  "contentType": "default"
}
EOF
```

Make an API call to create the Alert Endpoint from the JSON file:

```sh
curl -X POST --location 'https://api.ibm.com/infohub/run/metadata/api/v1/na/endpointdefinitions' \
  --header "X-IBM-Client-Id: infohub-${MY_TENANT_ID}" \
  --header 'Accept: application/json' \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer ${JWT}" \
  --data @myalertendpoint.json
```

### Step 5: Create a Contact

For each Contact, you specify the E-mail address, first name, last name, and language. Later, when you create an Asset, you can associate one or more Contacts with the Asset.

Create a JSON file for the Contact. For example:

```sh
cat <<EOF > mycontact.json
{
  "ce": {
    "tenantId": "${MY_TENANT_ID}",
    "timestampEventOccurred": "",
    "eventDetails": {
      "businessObject": {
        "contactIdentifier": "mycontact",
        "emailAddress": "jsmith@example.com",
        "firstName": "John",
        "lastName": "Smith",
        "locale": "en",
        "globalIdentifiers": [
          { "name": "eis.geospatial.globalId", "value": "mycontact" }
        ]
      }
    }
  }
}
EOF
```

Make an API call to create the Contact from the JSON file:

```sh
curl -X POST --location 'https://api.ibm.com/infohub/run/graph/na' \
  --header "X-IBM-Client-Id: infohub-${MY_TENANT_ID}" \
  --header 'Accept: application/json' \
  --header "Authorization: Bearer ${JWT}" \
  --header 'Content-Type: application/json' \
  --data "$(echo '{"query": "mutation ($ce: ContactInput!) { upsertContact(ContactEvent: $ce) }", "variables": '"$(cat mycontact.json | jq -r tostring)"'}')"
```

**NOTE**: If you get an error bash: !: event not found, you can run set +H to disable history substitution, and then run the command again.

### Step 6: Create an Asset

An Asset represents a location, which is defined by geographic coordinates. One or more Contacts can be associated with an Asset.

Create a JSON file for the Asset. For example:

```sh
cat <<EOF > myasset.json
{
  "ae": {
    "tenantId": "${MY_TENANT_ID}",
    "timestampEventOccurred": "",
    "eventDetails": {
      "businessObject": {
        "globalIdentifiers": [
          {
            "name": "eis.geospatial.globalId",
            "value": "myasset.STATIC"
          }
        ],
        "assetIdentifier": "myasset",
        "assetType": "STATIC",
        "currentLocationCoordinates": "58.474311344996245,-97.22213745117188",
        "description": "description of myasset",
        "contacts": [
          {
            "globalIdentifiers": [
              {
                "name": "eis.geospatial.globalId",
                "value": "mycontact"
              }
            ]
          }
        ]
      }
    }
  }
}
EOF
```

**NOTE**: In order to recieve a heartbeat message every 30min, the coordinates of the Asset's location must matches what's defined in a special HEARTBEAT criteria, which is currently defined as "58.474311344996245,-97.22213745117188".

Make an API call to create the Asset from the JSON file:

```sh
curl -X POST --location 'https://api.ibm.com/infohub/run/graph/na' \
  --header "X-IBM-Client-Id: infohub-${TENANT_ID}" \
  --header 'Accept: application/json' \
  --header "Authorization: Bearer ${JWT}" \
  --header 'Content-Type: application/json' \
  --data "$(echo '{"query": "mutation ($ae: AssetInput!) { upsertAsset(AssetEvent: $ae ) }", "variables": '"$(cat myasset.json | jq -r tostring)"'}')"
```

### Step 7: Create an E-mail Message Template

With an E-mail message template, you can customize the subject and content of the message you receive. The default template is provided in English but you can choose any of the supported languages. You can use a number of variables to insert dynamic text, such as the name of the weather event or the government alert headline message.

Create a JSON file for the E-mail Message Template. For example:

```sh
cat <<EOF > myemailtemplate.json
{
  "name": "myemailtemplate",
  "logicalName": "myemailtemplate",
  "description": "description of myemailtemplate",
  "descriptionInfo": "My E-mail template.",
  "tenantId": "${MY_TENANT_ID}",
  "type": "EmailMessageTemplate",
  "messages": [
    {
      "bodyContentType": "HTML",
      "subjectLine": "Weather notification - Example Co.",
      "body": "<!DOCTYPE html><html lang=\"en\"><head><title>Weather notification - Example Co.</title></head><body>This is a test.</body></html>",
      "locale": "en"
    }
  ],
  "enabled": true,
  "defaultLocale": "en",
  "isGloballyVisible": false,
  "isComplete": true
}
EOF
```

Make an API call to create the E-mail Template from the JSON file:

```sh
curl -X POST --location "https://api.ibm.com/infohub/run/metadata/api/v1/na/emailmessagetemplates?tenantId=${MY_TENANT_ID}" \
  --header "X-IBM-Client-Id: infohub-${MY_TENANT_ID}" \
  --header 'Accept: application/json' \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer ${JWT}" \
  --data @myemailtemplate.json
```

### Step 8: Create a Criteria

A criteria definition can represent a single weather alert or a set of alerts.

Create a JSON file for the Criteria. For example:

```sh
cat <<EOF > mycriteria.json
{
  "name": "mycriteria",
  "logicalName": "mycriteria",
  "tenantId": "${MY_TENANT_ID}",
  "description": "description for mycriteria",
  "type": "SavedFilter",
  "enabled": true,
  "primaryBusinessObjectTypes": [
    "WeatherEvent"
  ],
  "hint": {
    "viewId": "geospatialevents"
  },
  "advancedFilter": {
    "OR": [
      {
        "GREATER_EQUALS": [
          {
            "SELECT": "heartbeat"
          },
          {
            "VALUE": "0"
          }
        ]
      }
    ]
  }
}
EOF
```

Make an API call to create the Rule from the JSON file:

```sh
curl -X POST --location "https://api.ibm.com/infohub/run/metadata/api/v1/na/filters" \
  --header "X-IBM-Client-Id: infohub-${MY_TENANT_ID}" \
  --header 'Accept: application/json' \
  --header "Authorization: Bearer ${JWT}" \
  --header 'Content-Type: application/json' \
  --data @mycriteria.json
```

### Step 9: Create an Alert Rule

The Alert Rule connects the notification methods to a criteria definition. When the Alert Rule is activated, notifications are sent for the weather alerts that meet the criteria for the rule. Notifications are sent to HTTP endpoints, to contacts who are subscribed to the criteria, and to the Action center.

Create a JSON file for the Rule. For example:

```sh
cat <<EOF > myrule.json
{
  "name": "myrule",
  "type": "AlertRule",
  "tenantId": "${MY_TENANT_ID}",
  "isGloballyVisible": false,
  "logicalName": "myrule",
  "description": "myrule",
  "primaryBusinessObject": [
    "WeatherEvent"
  ],
  "isComplete": true,
  "enabled": true,
  "databaseViewUsedByThisRule": "geospatialevents",
  "instructions": {
    "ifEvent": {
      "OR": [
        {
            "MATCHES_SAVED_FILTER": {
                "LOGICAL_NAME": "HEARTBEAT",
                "VERSION": 1
            }
        }
      ]
    },
    "thenNotify": [
      {
        "emailMessageTemplateLogicalName": "myemailtemplate",
        "notificationType": "email",
        "contacts": {
          "contactLookupFieldPath": "Asset.contacts.id",
          "isLookup": true
        }
      },
      {
        "endpoint": {
          "endpointId": "1234abcd-5678-90ef-1234-567890abcdef",
          "isLookup": false,
          "endpointLogicalName": "myalertendpoint"
        },
        "notificationType": "webhook"
      },
      {
        "notificationType": "sendObjectUpsertEvents"
      }
    ]
  }
}
EOF
```

**NOTE**: The value of endpointId should be the actual id of the Alert Endpoint created in a previous step.

Make an API call to create the Rule from the JSON file:

```sh
curl -X POST --location "https://api.ibm.com/infohub/run/metadata/api/v1/na/alert/rules" \
  --header "X-IBM-Client-Id: infohub-${MY_TENANT_ID}" \
  --header 'Accept: application/json' \
  --header "Authorization: Bearer ${JWT}" \
  --header 'Content-Type: application/json' \
  --data @myrule.json
```

If you've completed all the steps successfully, you should be getting alert messages every 30 minutes, both at the E-mail address of the EIS Contact you just defined, and in the Event Mesh queue(s) you created in SAP BTP in Part 1.

---

## Maintainer

This project is maintained by **Manoj Gali**, a Senior SAP CPI Consultant with over 5 years of experience in SAP integration technologies.

### About the Developer
Manoj specializes in designing and optimizing scalable integration solutions across SAP CPI, PI/PO, and hybrid landscapes. With deep technical expertise in Groovy Scripting, XSLT, and User-Defined Functions (UDFs), he focuses on ensuring secure and high-performance data exchange for complex enterprise transformation initiatives.

**Contact Information:**
- Email: manoj.gali695@gmail.com
- Role: SAP CPI Consultant
- Expertise: SAP Integration, API Management, Cloud Transformation