# Authentication 
Jexia provides two ways of authentication:
1. **API Key** - for a large amount of clients, unauthorized users and partners etc.
2. **Project User** - for specific users to have the ability to perform specific actions, such as updated or delete. 

By default, all data is inaccessible and you need to specify what resources can be accessed and by whom. Regardless of what option you choose for authentication, you will need to have a **policy** relating to either the API key or Users. 

<iframe width="700" height="394" src="https://www.youtube.com/embed/o2ZN3nvdhi8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

::: warning
Please keep in mind that the access token is valid for 2 hours. 
:::

## API Keys
An API key can be used to organize public access (for example your blog visitors). To create an API-Key you need to navigate to the **API keys** section and click **Create API Key** button.

![API-Key](./api-key.png)

When creating a new key, **Description** is a general-purpose field that should be used to describe the purpose of the new API key.
When you click on **Create API key**, the API key and API Secret will be generated. Please copy the API Secret to a safe location as you will not be able to access it again. We perform one-way encryption for the API secret. Therefore, the only way to get the secret again is to regenerate the API key. That is it, you are ready to create a **policy** for your new API key. 

For example, let's show how to provide **Read Only** access to our `orders` Dataset:

![API Policy](./api_policy.png)

Using the automatically generated API, we can now interact with our data. 

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
// Jexia client
import { jexiaClient } from "jexia-sdk-js/node";
// Dataset operation
import { dataOperations } from "jexia-sdk-js/node";

const ds = dataOperations();

// You need to use your API Key / API Secret which is generated within your Jexia application. 
// Do not forget make a Policy for your API!
jexiaClient().init({
  projectID: "PROJECT_ID",
  key: "API_KEY",
  secret: "API_SECRET",
}, ds);

// Now you can run the read operation for your Datasets
const orders = ds.dataset("orders");
const selectQuery = orders
  .select()
  .where(field => field("dislike").isEqualTo(true));

selectQuery.subscribe(records => {
    // You will always get an array of created records, including their 
    // generated IDs (even when inserting a single record) 
  },
  error => {
    // If something goes wrong, the error information is accessible here 
});
```
</template>
<template v-slot:bash>

```bash
# Environment variables to be set
export PROJECT_ID=<project_id>
export API_KEY=<key_here>
export API_SECRET=<secret_here>

# save API key token to our environment in case we need to use it
export API_TOKEN=`curl -X POST -d '{
  "method":"apk",
  "key":"'"$API_KEY"'",
  "secret":"'"$API_SECRET"'"
}' "https://$PROJECT_ID.app.jexia.com/auth" | jq .access_token`

# Select all data with our API token
curl -H "Authorization: Bearer $API_TOKEN"
  -X GET "https://$PROJECT_ID.app.jexia.com/ds/orders" | jq .
```
</template>
</CodeSwitcher>

## Project Users
Another way to authenticate within Jexia is to use **Project Users**. Usually, you need this when you want to provide more rights to specific users. Let's say **Update** and **Delete** actions for your blog. Firstly, you need to create a user under the **Project Users** section.

![UMS Users](./ums-2.png)

After this, you need go to the **Policies** section and create a new policy, ensuring that **Subject** is selected as **AllUsers** and the selected Datasets within **Resource** are what you want these users to be able to access. Finally, sselect all **Actions** you want these users to perform.

![Policy](./policy.png)

This will allow any registered user to have full CRUD access to the example `orders` dataset. 

<CodeSwitcher :languages="{js:'JavaScript',bash:'cURL'}">
<template v-slot:js>

``` js
// Jexia client and Project Users module
import { jexiaClient, UMSModule} from "jexia-sdk-js/node"; 

const ums = new UMSModule(); 

jexiaClient().init({  
  projectID: "PROJECT_ID",  
}, ums); 

// Sign in with the user created in your Jexia project
ums.signIn({  
  email: 'user@jexia.com',  
  password: 'secret-password'
});  
```
</template>
<template v-slot:bash>

``` bash
# Environment variables to be set
export PROJECT_ID=<project_id>
export TEST_USER=<user_here>
export TEST_USER_PSW=<password_here>

# save UMS token to our environment in case we need to access Project Users
export UMS_TOKEN=`curl -X POST -d '{
  "method":"ums",
  "email":"'"$TEST_USER"'",
  "password":"'"$TEST_USER_PSW"'"
}' "https://$PROJECT_ID.app.jexia.com/auth" | jq -r .access_token`

# Select all data with our ums token
curl -H "Authorization: Bearer $UMS_TOKEN"
  -X GET "https://$PROJECT_ID.app.jexia.com/ds/orders" | jq .
```

</template>
</CodeSwitcher>

## Policies
As was mentioned above to authorize access to your data you need to have a **policy** created for your API keys and Project Users. 
Currently, you cannot manipulate a **policy** via the API. All admin actions are only available via Jexia's administration dashboard.

<iframe width="700" height="394" src="https://www.youtube.com/embed/i4dKznoXry0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

There are three main areas within a **Policy**:
1. **Subject** (who has access) - can be an **API key**, **All Users** (all project users) or a **namespace** (grouped users under the same name).
2. **Resources** (access to what) - can be any dataset, fileset or channel. Selecting **All Users** means you allow these operations to be performed by all Project Users. 
3. **Actions** (what CRUD actions can be performed) - here you can specify which actions are allowed to be performed. These can be: Create, Read, Update and Delete. For Channels you have specific actions: Subscribe (read) and Publish (write).

![Policy](./policy.png)

::: tip
You can create as many policies as you need. All of them will be evaluated during the request. Changes in a policy have an immediate effect. 
:::