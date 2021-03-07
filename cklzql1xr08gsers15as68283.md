## Add authorization to your Serverless APIs using AWS Cognito ☁

This is a continuation of the Serverless series I've been working on. If you haven't read my previous articles from this series, you can read them  [here](https://tejanshrana.hashnode.dev/). I would recommend going over at least the previous  [article](https://tejanshrana.hashnode.dev/build-your-first-serverless-application) since this article refers to the components we built there.

In the last article, we built a Serverless TODO application with 2 APIs - Get Tasks and Add Tasks. However, these APIs were open to the internet and thus anyone who had the endpoints could use these APIs. This is not the behavior we want since these APIs should only be accessible to our users. In this article, we will see how you can add authorization to these APIs so only the "logged in" users are allowed to use our application. In order to achieve this, we will be using AWS Cognito which is a user identity service. This service will allow us to register users and then let them log in to our application.

### Authentication vs Authorization

First things first, if you are confused between authentication and authorization, let me clarify the difference in simple terms.

- Authentication - This is the process of confirming whether the user is really who they claim they are. This is done using various methods like asking the user to enter their username and password, one-time password, biometrics, etc. In simpler terms, this is the `login` part.

- Authorization - This is the process of allowing access to a resource. For example, in your application, you may have a paid plan where users who pay $2 a month are given access to some special features, and the ones who don't are given access to basic features only. This process of giving access to some resources/features is `authorization`. You can imagine `authentication` as the precursor to this step. So, once the user is logged in, you check what plan they are on and then `authorize` them to access a few features.

So you can see that in order to `authorize` the users to access our TODO app, we will first have to `authenticate` them. Don't worry, we will cover them both in this article 😎

### What do you need to build this application?

1. A Serverless application that has its APIs exposed via AWS API Gateway. If you don't have one, you can build one by following this  [article](https://tejanshrana.hashnode.dev/build-your-first-serverless-application).

2. AWS command-line interface. You can refer to  [this](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)  page for instructions on how you can install it on your machine.

### Let's Build

You might remember this diagram from the last article. I've annotated the `Cognito` box as 2 to outline the scope of this article:

![architecture.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1615141102446/bYVL0O-2s.jpeg)

First, let's login into our AWS account and go to the Cognito service. A quick reminder that you can search for any AWS service by pressing `alt + S`. On the landing page of Cognito, you will see 2 options - `Manage User Pools` and `Manage Identity Pools`. Click on the `Manage User Pools` button. If you want to know what the difference is between the two options, you can read more about it  [here](https://aws.amazon.com/premiumsupport/knowledge-center/cognito-user-pools-identity-pools/#:~:text=Short%20Description,for%20authorization%20(access%20control).

On the next page, you will see a button on the top right corner to create a user pool. Click on this button and enter a name for the user pool. I am calling it `todo-user-pool`. Next, click on the `Review Defaults` button. This is a quick way to create a user pool wherein AWS chooses most of the options for us and we can review it to fit our use case. After clicking the review defaults button, you can leave all the options on the next page as is. Before creating the pool, we just want to add an app client, we will need this to allow our client (frontend, mobile app, etc.) to access this user pool and let users log in. To add an app client, you can click on the `Add app client...` option as shown in the screenshot below:

![userpool-config.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1615142285566/Ny3hmErBM.png)

On the next page, there are a bunch of options to configure our app client. First, let's give our client a name. I am calling this client `todo-app-client`. Next, uncheck the `Generate client secret` option and then check all options under the `Auth Flows Configuration`. Here's a screenshot of what the app client configuration should look like:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1615142692305/FpmXKK8JV.png)

Now you can click on the `Create app client` button. On the next page, you will see the app client listed. There's no save button here and it's not very intuitive what to do, but you just need to click on the `Return to pool details` link, and the app client will be ready. This will take you to the user pool creation page with the details filled in and the app client added as well. Great, our user pool is now ready to be created. Go ahead and click on the `Create pool` button and the user pool is now ready! 🙌

Okay, if you are confused as to what this means, let me break it down. We have basically created a pool where we can now add our users who can access our TODO app. We have also created an app client for this pool so we can add features like registration and login to our frontend. When the user logs in successfully, Cognito returns a JWT token which is used to access the APIs.

Our authorization function will sit in the AWS API Gateway which exposes our APIs. We get this authorization function out of the box in AWS API Gateway and it is called `Authorizer`. This authorizer checks the JWT token sent in the request header. If it is valid, it allows the request to go to Lambda. If it is invalid, it returns an error response.

