# Cloud-enabled Amplify DataStore workshop using Angular

In this workshop we'll learn how to use Amplify DataStore to create `Chatty` a single room realtime multi-user chat app using Angular 9 & [AWS Amplify](https://aws-amplify.github.io/).

![](./header.png)

### Topics we'll be covering:

- [Authentication](#adding-authentication)
- [GraphQL API with AWS AppSync](#adding-a-graphql-api)
- [Setup Amplify DataStore](#setup-amplify-datastore)
- [Deploying via the Amplify Console](#deploying-via-the-amplify-console)
- [Removing / Deleting Services](#removing-services)
- [Appendix and trobleshooting](#appendix)

## Pre-requisites

- Node: `13.12.0`. Visit [Node](https://nodejs.org/en/download/current/)
- npm: `6.14.4`. Packaged with Node otherwise run upgrade

```bash
npm install -g npm
```

## Getting Started - Creating the Application

To get started, we first need to create a new Angular project & change into the new directory using the [Angular CLI](https://cli.angular.io/).

If you already have it installed, skip to the next step. If not, either install the CLI & create the app or create a new app using:

```bash
npm install -g @angular/cli
ng new amplify-datastore
```
 
Now change into the new app directory and make sure it runs

```bash
cd amplify-datastore
npm install
ng serve
```
## Changes to Angular CLI project

Add type definitions for __Node__ by changing `tsconfig.app.json`. This is a requirement from `aws-js-sdk`.

```json
{
  "compilerOptions": {
    "types": ["node"]
  },
}
``` 

Add the following code, to the top of `src/polyfills.ts`. This is a requirement for Angular 6+.

```js
(window as any).global = window;

(window as any).process = {
  env: { DEBUG: undefined }
};
```

## Installing the CLI & Initializing a new AWS Amplify Project

Let's now install the AWS Amplify API & AWS Amplify Angular library:

```bash
npm install --save aws-amplify aws-amplify-angular moment
```
> If you have issues related to EACCESS try using sudo: `sudo npm <command>`.

### Installing the AWS Amplify CLI

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```
> If you have issues related to fsevents with npm install. Try: `npm audit fix --force`.

Now we need to configure the CLI with our credentials:

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __eu-west-2 (London)__
- Specify the username of the new IAM user: __amplify-datastore__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
  accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
  secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __default__

> To view the new created IAM User go to the dashboard at [https://console.aws.amazon.com/iam/home#/users/](https://console.aws.amazon.com/iam/home#/users/). Also be sure that your region matches your selection.

### Initializing A New Project

```bash
amplify init
```

- Enter a name for the project: __amplify-datastore__
- Enter a name for the environment: __dev__
- Choose your default editor: __Visual Studio Code__   
- Please choose the type of app that you're building __javascript__   
- What javascript framework are you using __angular__   
- Source Directory Path: __src__   
- Distribution Directory Path: __dist/amplify-datastore__   
- Build Command: __npm run-script build__   
- Start Command: __ng serve__
- Please choose the profile you want to use: __default__
- Do you want to use an AWS profile? __Yes__
- Please choose the profile you want to use __default__

Now, the AWS Amplify CLI has iniatilized a new project & you will see a new folder: __amplify__. The files in this folder hold your project configuration.

```bash
<amplify-datastore>
    |_ amplify
      |_ .config
      |_ #current-cloud-backend
      |_ backend
      team-provider-info.json
```

## Adding Authentication

In order to display the user sending messages for our chat, we will require users to register and login. We can implement this requirement using the `auth` category.

To add authentication to our Amplify project, we can use the following command:

```sh
amplify add auth
```

> When prompted choose 
- Do you want to use default authentication and security configuration?: __Default configuration__
- How do you want users to be able to sign in when using your Cognito User Pool?: __Username__
- Do you want to configure advanced settings? __Yes, I want to make some additional changes.__
- What attributes are required for signing up? (Press &lt;space&gt; to select, &lt;a&gt; to 
toggle all, &lt;i&gt; to invert selection): __Email__
- Do you want to enable any of the following capabilities? (Press &lt;space&gt; to select, &lt;a&gt; to toggle all, &lt;i&gt; to invert selection): __None__

> To select none just press `Enter` in the last option.

Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push

Current Environment: dev

| Category | Resource name      | Operation | Provider plugin   |
| -------- | ------------------ | --------- | ----------------- |
| Auth     | amplifyappuuid     | Create    | awscloudformation |
? Are you sure you want to continue? Yes
```

To quickly check your newly created __Cognito User Pool__ you can run

```bash
amplify status
```

> To access the __AWS Cognito Console__ at any time, go to the dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

### Configuring the Angular Application

Now, our resources are created & we can start using them!

The first thing we need to do is to configure our Angular application to be aware of our new AWS Amplify project. We can do this by referencing the auto-generated `aws-exports.js` file that is now in our `src` folder.

To configure the app, open __main.ts__ and add the following code below the last import:

```js
import Auth from '@aws-amplify/auth';
import amplify from './aws-exports';
Auth.configure(amplify);
```

Now, our app is ready to start using our AWS services.

### Importing the Angular Module

Add the Amplify Module and Service to `src/app/app.module.ts`:

```js
import { AmplifyAngularModule, AmplifyService } from 'aws-amplify-angular';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AmplifyAngularModule
  ],
  providers: [
    AmplifyService
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

### Using Amplify Service

The `AmplifyService` provides access to AWS Amplify core categories via Dependency Injection: auth, analytics, storage, api, cache, pubsub; and authentication state via Observables.

### Using the Authenticator Component

AWS Amplify provides UI components that you can use in your App. Let's add these components to the project

```bash
npm i --save @aws-amplify/ui
```

Also include these imports to the top of `styles.css`

```css
@import "~@aws-amplify/ui/src/Theme.css";
@import "~@aws-amplify/ui/src/Angular.css";
```

In order to use the Authenticator Component replace all content in __src/app.component.html__ with:

```html
<amplify-authenticator></amplify-authenticator>
```

Now, we can run the app and see that an Authentication flow has been added in front of our App component. This flow gives users the ability to sign up & sign in.

> To view any users that were created, go back to the __Cognito__ dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

Alternatively we can also use

```bash
amplify console auth
```

### Accessing User Data

We can access the user's info now that they are signed in by calling `currentAuthenticatedUser()` which returns a Promise.

```js
import { Component } from '@angular/core';
import { AmplifyService } from 'aws-amplify-angular';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  constructor(public amplify: AmplifyService) {
    amplify.auth().currentAuthenticatedUser().then(console.log)
  }
}
```

## Adding a GraphQL API

To add a GraphQL API, we can use the following command:

```sh
amplify add api
```

Answer the following questions

- Please select from one of the below mentioned services __GraphQL__
- Provide API name: __ChattyAPI__
- Choose the default authorization type for the API __API key__
- Enter a description for the API key: __(empty)__
- After how many days from now the API key should expire (1-365): __180__
- Do you want to configure advanced settings for the GraphQL API __Yes, I want to make some additional changes.__
- Configure additional auth types? __N__
- Configure conflict detection? __Y__
- Select the default resolution strategy __Auto Merge__
- Do you want to override default per model settings? __N__
- Do you have an annotated GraphQL schema? __N__ 
- Do you want a guided schema creation? __Y__   
- What best describes your project: __Single object with fields (e.g. “Todo” with ID, name, description)__   
- Do you want to edit the schema now? __Y__

> To select none just press `Enter`.

> When prompted, update the schema to the following:   

```graphql
type Chatty @model {
  id: ID!
  user: String!
  message: String!
  createdAt: AWSDateTime
}
```

This will allow us to display each user messages together with the creation date and time.

> Note: Don't forget to save the changes to the schema file!

> Next, let's push the configuration to our account:

```bash
amplify push
```

- Are you sure you want to continue? __Yes__
- Do you want to generate code for your newly created GraphQL API __Yes__
- Choose the code generation language target __angular__
- Enter the file name pattern of graphql queries, mutations and subscriptions __src/graphql/**/*.graphql__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions __Yes__
- Enter maximum statement depth [increase from default if your schema is deeply nested] __2__
- Enter the file name for the generated code __src/app/API.service.ts__

Notice your __GraphQL endpoint__ and __API KEY__.

This step created a new AWS AppSync API. Use the command below to access the AWS AppSync dashboard. Make sure that your region is correct.

```bash
amplify console api
```

## Setup Amplify DataStore

### Installing the Amplify DataStore

Next, we'll install the necessary dependencies:

```bash
npm install --save @aws-amplify/core @aws-amplify/datastore
```

### Data Model Generation

Next, we'll generate the models to access our messages from our __ChattyAPI__

```bash
amplify codegen models
```

> Important: DO NOT forget to generate models every time you introduce a change in your schema.

Now, the AWS Amplify CLI has generated the necessary data models and you will see a new folder in your source: __models__. The files in this folder hold your data model classes and schema.

```bash
<amplify-app>
    |_ src
      |_ models
```

### Creating a message

Now that the GraphQL API and Data Models are created we can begin interacting with them!

The first thing we'll do is create a new message using the generated Data Models and save.

```js
import { DataStore } from "@aws-amplify/datastore";
import { Chatty } from "./models";

await DataStore.save(new Chatty({
  user: "amplify-user",
  message: "Hi everyone!"
}))
```

This will create a record locally in your browser and synchronise it in the background using the underlying GraphQL API. 

### Querying data

Let's now see how we can query data using Amplify DataStore. In order to query our Data Model we will use a query and a predicate to indicate that we want all records. 

```js
import { DataStore, Predicates } from "@aws-amplify/datastore";
import { Chatty } from "./models";

const messages = await DataStore.query(Chatty, Predicates.ALL);
```

This will return an array of messages that we can display in our UI.

Predicates also support filters for common types like Strings, Numbers and Lists.

> Find all supported filters at [Query with Predicates](https://aws-amplify.github.io/docs/js/datastore#query-with-predicates)

## Creating the UI

Now, let's look at how we can create the UI to create and display messages for our chat.

```js
import { DataStore, Predicates } from "@aws-amplify/datastore";
import { Chatty } from "../../models";

@Component({
  template: `
    <div>
      <form [formGroup]="createForm" (ngSubmit)="onCreate(createForm.value)">
        <input type="text" formControlName="message" placeholder="Enter your message...">
        <button type="submit">Send</button>
      </form>
      <div *ngFor="let message of messages">
        <div>{{ message.user }} - {{ moment(message.createdAt).format('YYYY-MM-DD HH:mm:ss')}})</div>
        <div>{{ message.message }}</div>
      </div>
    </div>`
})
export class AppComponent implements OnInit {
  messages: Array<Chatty>;

  ngOnInit() {
    this.loadMessages();
  }

  loadMessages() {
    DataStore.query<Chatty>(Chatty, Predicates.ALL)
    .then(messages => {
      this.messages = [...messages].sort((a, b) => -a.createdAt.localeCompare(b.createdAt));
    })
  }
}
```

## Creating a message

 Now, let's look at how we can create new messages.

```js
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component(...)
export class HomeComponent implements OnInit {
  public createForm: FormGroup;

  constructor(private fb: FormBuilder) {
    Auth.currentAuthenticatedUser().then(cognitoUser => {
      this.user = cognitoUser.username
    })
  }

  ngOnInit() {
    this.createForm = this.fb.group({
      'message': ['', Validators.required],
    });
    this.loadMessages();
  } 
  
  public onCreate(message: any) {
    if ( message.message=="" ) return;

    DataStore.save(new Chatty({
      user: this.user,
      message: message.message
    })).then(() => {
      console.log('message created!');
      this.createForm.reset();
      this.loadMessages();
    })
    .catch(e => {
      console.log('error creating message...', e);
    });
  }
}
```

## Deleting all messages

One of the main advantages of working using Amplify DataStore is being able to run batch mutations without having to use a series of individual operations. 

See below how we can use delete together with a predicate to remove all messages.

```js
DataStore.delete(Chatty, Predicates.ALL).then(() => {
  console.log('messages deleted!');
});
```

### GraphQL Subscriptions

Next, let's see how we can create a subscription to subscribe to changes of data in our API.

To do so, we need to listen to the subscription, & update the state whenever a new piece of data comes in through the subscription.

When the component is destroyed we will unsubscribe to avoid memory leaks.

```js
@Component(...)
export class HomeComponent implements OnInit, OnDestroy {
  subscription;

  ngOnInit() {
    this.subscription = DataStore.observe<Chatty>(Chatty).subscribe(msg => {
      console.log(msg.model, msg.opType, msg.element);
      this.loadMessages();
    });
  }

  ngOnDestroy() {
    if (!this.subscription) return;
    this.subscription.unsubscribe();
  }
}
```

## Deploying via the Amplify Console

We have looked at deploying via the Amplify CLI hosting category, but what about if we wanted continous deployment? For this, we can use the [Amplify Console](https://aws.amazon.com/amplify/console/) to deploy the application.

The first thing we need to do is [create a new GitHub repo](https://github.com/new) for this project. Once we've created the repo, we'll copy the URL for the project to the clipboard & initialize git in our local project:

```sh
git init

git remote add origin git@github.com:username/project-name.git

git add .

git commit -m 'initial commit'

git push origin master
```

Next we'll visit the Amplify Console in our AWS account at [https://ap-northeast-1.console.aws.amazon.com/amplify/home](https://ap-northeast-1.console.aws.amazon.com/amplify/home).

Here, we'll click __Get Started__ to create a new deployment. Next, authorize Github as the repository service.

Next, we'll choose the new repository & branch for the project we just created & click __Next__.

In the next screen, we'll create a new role & use this role to allow the Amplify Console to deploy these resources & click __Next__.

Finally, we can click __Save and Deploy__ to deploy our application!

Now, we can push updates to Master to update our application.

## Removing Services

If at any time, or at the end of this workshop, you would like to delete a service from your project & your account, you can do this by running the `amplify remove` command:

```sh
amplify remove auth
amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.


## Appendix

### Setup your AWS Account

In order to follow this workshop you need to create and activate an Amazon Web Services account. 

Follow the steps [here](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account)

### Trobleshooting

> Message: The AWS Access Key Id needs a subscription for the service

Solution: Make sure you are subscribed to the free plan. [Subscribe](https://portal.aws.amazon.com/billing/signup?type=resubscribe#/resubscribed)


> Message: TypeError: fsevents is not a constructor

Solution: `npm audit fix --force`


> Message: Cannot read property 'viewContainerRef' of undefined at AuthenticatorComponent...

Solution: [#4681](https://github.com/aws-amplify/amplify-js/issues/4681)

Replace package an occurrences of `aws-amplify-angular` with `@flowaccount/aws-amplify-angular`.

> Behaviour: data seems not to be synchronising with the cloud and or viceversa

Solution: 

```
amplify update api
amplify push
```

Make sure you answer the following questions as
- Configure conflict detection? __Y__
- Select the default resolution strategy __Auto Merge__
- Do you want to override default per model settings? __N__