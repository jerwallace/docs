---
---
# API

The API category provides a solution for making HTTP requests to REST and GraphQL endpoints. It includes a [AWS Signature Version 4](http://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) signer class which automatically signs all AWS API requests for you.

## Using GraphQL Endpoints

AWS Amplify API Module supports AWS AppSync or any other GraphQL backend.

To learn more about GraphQL, please visit [GraphQL website](http://graphql.org/learn/).
{: .callout .callout--action}

### Using AWS AppSync

AWS AppSync is a cloud-based fully-managed GraphQL service that is integrated with AWS Amplify. You can create your AWS AppSync GraphQL API backend with the Amplify CLI and connect to your backend using AWS Amplify API category. 

Learn more about [AWS AppSync](https://aws.amazon.com/appsync/) by visiting [AWS AppSync Developer Guide](https://docs.aws.amazon.com/appsync/latest/devguide/welcome.html){: .target='new'}.
{: .callout .callout--action}

#### Automated Configuration with CLI

After creating your AWS AppSync API, following command enables AppSync GraphQL API in your  project:

```bash
$ amplify add api
```

Select *GraphQL* when prompted for service type:

```terminal
? Please select from one of the below mentioned services (Use arrow keys)
❯ GraphQL
  REST
```

Name your GraphQL endpoint and select authorization type:

```terminal
? Please select from one of the below mentioned services GraphQL
? Provide API name: myNotesApi
? Choose an authorization type for the API (Use arrow keys)
❯ API key
  Amazon Cognito User Pool
```

AWS AppSync API keys expire seven days after creation, and using API KEY authentication is only suggested for development. To change AWS AppSync authorization type after the initial configuration, use the `$ amplify update api` command and select `GraphQL`.
{: .callout .callout--info}

When you update your backend with *push* command, you can go to [AWS AppSync Console](https://aws.amazon.com/appsync/) and see that a new API is added under *APIs* menu item:

```bash
$ amplify push
```

##### Updating Your GraphQL Schema

When you create a GraphQL backend with the CLI, the schema definition for your backend data structure is saved in *amplify/backend/api/YOUR-API-NAME/schema.graphql* file. 

Once your API is deployed, updating the schema is easy with the CLI. You can edit the schema file and run *amplify push* command to update your GraphQL backend.

For example, a sample GraphQL schema will look like this:

```graphql
type Todo @model {
  id: ID!
  name: String!
  description: String
}
```

Add a new  *Model* type to your schema:

```graphql
type Todo @model {
  id: ID!
  name: String!
  description: String
  model: String
}
```

Save your schema file and update your GraphQL backend:

```bash
$ amplify push
```

When you run the *push* command, you will notice that your schema change is automatically detected, and your backend will be updated respectively. 

```terminal
| Category | Resource name   | Operation | Provider plugin   |
| -------- | --------------- | --------- | ----------------- |
| Api      | myNotesApi      | Update    | awscloudformation |
| Auth     | cognito6255949a | No Change | awscloudformation |
```

When the update is complete, you can see the changes on your backend by visiting [AWS AppSync Console](https://aws.amazon.com/appsync/).

##### Using GraphQL Transformers

As you can notice in the sample schema file above, the schema has a `@model` directive. This tells Amplify CLI that the related types should be stored in an Amazon DynamoDB table. When you create or update your backend with *push* command, the CLI will automatically create the necessary data sources for you, behind the scenes.

The resource creation is based on AWS CloudFormation templates that you can find under *amplify/backend/api/YOUR-API-NAME/cloudformation-template.json* 

The following transformers are available to be used with AWS AppSync when defining your schema:  

| Directive | Description |
| --- | --- |
| @model | Used for storing types in Amazon DynamoDB. |
| @auth | Used to define different authorization strategies. | 
| @connection | Used for specifying relationships between @model object types. |
| @searchable | Used for streaming the data of an @model object type to Amazon Elasticsearch Service. |

##### Type Generation using GraphQL Schemas

When working with GraphQL data it is useful to import types from your schema for type safety. You can do this with the Amplify CLI's automated code generation feature. The CLI automatically downloads GraphQL Introspection Schemas from the defined GraphQL endpoint and generates TypeScript or Flow classes for you. Every time you push your GraphQL API, the CLI will provide you the option to generate types and statements.

If you want to generate your GraphQL statements and types, run:

```bash
$ amplify codegen
```

A TypeScript or Flow type definition file will be generated in your target folder.  

##### Using the Configuration File in Your Code

Import your auto-generated `aws-exports.js` file to configure your app to work with your AWS AppSync GraphQL backend:

```javascript
import aws_config from "./aws-exports";
Amplify.configure(aws_config);
```

#### Manual Configuration

As an alternative to automatic configuration, you can manually enter AWS AppSync configuration parameters in your app. Authentication type options are `API_KEY`, `AWS_IAM`, `AMAZON_COGNITO_USER_POOLS` and `OPENID_CONNECT`.

##### Using API_KEY

```javascript
let myAppConfig = {
    // ...
    'aws_appsync_graphqlEndpoint': 'https://xxxxxx.appsync-api.us-east-1.amazonaws.com/graphql',
    'aws_appsync_region': 'us-east-1',
    'aws_appsync_authenticationType': 'API_KEY',
    'aws_appsync_apiKey': 'da2-xxxxxxxxxxxxxxxxxxxxxxxxxx',
    // ...
}

Amplify.configure(myAppConfig);
```

##### Using AWS_IAM

```javascript
let myAppConfig = {
    // ...
    'aws_appsync_graphqlEndpoint': 'https://xxxxxx.appsync-api.us-east-1.amazonaws.com/graphql',
    'aws_appsync_region': 'us-east-1',
    'aws_appsync_authenticationType': 'AWS_IAM',
    // ...
}

Amplify.configure(myAppConfig);
```

##### Using AMAZON_COGNITO_USER_POOLS

```javascript
let myAppConfig = {
    // ...
    'aws_appsync_graphqlEndpoint': 'https://xxxxxx.appsync-api.us-east-1.amazonaws.com/graphql',
    'aws_appsync_region': 'us-east-1',
    'aws_appsync_authenticationType': 'AMAZON_COGNITO_USER_POOLS', // You have configured Auth with Amazon Cognito User Pool ID and Web Client Id
    // ...
}

Amplify.configure(myAppConfig);
```

##### Using OPENID_CONNECT

```javascript
let myAppConfig = {
    // ...
    'aws_appsync_graphqlEndpoint': 'https://xxxxxx.appsync-api.us-east-1.amazonaws.com/graphql',
    'aws_appsync_region': 'us-east-1',
    'aws_appsync_authenticationType': 'OPENID_CONNECT', // Before calling API.graphql(...) is required to do Auth.federatedSignIn(...) check authentication guide for details.
    // ...
}

Amplify.configure(myAppConfig);
```

### Using a GraphQL Server

To access a GraphQL API with your app, you need to configure the endpoint URL in your app's configuration. Add the following line to your setup:

```javascript

import Amplify, { API } from "aws-amplify";
import aws_config from "./aws-exports";
 
// Considering you have an existing aws-exports.js configuration file 
Amplify.configure(aws_config);

// Configure a custom GraphQL endpoint
Amplify.configure({
  API: {
    graphql_endpoint: 'https:/www.example.com/my-graphql-endpoint'
  }
});

```

#### Set Custom Request Headers for Graphql 

When working with a GraphQL endpoint, you may need to set request headers for authorization purposes. This is done by passing a `graphql_headers` function into the configuration:

```javascript
Amplify.configure({
  API: {
    graphql_headers: async () => ({
        'My-Custom-Header': 'my value'
    })
  }
});
```

#### Signing Request with IAM

AWS Amplify provides the ability to sign requests automatically with AWS Identity Access Management (IAM) for GraphQL requests that are processed through AWS API Gateway. Add *graphql_endpoint_iam_region* parameter to your GraphQL configuration statement to enable signing: 

```javascript
Amplify.configure({
  API: {
    graphql_endpoint: 'https://www.example.com/my-graphql-endpoint',
    graphql_endpoint_iam_region: 'my_graphql_apigateway_region'
  }
});
```

### Working with the GraphQL Client

AWS Amplify API category provides a GraphQL client for working with queries, mutations, and subscriptions.

#### Query Declarations


The Amplify cli codegen automatically generates all possible GraphQL statements (queries, mutations and subscriptions) and for JavaScript applications saves it in `src/graphql` folder

```javascript
import * as queries from './graphql/queries';
import * as mutations from './graphql/mutations';
import * as subscriptions from './graphql/subscriptions';
```

#### Simple Query

Running a GraphQL query is simple. Import the generated query and execute it with `API.graphql`:

```javascript
import Amplify, { API, graphqlOperation } from "aws-amplify";
import * as queries from './graphql/queries';


// Simple query
const allTodos = await API.graphql(graphqlOperation(queries.listTodos));
console.log(allTodos);

// Query using a parameter
const oneTodo = await API.graphql(graphqlOperation(queries.getTodo, { id: 'some id' }));
console.log(oneTodo);

```

#### Mutations

Mutations are used to create or update data with GraphQL. A sample mutation query to create a new *Todo* looks like this:

```javascript
import Amplify, { API, graphqlOperation } from "aws-amplify";
import * as mutations from './graphql/mutations';

// Mutation
const todoDetails = {
    name: 'Todo 1',
    description: 'Learn AWS AppSync'
};

const newTodo = await API.graphql(graphqlOperation(mutations.createTodo, {input: todoDetails}));
console.log(newTodo);
```

#### Subscriptions

Subscriptions is a GraphQL feature allowing the server to send data to its clients when a specific event happens. You can enable real-time data integration in your app with a subscription. 

```javascript
import Amplify, { API, graphqlOperation } from "aws-amplify";
import * as subscriptions from './graphql/subscriptions';

// Subscribe to creation of Todo
const subscription = API.graphql(
    graphqlOperation(subscriptions.onCreateTodo)
).subscribe({
    next: (todoData) => console.log(todoData)
});

// Stop receiving data updates from the subscription
subscription.unsubscribe();

```

When using **AWS AppSync** subscriptions, be sure that your AppSync configuration is at the root of the configuration object, instead of being under API: 

```javascript
Amplify.configure({
  Auth: {
    identityPoolId: 'xxx',
    region: 'xxx' ,
    cookieStorage: {
      domain: 'xxx',
      path: 'xxx',
      secure: true
    }
  },
  aws_appsync_graphqlEndpoint: 'xxxx',
  aws_appsync_region: 'xxxx',
  aws_appsync_authenticationType: 'xxxx',
  aws_appsync_apiKey: 'xxxx'
});
```

### React Components

API category provides React components for working with GraphQL data. 

#### Connect

`<Connect/>` component is used to execute a GraphQL query or mutation. You can execute GraphQL queries by passing your queries in `query` or `mutation` attributes:

```javascript
import React from 'react';
import Amplify, { graphqlOperation }  from "aws-amplify";
import { Connect } from "aws-amplify-react";

import * as queries from './graphql/queries';
import * as subscriptions from './graphql/subscriptions';

class App extends React.Component {

    render() {

        const ListView = ({ todos }) => (
            <div>
                <h3>All Todos</h3>
                <ul>
                    {todos.map(todo => <li key={todo.id}>{todo.name} ({todo.id})</li>)}
                </ul>
            </div>
        );

        return (
            <Connect query={graphqlOperation(queries.listTodos)}>
                {({ data: { listTodos }, loading, error }) => {
                    if (error) return (<h3>Error</h3>);
                    if (loading || !listTodos) return (<h3>Loading...</h3>);
                    <ListView todos={listTodos.items} />
                }}
            </Connect>
        )
    }
} 

export default App;

```

Also, you can use `subscription` and `onSubscriptionMsg` attributes to enable subscriptions:

```javascript

<Connect
    query={graphqlOperation(queries.listTodos)}
    subscription={graphqlOperation(subscriptions.onCreateTodo)}
    onSubscriptionMsg={(prev, { onCreateTodo }) => {
        console.log ( onCreateTodo );
        return prev; 
    }}
>
    {({ data: { listTodos }, loading, error }) => {
        if (error) return (<h3>Error</h3>);
        if (loading || !listTodos) return (<h3>Loading...</h3>);
        <ListView todos={listTodos ? listTodos.items : []} />
    }
 </Connect>

```

For mutations, a `mutation` function needs to be provided with the `Connect` component. A `mutation` returns a promise that resolves with the result of the GraphQL mutation.

```jsx
import * as mutations from './graphql/mutations';
import * as queries from './graphql/queries';
import * as subscriptions from './graphql/subscriptions';

class AddTodo extends Component {
  constructor(props) {
    super(props);
    this.state = {
        name: '',
        description: '',
    };
  }

  handleChange(name, ev) {
      this.setState({ [name]: ev.target.value });
  }

  async submit() {
    const { onCreate } = this.props;
    var input = {
      name: this.state.name,
      description: this.state.description
    }
    console.log(input);
    await onCreate({input})
  }

  render(){
    return (
        <div>
            <input
                name="name"
                placeholder="name"
                onChange={(ev) => { this.handleChange('name', ev)}}
            />
            <input
                name="description"
                placeholder="description"
                onChange={(ev) => { this.handleChange('description', ev)}}
            />
            <button onClick={this.submit.bind(this)}>
                Add
            </button>
        </div>
    );
  }
}

class App extends Component {
  render() {

    const ListView = ({ todos }) => (
      <div>
          <h3>All Todos</h3>
          <ul>
            {todos.map(todo => <li key={todo.id}>{todo.name}</li>)}
          </ul>
      </div>
    )

    return (
      <div className="App">
        <Connect mutation={graphqlOperation(mutations.createTodo)}>
          {({mutation}) => (
            <AddTodo onCreate={mutation} />
          )}
        </Connect>

        <Connect query={graphqlOperation(queries.listTodos)}
          subscription={graphqlOperation(subscriptions.onCreateTodo)}
          onSubscriptionMsg={(prev, {onCreateTodo}) => {
              console.log('Subscription data:', onCreateTodo)
              return prev;
            }
          }>
        {({ data: { listTodos }, loading, error }) => {
          if (error) return <h3>Error</h3>;
          if (loading || !listTodos) return <h3>Loading...</h3>;
            return <ListView todos={listTodos.items} />
        }}
        </Connect>
      </div>

    );
  }
}
```

## Angular
Amplify CLI generates APIService to make it easier to use Appsync API. Add an GraphQL API by running add api command in your project root folder
```bash
$ amplify add api
? Please select from one of the below mentioned services GraphQL
? Provide API name: angularcodegentest
? Choose an authorization type for the API API key
? Do you have an annotated GraphQL schema? No
? Do you want a guided schema creation? true
? What best describes your project: (Use arrow keys)
? What best describes your project: Single object with fields (e.g., “Todo” with ID, name, description)
? Do you want to edit the schema now? (Y/n) n
? Do you want to edit the schema now? No

```

Push the API to cloud by running `$amplify push`

```bash
 amplify push
| Category | Resource name      | Operation | Provider plugin   |
| -------- | ------------------ | --------- | ----------------- |
| Api      | angularcodegentest | Create    | awscloudformation |
? Are you sure you want to continue? true

GraphQL schema compiled successfully. Edit your schema at /Users/yathiraj/Documents/code/angular-codegen-test/amplify/backend/api/angularcodegentest/schema.graphql
? Do you want to generate code for your newly created GraphQL API (Y/n)
? Do you want to generate code for your newly created GraphQL API Yes
? Choose the code generation language target (Use arrow keys)
? Choose the code generation language target angular
? Enter the file name pattern of graphql queries, mutations and subscriptions (src/graphql/**/*.graphql)
? Enter the file name pattern of graphql queries, mutations and subscriptions src/graphql/**/*.graphql
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions (Y/n)
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions Yes
? Enter the file name for the generated code (src/app/API.service.ts)
? Enter the file name for the generated code src/app/API.service.ts
...
...
...
...
✔ Code generated successfully and saved in file src/app/API.service.ts
✔ Generated GraphQL operations successfully and saved at src/graphql
✔ All resources are updated in the cloud
```

Configure your Angular app to use the aws-exports. Rename the generated `aws-exports.js` -> `aws-exports.ts` and import it in `main.ts`

```typescript
// file: src/main.ts
// ...
import PubSub from '@aws-amplify/pubsub';
import API from '@aws-amplify/api';
import awsConfig from './aws-exports';

PubSub.configure(awsConfig);
API.configure(awsConfig);
// ...
```

Expose the APIService from the root of your app

```typescript
// file: src/app/app.module.ts

// ...
import { AppComponent } from './app.component';
import { APIService } from './API.service';
// ...

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ],
  providers: [APIService],
  bootstrap: [AppComponent]
})
export class AppModule { }

```

In your component use the API service 
```typescript
// file: app.component.ts
// ...
import { APIService } from './API.service';

// ...
export class AppComponent {
  constructor(private apiService: APIService) {}
}
```

APIService exposes all the queries and subscription as methods
```typescript
// file: app.component.ts
@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'angular-codegen-test';
  constructor(private apiService: APIService) {
    this.onNewTodo()
  }
  async createTodo() {
    const response = await this.apiService.CreateTodo({description: 'foo', name: 'bar'});
    console.log(response);
  }
  async onNewTodo() {
    this.apiService.OnCreateTodoListener.subscribe((next) => {
      console.log(next.value.OnCreateTodo);
    });
  }
}
```


## Using REST

The API category can be used for creating signed requests against Amazon API Gateway when the API Gateway Authorization is set to `AWS_IAM`. 

Ensure you have [installed and configured the Amplify CLI and library]({%if jekyll.environment == 'production'%}{{site.amplify.docs_baseurl}}{%endif%}/js/start).
{: .callout .callout--info}

### Automated Setup

Run the following command in your project's root folder:

```bash
$ amplify add api
```

Select `REST` as the service type.

```bash
? Please select from one of the below mentioned services
  GraphQL
❯ REST
```

The CLI will prompt several options to create your resources. With the provided options you can create:
- REST endpoints that triggers Lambda functions
- REST endpoints which enables CRUD operations on an Amazon DynamoDB table

During setup you can use existing Lambda functions and DynamoDB tables or create new ones by following the CLI prompts. After your resources have been created update your backend with the `push` command:

```bash
$ amplify push
```

A configuration file called `aws-exports.js` will be copied to your configured source directory, for example `./src`.

##### Configure Your App

Import and load the configuration file in your app. It's recommended you add the Amplify configuration step to your app's root entry point. For example `App.js` in React or `main.ts` in Angular.

```javascript
import Amplify, { API } from 'aws-amplify';
import aws_exports from './aws-exports';

Amplify.configure(aws_exports);
```

### Manual Setup

For manual configuration you need to provide your AWS Resource configuration and optionally configure authentication.

```javascript
import Amplify, { API } from 'aws-amplify';

Amplify.configure({
    // OPTIONAL - if your API requires authentication 
    Auth: {
        // REQUIRED - Amazon Cognito Identity Pool ID
        identityPoolId: 'XX-XXXX-X:XXXXXXXX-XXXX-1234-abcd-1234567890ab',
        // REQUIRED - Amazon Cognito Region
        region: 'XX-XXXX-X', 
        // OPTIONAL - Amazon Cognito User Pool ID
        userPoolId: 'XX-XXXX-X_abcd1234', 
        // OPTIONAL - Amazon Cognito Web Client ID (26-char alphanumeric string)
        userPoolWebClientId: 'a1b2c3d4e5f6g7h8i9j0k1l2m3',
    },
    API: {
        endpoints: [
            {
                name: "MyAPIGatewayAPI",
                endpoint: "https://1234567890-abcdefgh.amazonaws.com"
            },
            {
                name: "MyCustomCloudFrontApi",
                endpoint: "https://api.my-custom-cloudfront-domain.com",

            },
            {
                name: "MyCustomLambdaApi",
                endpoint: "https://lambda.us-east-1.amazonaws.com/2015-03-31/functions/yourFuncName/invocations",
                service: "lambda",
                region: "us-east-1"
            }
        ]
    }
});
```

### AWS Regional Endpoints

You can also utilize regional endpoints by passing in the *service* and *region* information to the configuration. For a list of available service endpoints see [AWS Regions and Endpoints](https://docs.aws.amazon.com/general/latest/gr/rande.html). 

As an example, the following API configuration defines a Lambda invocation in the `us-east-1` region:  

```javascript
API: {
    endpoints: [
        {
            name: "MyCustomLambda",
            endpoint: "https://lambda.us-east-1.amazonaws.com/2015-03-31/functions/yourFuncName/invocations",
            service: "lambda",
            region: "us-east-1"
        }
    ]
}
```

For more information related to invoking AWS Lambda functions, see [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/API_Invoke.html).

 **Configuring Amazon Cognito Regional Endpoints** To call regional service endpoints, your Amazon Cognito role needs to be configured with appropriate access for the related service. See [AWS Cognito Documentation](https://docs.aws.amazon.com/cognito/latest/developerguide/iam-roles.html) for more details.
 {: .callout .callout--warning}

### Using the API Client

To invoke a REST API, you need the name for the related endpoint. If you manually configure the API, you already have a name for the endpoint. If you use Automated Setup,  you can find the API name in your local configuration file. 

The following code sample assumes that you have used Automated Setup.

To invoke an endpoint, you need to set `apiName`, `path` and `headers` parameters, and each method returns a Promise.

Under the hood the API category utilizes [Axios](https://github.com/axios/axios) to execute the HTTP requests. API status code response > 299 are thrown as an exception. If you need to handle errors managed by your API, work with the `error.response` object.

#### **GET**

```javascript
let apiName = 'MyApiName';
let path = '/path'; 
let myInit = { // OPTIONAL
    headers: {}, // OPTIONAL
    response: true, // OPTIONAL (return the entire Axios response object instead of only response.data)
    queryStringParameters: {  // OPTIONAL
        name: 'param'
    }
}
API.get(apiName, path, myInit).then(response => {
    // Add your code here
}).catch(error => {
    console.log(error.response)
});
```

Example with async/await

```javascript
async getData() { 
    let apiName = 'MyApiName';
    let path = '/path';
    let myInit = { // OPTIONAL
        headers: {} // OPTIONAL
    }
    return await API.get(apiName, path, myInit);
}

getData();
```

**Using Query Parameters**

To use query parameters with *get* method, you can pass them in `queryStringParameters` parameter in your method call:

```javascript
let items = await API.get('myCloudApi', '/items', {
  'queryStringParameters': {
    'order': 'byPrice'
  }
});
```

**Accessing Query Parameters in Cloud API**

If you are using a Cloud API which is generated with Amplify CLI, your backend is created with Lambda Proxy Integration, and you can access your query parameters within your Lambda function via the *event* object:

```javascript
exports.handler = function(event, context, callback) {
    console.log (event.queryStringParameters);
}
```

Alternatively, you can update your backend file which is locates at `amplifyjs/backend/cloud-api/[your-lambda-function]/app.js` with the middleware:

```javascript
var awsServerlessExpressMiddleware = require('aws-serverless-express/middleware')
app.use(awsServerlessExpressMiddleware.eventContext())
```

In your request handler use `req.apiGateway.event`:

```javascript
app.get('/items', function(req, res) {
  // req.apiGateway.event.queryStringParameters
  res.json(req.apiGateway.event)
});
```

Then you can use query parameters in your path as follows:

```javascript
API.get('sampleCloudApi', '/items?q=test');
```

To learn more about Lambda Proxy Integration, please visit [Amazon API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html).
{: .callout .callout--info}

#### **POST**

Posts data to the API endpoint:

```javascript
let apiName = 'MyApiName'; // replace this with your api name.
let path = '/path'; //replace this with the path you have configured on your API
let myInit = {
    body: {}, // replace this with attributes you need
    headers: {} // OPTIONAL
}

API.post(apiName, path, myInit).then(response => {
    // Add your code here
}).catch(error => {
    console.log(error.response)
});
```

Example with async/await

```javascript
async function postData() { 
    let apiName = 'MyApiName';
    let path = '/path';
    let myInit = { // OPTIONAL
        body: {}, // replace this with attributes you need
        headers: {} // OPTIONAL
    }
    return await API.post(apiName, path, myInit);
}

postData();
```

#### **PUT**

When used together with [Cloud API](https://docs.aws.amazon.com/aws-mobile/latest/developerguide/web-access-apis.html), PUT method can be used to create or update records. It updates the record if a matching record is found. Otherwise, a new record is created.

```javascript
let apiName = 'MyApiName'; // replace this with your api name.
let path = '/path'; // replace this with the path you have configured on your API
let myInit = {
    body: {}, // replace this with attributes you need
    headers: {} // OPTIONAL
}

API.put(apiName, path, myInit).then(response => {
    // Add your code here
}).catch(error => {
    console.log(error.response)
});
```

Example with async/await:

```javascript
async function putData() { 
    let apiName = 'MyApiName';
    let path = '/path';
    let myInit = { // OPTIONAL
        body: {}, // replace this with attributes you need
        headers: {} // OPTIONAL
    }
    return await API.put(apiName, path, myInit);
}

putData();
```

Update a record:

```javascript
const params = {
    body: {
        itemId: '12345',
        itemDesc: ' update description'
    }
}
const apiResponse = await API.put('MyTableCRUD', '/manage-items', params);
```

#### **DELETE**

```javascript
let apiName = 'MyApiName'; // replace this with your api name.
let path = '/path'; //replace this with the path you have configured on your API
let myInit = { // OPTIONAL
    headers: {} // OPTIONAL
}

API.del(apiName, path, myInit).then(response => {
    // Add your code here
}).catch(error => {
    console.log(error.response)
});
```

Example with async/await

```javascript
async function deleteData() { 
    let apiName = 'MyApiName';
    let path = '/path';
    let myInit = { // OPTIONAL
        headers: {} // OPTIONAL
    }
    return await API.del(apiName, path, myInit);
}

deleteData();
```

#### **HEAD**

```javascript
let apiName = 'MyApiName'; // replace this with your api name.
let path = '/path'; //replace this with the path you have configured on your API
let myInit = { // OPTIONAL
    headers: {} // OPTIONAL
}
API.head(apiName, path, myInit).then(response => {
    // Add your code here
});
```

Example with async/await:

```javascript
async function head() { 
    let apiName = 'MyApiName';
    let path = '/path';
    let myInit = { // OPTIONAL
        headers: {} // OPTIONAL
    }
    return await API.head(apiName, path, myInit);
}

head();
```

### Custom Request Headers

When working with a REST endpoint, you may need to set request headers for authorization purposes. This is done by passing a `custom_header` function into the configuration:

```javascript
Amplify.configure({
  API: {
    endpoints: [
      {
        name: "sampleCloudApi",
        endpoint: "https://xyz.execute-api.us-east-1.amazonaws.com/Development",
        custom_header: async () => { 
          return { Authorization : 'token' } 
          // Alternatively, with Cognito User Pools use this:
          // return { Authorization: (await Auth.currentSession()).idToken.jwtToken } 
        }
      }
    ]
  }
});
```

### Unauthenticated Requests

You can use the API category to access API Gateway endpoints that don't require authentication. In this case, you need to allow unauthenticated identities in your Amazon Cognito Identity Pool settings. For more information, please visit [Amazon Cognito Developer Documentation](https://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html#enable-or-disable-unauthenticated-identities).

## Customization

### Customizing HTTP Request Headers

To use custom headers on your HTTP request, you need to add these to Amazon API Gateway first. For more info about configuring headers, please visit [Amazon API Gateway Developer Guide](http://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-cors.html)

If you have used Amplify CLI to create your API, you can enable custom headers by following above steps:  

1. Visit [Amazon API Gateway console](https://aws.amazon.com/api-gateway/).
3. On Amazon API Gateway console, click on the path you want to configure (e.g. /{proxy+})
4. Then click the *Actions* dropdown menu and select **Enable CORS**
5. Add your custom header (e.g. my-custom-header) on the text field Access-Control-Allow-Headers, separated by commas, like: 'Content-Type,X-Amz-Date,Authorization,X-Api-Key,X-Amz-Security-Token,my-custom-header'.
6. Click on 'Enable CORS and replace existing CORS headers' and confirm.
7. Finally, similar to step 3, click the Actions drop-down menu and then select **Deploy API**. Select **Development** on deployment stage and then **Deploy**. (Deployment could take a couple of minutes).


## Using Modular Imports

If you only need to use API, you can run: `npm install @aws-amplify/api` which will only install the API module.
Note: if you're using Cognito Federated Identity Pool to get AWS credentials, please also install `@aws-amplify/auth`.
Note: if you're using Graphql, please also install `@aws-amplify/pubsub`

Then in your code, you can import the Api module by:

```javascript
import API from '@aws-amplify/api';

API.configure();

```

## API Reference   

For the complete API documentation for API module, visit our [API Reference](https://aws-amplify.github.io/amplify-js/api/classes/apiclass.html)
{: .callout .callout--info}
