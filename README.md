# AWS Serverless Summary

This repo is mainly for myself.
It is based on what I've Learned doing [Maximilian Schwarzmüller](https://www.udemy.com/course/aws-serverless-a-complete-introduction/) course on [Udemy.com](https://udemy.com) and try to summarize it.

AWS has gazilion and one services if not more. This paper only include the minimum required to build a serverless API / App.
Each of the services described below, has its own site with lots and lots of examples and documentation. This is just my own short summary.
You can look at it as my personal notebook.

That said, you are more than welcome to go along and read it, use it and if you have any comments, I'll be happy to receive them and learn some more.

I will try to keep each topic organized under it's title, but I'm not making any promises.

---

- [IAM (Identity and Access Management)](#iam--identity-and-access-management-)
- [API Gateway](#api-gateway)
  - [Creating a RESTful API](#creating-a-restful-api)
  - [Add a resource](#add-a-resource)
  - [Add a method](#add-a-method)
  - [Models](#models)
  - [Creating sub resources](#creating-sub-resources)
- [Lambda functions](#lambda-functions)
- [DynamoDB](#dynamodb)
  - [Creating a table](#creating-a-table)
- [AWS Cognito](#aws-cognito)
- [Hosting](#hosting)
- [Route 53](#route-53)

<small><i><a href='http://ecotrust-canada.github.io/markdown-toc/'>Table of contents generated with markdown-toc</a></i></small>

## IAM (Identity and Access Management)

> AWS Identity and Access Management (IAM) enables you to manage access to AWS services and resources securely. Using IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.

It is best that after you create your AWS account, you go into IAM panel and set your account permissions as instructed.

1.  Delete Root access keys.
    The user creating the account is the root user.
    By default, this user has full access to all resources in the account.
    It is best practice to delete it's access keys and create new user groups for each task needed in the account.
2.  Create a new user group. Use IAM groups to assign permissions to your IAM users to simplify managing and auditing permissions in your account.

    You can do it by:

    - Manage Groups
    - Create New Group
    - Set group name
    - Attach policies to group. (We can create our own policies.)

3.  Create new users and add to groups.
    Create IAM users and give them only the permissions they need.

    You can do it by:

    - Manage Users
    - Add User
    - Set user name
    - Select AWS access type
    - Attach to one of the user's groups.
    - Add tags (optional) - This is good to keep external data on the user. for example his / her email or phone number.

4.  Set users MFA (Multi factor authentication)
    for example Google authenticator.

## API Gateway

> Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. APIs act as the "front door" for applications to access data, business logic, or functionality from your backend services. Using API Gateway, you can create RESTful APIs and WebSocket APIs that enable real-time two-way communication applications.

### Creating a RESTful API

1. Click Create API
2. Click on the build button for REST API
3. Set API name and description (You can choose to change Endpoint type)

Once this is done, we have our API. Now we need to add to it resources.
A resource is like a controller. Each resource can be set with different rest methods we like to expose for it.

### Add a resource

From the `Actions` dropdown, select `Create Resource`.
Then set the resource name and path (usually we will use the same value for both).
If needed, set `Enable API Gateway CORS` this will create an Options method for our resource.
Notice that this does not expose our methods to CORS. We will need to do it ourselves for each method we need to be CORS enabled.

### Add a method

1. Select the resource for which you want to add a method. Then select `Create Method` from the dropdown menu.
2. Select the method type
3. Set `Integration type`. usually we'd like to trigger a lambda function.
   We could check "Use Lambda Proxy integration" option. This will pass the entire request event to the lambda function.
   Or we could manipulate the request passed to the function using `Integration Request` (more on that below).
   Now select the region where the lambda function is and select the function itself.

Now you should see 5 boxes on the right side of the screen.

**Test**

Enables you to test the API method.

**Method Request**

Configures the public interface for the API.
Here we can set if the request requires authentication / API key.
What headers does the request requires. For example "Auth".

**Integration Request**

Here we can map the data passed to the lambda function we trigger.

Add mapping template "application/json" and click on it to edit the json we pass.

If for example the request includes a "Person" object and we only need its name, we can use it as so.
'name': **"**$input.json('$.Person.name')**"** (notice the quotes for the string).

Now we can use `event.name` in our lambda function

More on the mapping template can be found [here](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html)

**Method Response**

Provide information about this method's response types, their headers and content types.

I'm gonna reference `Method Response` before I reference `Integration Response`, because here is where we set our response types and their headers.
**We do not set the values fot this response types**. This is done under `Integration Response`.

**Integration Response**

Here we can map our response back to the client.
Here is where we set for example the value for "Access-Control-Allow-Origin" header.
Or where we set the data we pass as the response. We do this as we did with the request mapping.
We don't have to set the mapping templates and just use the response from our lambda function as the response body.

### Models

We can create a model using a ["JSON Schema"](https://json-schema.org/learn/getting-started-step-by-step.html).
This model enables us to validate the request and map the response.

**Creating a model**
Models, Create -- Set model name and schema.

**Validating Request**
Method Request:

- Request Validator (Validate Body + ...)
- Request Body -> Add Model -- application/json + select the model

**Mapping request and response**
We can use the model to map the data from the request to the lambda function.
In `Integration Request`, `Mapping Templates`, `application/json` in the `Generate Template` dropdown, select the model we want to map.

The same goes for mapping the response using `Integration Response`.

### Creating sub resources

We can create sub resources and its path using `{}`.
This will pass what ever is in the `{}` as a parameter to our API method
We can then use this parameter to map it to our lambda function like this.

```js
    {
        "parameter": "$input.params('parameter')"
    }
```

**It is important to remember that in order to reach our API and the changes made, we need to deploy it. This should be done for every change we want to share (Make sure to select the right resource to deploy)**

## Lambda functions

- Set function name
- Set function runtime (nodejs, .net core ...)
- Set function user role. This is needed to enable function permissions. We can create a new role or use an existing one.

Each function has a `handler`.
This handler is the module-name.export value in the function. For example, "index.handler" calls exports.handler in index.js.

Environment variables - Can be passed into the function. This are just like `.env` variables we use in any nodejs application.

Tags - Are good to tag the function for analytics

We can split our function in to multiple files and require those files, using `require('')`.

**Connecting Lambda functions to Other services**
In order to use other AWS services, such as DynamoDB, in our lambda function, we need to import AWS SDK.
This SDK is already available in lambda function and no extra installation required.

There are SDK available for in browser javascript, nodejs applications or many other languages.

```js
const AWS = require('aws-sdk');
const dynamoDB = new AWS.DynamoDB({
  region: 'us-west-2',
  apiVersion: '2012-08-10'
});

// It is important to tell DynamoDB which region to use or it will use AWS default region.
// It is also important to set apiVersion to prevent conflicts
```

Now we can use any of dynamoDB's methods.

```js
const params = {
  Item: {
    UserId: { S: '12345' }
    Age: {N: 37},
    IsActive: {BOOL: true},
    Name: {S: 'Gilad Lev-Ari'},
    Kids: {SS: ['Yair', 'Noam', 'Lilach']},
    KidsAge: {NS: [8, 5, 1]}
  },
  TableName: "MyTable"
};

dynamoDB.putItem(params, (err, data) => {
    // Return response to client
})

```

A reference to AWS SDK can be found [here](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/).

**First thing to note, is that the lambda function is async while the dynamoDB.putItem uses a callback**. This means that we might want to either create a rapper around the dynamoDB function that will return a promise. Or we want to use the old way to implement lambda function using callbacks.
I prefer the first approach.

**Second thing to note, is that when a lambda function uses another AWS service, it need to have the proper permissions**.

When we query a DynamoDB table, the returned data, will be in the same shape as our params from the above example. This means that we'll need to parse it to what the client expects. A good place for that could be in the API `Integration Response`

## DynamoDB

> Amazon DynamoDB is a key-value and document database that delivers single-digit millisecond performance at any scale. It's a fully managed, multiregion, multimaster, durable database with built-in security, backup and restore, and in-memory caching for internet-scale applications. DynamoDB can handle more than 10 trillion requests per day and can support peaks of more than 20 million requests per second.

We can only create one DB per AWS region.

### Creating a table

First thing we need to do when creating a table, is to set its name and primary key.
DynamoDb divides the data by partitions (groups). This what enables it to be so fast.
When the key of a record is made of a partition key only, no two items can have the same partition key.

**Primary key** can either be structured using the partition key or using a partition key + sort key.

An example of a partition key as primary key will be "UserId"

An example of a partition key + sort key as primary key will be "ArtistName" + "SongTitle"

We can add more secondary indexes to help DynamoDB query the table faster.

When creating a new table, we could use the default table settings **but** this may come with default costs.
I prefer to set the Read / Write capacity mode to "on demand" (we can change it later).

Once the table was created we can add extra settings.
For example if the data in the table is temporary, we could set a TTL, and it will be deleted after this time is over.

We can also set alarms on the table, to be informed when needed. For example exceeding number of reads per second or when table size exceeds the maximum we want to enable.

## AWS Cognito

> Amazon Cognito lets you add user sign-up, sign-in, and access control to your web and mobile apps quickly and easily. Amazon Cognito scales to millions of users and supports sign-in with social identity providers, such as Facebook, Google, and Amazon, and enterprise identity providers via SAML 2.0.

**Creating a User Pool**

**User Pool** is the process of registering to the service without using a third party login service such as google or facebook connect.

**Federated Identities** is the use of third party authentication services such as google or facebook connect.

1. Set Pool name
2. Select how you want to configure your user pool. Either by using a default setting which you can change or by starting with a fresh sheet.
3. Set the way you want your user to be able to sign in (username + password, email + password, username / email + password ….)
4. Select which attributes you want your user should input while registering.

Once done. Go to the next phase.
Setting password and registration policies.
We can also add user verification options. Such as multi factor authentication.

**App clients** This is what allows us to connect our frontend client with the cognito authentication which later enables or disables our API gateway access.
Notice that if our app is running in the browser, we should **uncheck** “Generate client secret”, because we cannot protect the secret in the client

- We can trigger lambda functions on different steps in the authentication process.

**The Tokens**

1. **Identity Token** - contains claims about the identity of the authenticated user such as name, email, and phone_number You can use this identity information inside your application. The ID token can also be used to authenticate users against your resource servers or server applications. When an ID token is used outside of the application against your web APIs, you must verify the signature of the ID token before you can trust any claims inside the ID token.
2. **Access Token** - contains scopes and groups and is used to grant access to authorized resources. The primary purpose of the access token is to authorize API operations in the context of the user in the user pool. For example, you can use the access token to grant your user access to add, change or delete user attributes. The access token can also be used with any of your web APIs to make access control decisions and authorize operations for your users based on scopes or groups.
3. **Refresh Token** - Contains the information necessary to obtain a new ID or access token.

**Adding cognito SDK to the client**

We could use the full AWS SDK (either by npm install or by a script tag in our HTML) or if we only need the cognito part of the SDK we could just use `amazon-cognito-identity-js` (npm install --save amazon-cognito-identity-js)

** UserPoolId can be found in the pool general settings.
** ClientId can be found in the App clients section.

SignUp example:

```js
import { CognitoUserPool, CognitoUserAttribute } from 'amazon-cognito-identity-js';

const poolData = { UserPoolId: '...', ClientId: '...' };
const userPool = new CognitoUserPool(poolData);
const dataEmail = { Name: 'email', Value: '...' };
const dataPhoneNumber = { Name: 'phone_number', Value: '...' };
const attributeEmail = new CognitoUserAttribute(dataEmail);
const attributePhoneNumber = new CognitoUserAttribute(dataPhoneNumber);

const attributeList = [];
attributeList.push(attributeEmail);
attributeList.push(attributePhoneNumber);

userPool.signUp('username', 'password', attributeList, null, (err, result) => {
  if (err) {
    alert(err.message || JSON.stringify(err));
    return;
  }
  const cognitoUser = result.user;
  console.log('user name is ' + cognitoUser.getUsername());
});
```

We pass username, password and the list of attributes we marked as required when creating our UserPoll.

Examples can be found [here](https://github.com/aws-amplify/amplify-js/tree/master/packages/amazon-cognito-identity-js#usage)

**Using a Cognito Authorizer in the API**

In Api gateway, select the resource you want to authorize and select `Authorizer`
Set the Authorizer name and select Cognito as its type.
Now select the method we want to check for authorization and select the new cognito authorizer.

Sending the token to the API

```js
const poolData = {
  UserPoolId: '...', // Your user pool id here
  ClientId: '...' // Your client id here
};
const userPool = new CognitoUserPool(poolData);
const cognitoUser = userPool.getCurrentUser();
cognitoUser.getSession(function(err, session) {
  const token = session.getIdToken().getJwtToken();
  // Set request headers - “Authorization”: token
});
```

We could also get the accessToken the same way: session.getAccessToken().getJwtToken();

We can map our UserId from Cognito to lambda using `$context.authorizer.claims.sub`
Just like we map any other param passed to the lambda function in `Integration Request`

Using Cognito inside Lambda

```js
const cognito = new AWS.CognitoIdentityServiceProvider({apiVersion: ‘2016-04-18’});
var params = {  AccessToken: event.accessToken };
cognito.getUser(params, function(err, data) {
    if (err) {
        console.log(err, err.stack); // an error occurred
    } else {
        console.log(data);           // successful response
        const userId = data.UserAttributes[0].Value;
    }
});
```

[More in the reference page](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/CognitoIdentityServiceProvider.html)

## Hosting

We can host our static site as all of our static files on `S3` (Simple Storage Service).
For that we need to create a bucket to save our files.

**Creating a bucket on S3**
All buckets are shared across AWS. This means you cannot use a bucket name if someone else already took that name.
**Bucket names shouldn’t include spaces.**

We then set our bucket properties (versioning, logging ...) and set our bucket permissions

Once Our bucket is ready, we just select the files we need and upload them to `S3`.
We need to select how `S3` will store our files (Which type of store class to use).
Each store class has its own usage. [More on that](https://docs.aws.amazon.com/AmazonS3/latest/dev/storage-class-intro.html)

[We also need to set the permission to our bucket](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2)

**We want to keep our bucket public so users could get to it and we want to grant anonymous users access to read only to our bucket**

```js
{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Sid":"PublicRead",
      "Effect":"Allow",
      "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::examplebucket/*"]
    }
  ]
}
```

To set our `S3` bucket as a hosting server, we go into properties and select “Static website hosting”. Set the entry page (index.html) as the index document and if it is an SPA site use it as the error page as well.

**Adding Logging**

We can add another bucket just as we did before.
Then we can go to our original bucket and under “Server access logging” set our logging bucket as our site logs destination.

**Using CloudFront**
Create Distribution - Get Started (Web) - Select S3 bucket + set site properties

- We can select how long to cache our site
- We can ask `CloudFront` to gzip our files and more

Now we get a new domain name. The `CloudFront` domain.

In order to use this domain name with Route 53, we need to set its “Alternate Domain Names (CNAMEs)” to the domain name we really want to use.

## Route 53

This is AWS domain service.
We can assign our domain and register it under `Route 53` to use our `CloudFront` domain.

**Hosted zones** - If we bought our domain on AWS it will be in our hosted zones otherwise we can create it.

We need to create a new record set. We can select Alias and select our `CloudFront` domain from the list.

If we want to enable any subdomain, we need to create a record set for each subdomain and we need to register it in our `CloudFront` domain `CNAMES`.