Let's add this functionality now. Go to the API Gateway service from your AWS console. We will now create APIs to expose the Lambda functions exactly as we did in the last article.  Create an API and before adding methods (this is just my preference, you can create the methods first if you prefer that), we will first create the Authorizer.  To do so, click on `Authorizers` option in the left pane. You can refer to the screenshot below to find this option. Note that in the screenshot you can see that I don't have any methods added yet. 

![APIGW-preconfig.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1615145190416/FM_tIvbQ0.png)

On the next page, click on `Create New Authorizer` button. This will open up a few options to configure the authorizer. First, let's give the authorizer a name. I am calling mine `todo-authorizer`. Next, there are 2 radio buttons to choose the type of authorizer - Lambda and Cognito. We will choose Cognito here. A Lambda authorizer is used when we are using a Lambda function with some custom logic to validate the request. Since we are using Cognito as the identity service in our application, we can leverage the built-in feature to validate the token issued by Cognito.

Next, we need to add the name of the user pool we want to use. To do that, click on the text box under `Cognito User Pool` and it will show you the name of the user pool we had created earlier. You can select that. Now, under `Token Source` we need to add the name of the header wherein the token will be passed while calling our APIs. We will use the `Authorization` header for this, so you can add `Authorization` in the text box. `Token Validation` can be left empty since it is a non-mandatory field and we will not be using it. Your authorizer configuration should look like below:


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1615146185530/t1TYEHKuN.png)

Great, now click on `Create` and our authorizer is ready! 

