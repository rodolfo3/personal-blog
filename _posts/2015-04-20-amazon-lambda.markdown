---
layout: post
title:  "Playing with Amazon Lambda"
date:   2015-04-20 00:00:00 -3
categories: aws
---

[Lambda] is a (preview) service from Amazon to allow run automatically code based on some events. This works like the "Observer" pattern. In this example, I will write a simple script to notify (send an e-mail) when an file is uploaded to [S3]. This is only one example: lambda can get events from other (Amazon) sources, like DynamoDB or Kinesis. This can be used to build an mobile app allowing sync data without get your own server (some guys showed it on [QCon SP 2015][mobile-sem-servidores]).

The idea to test the service is load e-mail address from a json file called `to-notify.json` from the same folder of the uploaded file and send an e-mail to them. The e-mail should contains a link to download the file.


## Hello lambda ##
The first time accessing the Lambda console, it shows some example code (like the Hello World). The code, basically, exports a function and call it when appropriate:

``` javascript
exports.handler = function(event, context) {
    console.log(event);
}
```

The `handler` name is not fixed (it can be changed in "Handler name"). One required step is create a "role", to grant necessary permissions to run the code. By default, there is no one. Clicking in the button "Create/Select Role", create a new one with default values. Back to the lambda console, create the `HelloWorld` function (select it on the "Code Template").

To try it, on the list of functions, click on "Edit/Test" button. In the new page, select an sample event and click on "Invoke". The result will appear in the bottom.


## Setup AWS js sdk locally ##

Lambda (for now) only accepts JavaScript functions. To code locally, just install the `aws-swk` npm package:

``` bash
$ npm install aws-sdk
```

The `aws-sdk` package allows get files from [S3] and send e-mails using [SES] - and much more, as you can check in the [docs][aws-sdk].

To run the code locally, is required to generate the access keys on Amazon. Do it on the console, clicking the user name (on the right-top), go to `Security Credentials`, select `Access Keys` and click `Create New Access Key`. Download it or copy the generated value. Then, create a config file with the data:

``` json
{
    "accessKeyId": "<your key>",
    "secretAccessKey": "<your secret>",
    "region": "us-east-1"
}
```

This is only to run locally - Lambda will do it for you in the real world.

The code to send an e-mail is:

``` javascript
    AWS = require("aws-sdk");

    AWS.config.loadFromPath('./config.json'); // the file with your keys
    var s3 = new AWS.S3();

    var params = {
      Destination: { /* required */
        ToAddresses: 'your@email.com'
      },
      Message: { /* required */
        Body: { /* required */
          // Html: {
          //   Data: 'STRING_VALUE', /* required */
          //   Charset: 'STRING_VALUE'
          // },
          Text: {
            Data: 'Report ' + reportName + ' is ready: ' + reportUrl,
            Charset: 'UTF-8'
          }
        },
        Subject: { /* required */
          Data: 'A new report ' + reportName + ' is ready!', /* required */
          Charset: 'UTF-8'
        }
      },
      Source: 'noreply@email.com', /* required */
      // ReplyToAddresses: [
      //   'noreply@email.com',
      // ],
      // ReturnPath: 'STRING_VALUE'
    };

    ses.sendEmail(params, function(err, data) {
      if (err) {
        // an error occurred
        console.log(err, err.stack);
      } else {
          // successful response
          console.log("notified!");
      }
    });
```

Run it and probably get an error like:

```
{ [MessageRejected: Email address is not verified.]
  message: 'Email address is not verified.',
  code: 'MessageRejected',
```

To fix this, verify the domain or e-mail as a sender on [SES] console, in the `Verifyed senders` menu items. Running it againd, it should work.

Congrats! You are able to send e-mails using Amazon [SES].

## Catching events ##

The S3 put event provide some useful information, some special to get the uploaded file:

``` json
{
  "Records": [
    {
      (...)
      "s3": {
        (...)
        "bucket": {
          "name": "sourcebucket",
          (...)
        },
        "object": {
          "key": "HappyFace.jpg",
          (...)
        }
      }
    }
  ]
}
```

To start out Lambda function, create a `index.js` following the same structure from "hello world".
Open the [S3] console and create a new bucket, if there is no one there yet. It should be in the same region of your lambda function. In "properties" tab, select "Events" and set "ObjectCreated" in the "Event" field. The name of lambda function should appear in a "select box" widget. Select it and save it. Upload a file to test and check the lambda console - the execution should be logged there.

It works!
The logs (after reload the lambda console) show the data from the event.


## Putting everything together ##

To send the e-mail, just put code a `handle` function to receive the event, get the bucket and folder name, fetch the file with the destination address and make a call to the [SES]. My code is [here][github].

Then, zip the `index.js` file and `node_modules` folder (with the dependencies). Go to lambda console and upload it, creating a new function.
The function will not work if the selected `Role` does not allow to use [SES]. The error will be like (on the lambda console, there is a link to the errors, is useful to debug):

```
`arn:aws:sts::089152892364:assumed-role/lambda_exec_role/awslambda_618_20150420010051044' is not authorized to perform `ses:SendEmail' on resource `arn:aws:ses:us-west-2:089152892364:identity/rodolfo.3@gmail.com'] message: 'User `arn:aws:sts::089152892364:assumed-role/lambda_exec_role/awslambda_618_20150420010051044\' is not authorized to perform `ses:SendEmail\' on resource `arn:aws:ses:us-west-2:089152892364:identity/rodolfo.3@gmail.com\'', code: 'AccessDenied',
```

To fix this, go to [IAM] console. Select `Role` in the menu and create a new one.
Follow the steps until `Attach Policy`. Then, select the `AWS Lambda` option and mark `AmazonS3ReadOnlyAccess` (to get the addresses to send e-mails) and `AmazonSESFullAccess` (to send the e-mail).

Back to the lambda console, select this role to the function (in `Change function configuration and role`, when editing it).


Not it's ready!
Upload a file and see it working.

## Conclusion ##

Lambda can be very useful as a tool to integrate some distribute processes.
If you have some scheduled script (like in a contab) to generate some files and upload it to the S3, Lambda can be a useful tool to automate the rest of the process (like convert files, notify people, create thumbnails, etc).
The only "problem" is write everything in JavaScript - but I believe it is better then write bash files ;)

[Lambda]: https://aws.amazon.com/lambda/
[S3]: https://aws.amazon.com/s3/
[mobile-sem-servidores]: http://qconsp.com/presentation/Sem-Servidores-Mobile-Backend-as-a-Service-na-plataforma-AWS
[aws-sdk]: https://www.npmjs.com/package/aws-sdk
[SES]: https://aws.amazon.com/ses/
[github]: https://github.com/rodolfo3/lambda-s3-file-notifier
[IAM]: https://console.aws.amazon.com/iam/home