Well done. We have everything we need. Now it's time to tie things together. We want to create our endpoints and attach the authorizers to them as well as the Lambda functions. To do so, let's create the methods and attach our Lambda functions. If you don't remember how to do that, don't worry, you can go back and refer to the  [last article](https://tejanshrana.hashnode.dev/build-your-first-serverless-application#the-gateway)  where we did this. 

Before deploying, we will attach the authorizers to our methods now. If you click on any one (GET or POST) of the methods you have just created, you will see a few options show up on the right like below:

![APIGW-configured.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1615146977434/VjOZm1nHa.png)

Now, click on `Method Request` that I have highlighted above. This will open a few more options and the first one should be `Settings`, under which you would see `Authorization` with a drop-down next to it. By default, this would be set to `NONE`. When you click on the drop-down, the authorizer we had created earlier should show up here. Select the one you had created and click on the ✔ button next to it. Mine was called `todo-authorizer` so I will select that. Now, repeat the same steps to attach the authorizer to the other method as well. You should now have the authorizer attached to both the GET and POST method. 

We are nearly there. Now, deploy these APIs by clicking on `Actions` > `Deploy API`. In the dialog box, choose [New Stage]  for `Deployment stage` and add a name for it. I am calling it `todo`. Now, click on `Deploy`. We have now successfully deployed our APIs with the authorizers attached! 🚀

To validate that the authorizer works, click on the URL that API Gateway has created for you. Refer to the screenshot below to find that URL:

![APIGW-url.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1615147673118/lH8EN5Ste.png)

If you click on that URL, it should open a new tab and you should see 

```
{"message":"Unauthorized"}
``` 
on that page. This is exactly what we want to see. Why? We are not logged in to our TODO app and thus the authorizer is not letting us access the API. 🙂


### Creating a user

Okay, so we have tested that the authorizer does not allow us to access the APIs when we are not logged in. Now, we also want to test that it allows us to access the API when we are logged in. To do that, we first need to create a user in the user pool we have created. This is essentially the "registration" step but we are doing it via AWS console in this article.

Go to the Cognito service from your AWS Console and click on `Manage User Pools`. This should open a page showing the user pool you created earlier. Click on that and you should not see all the details of your user pool listed like below:


![cognito-configured.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1615148314101/ChxUPFLCm.png)


Click on the `Users and groups` option as highlighted in the screenshot above. This will open a new page where we can create a user. Click on `Create user` and a dialog box will open up. Here, add your email address as the `Username`, add a temporary password, and again in the `Email` field enter the same email address as you've entered in `Username`, uncheck the `Mark phone number as verified?` option and click on `Create User`. If you see an error message like `Password did not conform with password policy`, you will need to change the password. While creating the user pool, we choose to follow the default settings which creates a rule for a complex password. This is also highly recommended to ensure that users create a strong password. 

After creating the user, you will see the user with `Account status` as `FORCE_CHANGE_PASSWORD` and you should also receive an email from AWS with the temporary password. Now, we have 2 things to do:

1. Confirm the user by changing the password.
2. Login with the user.

We will be doing this via AWS CLI. Note that you will need your AWS keys configured in your environment to do this. If you are not sure how to do this, you can follow along.

First, go to IAM service from your AWS console and click on Users > Add User.  Give a name to your user, and check the box with `Programmatic access` since we are creating this user to be able to access Cognito from the command line. Click Next and on the next page click on `Attach existing policies directly` and search for `Cognito`. An option called `AmazonCognitoPowerUser` should show up. Check this option and click Next and Create User. You now have a user created. On the final page, you will see your `Access key ID` and `Secret access key`. Copy these values and paste them somewhere as we need to use them in a minute. 

Now, follow the below steps on your command line:

1. Run `aws configure` command.
2. It would ask you to enter the Access Key ID first. Paste the one you had copied after creating the user in IAM.
3. Enter the Secret Access Key. This is the key that was hidden when the keys were created in IAM. 
4. Enter the default region name. This is the region where you have created your components. I have created mine in Ireland region, so I will enter `eu-west-1`. 
5. Enter the output format as `json`.

You can also find this information [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html). Your CLI is now ready to use.

Now, before we use Cognito from CLI, let's first get the properties we will need - user pool ID and client ID. When you go to your user pool in Cognito, you will see the pool ID listed as the first value, copy this somewhere. You can grab the client ID by clicking on `App Clients` option on the left pane. Refer to the below screenshot if need be:


![cognito-pool-details.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1615150881845/ikfy7UYk-.png)

Now, there are two ways of passing these values in the AWS CLI - passing them as individual values or passing them all in a JSON file. I prefer the latter since it is cleaner that way, IMO. So, let's create two JSON files now. The first one will be called `auth.json` and should have the following keys and their respective values:


```
{
    "UserPoolId": "your-pool-id",
    "ClientId": "your-client-id",
    "AuthFlow": "ADMIN_NO_SRP_AUTH",
    "AuthParameters": {
        "USERNAME": "the-username-you-had-created-earlier",
        "PASSWORD": "the-temporary-password-you-received-in-the-email"
    }
}
``` 

The second one will be called `challenge-response.json` and should have the following structure:


```
{
    "UserPoolId": "your-pool-id",
    "ClientId": "your-client-id",
    "ChallengeName": "NEW_PASSWORD_REQUIRED",
    "ChallengeResponses": {
        "USERNAME": "the-username-you-had-created-earlier",
        "NEW_PASSWORD": "a-new-password"
    },
    "Session": "ignore-this-for-now"
} 

``` 
Great, we are now ready to log in. Use the following command from where your JSON files are stored to log in using AWS CLI - `aws cognito-idp admin-initiate-auth --cli-input-json file://auth.json`. We are not logged in yet. Remember we set a temporary password while creating the user. So Cognito will first force us to change this temporary password. Thus, that command should return a JSON response with a key called `Session` which we will use to set our permanent password. Copy the value of this `Session` attribute and paste it in the `challenge-response.json` file for the `Session` key which we previously ignored. Now, to confirm this user and create a permanent password, run the following command from AWS CLI - `aws cognito-idp admin-respond-to-auth-challenge --cli-input-json file://challenge_response.json`. And now, we are logged in! Going forward, you can just use the first command to log in but first, remember to change the password to the new password in the `auth.json` file. 🙂

### Testing

Now, let's test and ensure that we are able to access our APIs once we are logged in. To do so, we will be using Postman. Create a new GET request in Postman with the URL that API Gateway had generated for us. Next, add a header called `Authorization`. For the value of this header, just run the Cognito login command from AWS CLI. This will return a JSON with a key called `ID Token`. Copy the value of this key and paste it as the value for the `Authorization` header. 

Now, when you send the request, you should be able to see a successful response with the tasks being returned like below:


![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1615152449131/gdsXqfpiw.png)


You have now successfully built your first Serverless application and added authentication and authorization to it. 🚀

This is the last article from this series. If you wish to create a frontend for your TODO app and add authentication, you can do so using AWS Amplify as detailed  [here](https://docs.amplify.aws/lib/auth/getting-started/q/platform/js). Alternatively, if you would like me to cover that in another article, please feel free to comment below and I will be happy to do that. Keep an eye on this space and I plan to post a lot of exciting stuff soon. 🙂 