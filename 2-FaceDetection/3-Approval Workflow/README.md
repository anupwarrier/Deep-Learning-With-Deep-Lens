# Approval Workflow (Step Functions, API GW and Approval Website)

## Create IAM Roles for API Gateway and Step Functions

### Create IAM Role for API Gateway to Step Functions

_As seen in the architecture diagram, API gateway will be used to invoke the 2nd Step Function.  We need to create an IAM role to allow this to happen._

1.	To access the IAM Console: In the AWS Console, click on “Services” in the top, left-hand corner of the screen, and click on “IAM” (you can find it by typing _iam_ into the search field at the top of the screen).
2.	On the left-hand side of the screen, click "Roles", and then click “Create Role”.
2.	Click “AWS service” and click “API Gateway”.
3.	Click “Next: Permissions” (at the bottom of the screen).
4.	Click “Next: Review” (at the bottom of the screen).
5.	In the “Role name” text box, type _APIGatewayToStepFunctions_
6.	Click “Create role” (at the bottom of the screen).
7.	On the left-hand side of the screen, click "Roles", search for your role (APIGatewayToStepFunctions) and then click on that role.
8.	On the Permissions tab, choose "Attach Policy".
9.	On the Attach Policy page, type _step_ into the search field and click the checkbox to the left of "AWSStepFunctionsFullAccess".
10.	Click "Attach Policy".


### IAM Role for Step Functions to Lambda

_As seen in the architecture diagram, Step Functions will be used to invoke the PostApproval Lambda function.  We need to create an IAM role to allow this to happen._

1.	In the IAM Console, on the left-hand side of the screen, click "Roles", and then click “Create Role”
2.	Click “AWS service” and click “Step Functions”.
3.	Click “Next: Permissions” (at the bottom of the screen).
4.	Click “Next: Review” (at the bottom of the screen).
5.	In the “Role name” text box, type _StepFunctionstoLambda_
6.	Click “Create role” (at the bottom of the screen).

## Configure Step Functions

_We will use the AWS Step Functions service to define and control our overall workflow.  (For more information: https://aws.amazon.com/step-functions)_

1.	In the AWS Console, click on “Services” in the top, left-hand corner of the screen, and click on “Step Functions” (you can find it by typing _step_ into the search field at the top of the screen).
2.	Click "Get Started", and then click "Author from scratch".
3.	For the name, type _MLApprovalProcess_ (see screenshot below).
4.	For the IAM Role, ensure that the APIGatewayToStepFunctions role is selected.
5.	In the "Code" section, paste the following code and **replace the strings 111111111111 with your own AWS account Number**:
```{
  "Comment": "Image review process!",
  "StartAt": "ManualApproval",
  "States": {
    "ManualApproval": {
      "Type": "Task",
      "Resource": "arn:aws:states:us-east-1:111111111111:activity:ManualStep",
      "Next": "PostApproval"
    },
    "PostApproval": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:111111111111:function:PostApproval",
      "End": true
    }
  }
}
```

![](images/Step_fns.png)

6.	Click "Create State Machine" (at the bottom of the page).
7.	On the left-hand side of the screen, click "Tasks", then click "Create new activity".
8.	Type ManualStep in the "Activity Name" textbox, and then click "Create Activity".


### Test Step Functions

1.	While still in the step functions console, on the left-hand side of the screen, click “Dashboard”.

![](images/Test_step_fns.png)

2.	Click directly on the state machine you just created (not on the radio button next to it) and then click “New execution”.
3.	Enter a name for the execution test and then click “Start execution” (see screenshot below).

![](images/State_machines.png)

4.	This will simply go into the ManualApproval activity for now:

![](images/New_execution.png)

5.	Now click “Stop execution” in the top, right-hand corner of the screen.

## Configure API Gateway

1.	Save the following Swagger file to your computer: [API Gateway Swagger File](./APIGatewayToStepFunctions-respond-swagger-apigateway.yaml), open the file for editing, and substitute all instances of the string **111111111111** with your own AWS account number.
2.	In the AWS Console, click on “Services” in the top, left-hand corner of the screen, and click on “API Gateway” (you can find it by typing api into the search field at the top of the screen).
3.	Click "Get Started", and then click "OK".
4.	Select the "Import from Swagger" option (see screenshot below).

![](images/Create_new_api.png)

5.	Click “Select Swagger File” and upload the swagger file you created in step 1 above.
6.	Click "Import" (in the bottom, right-hand corner of the screen).
7.	In the "Resources" section for your API, click the "Actions" drop-down menu and select "Deploy API". Enter the following details:
o	 Deployment stage: [New Stage]
o	Stage name: respond
8.	Click "Deploy".
9.	Note the "Invoke URL" that gets created for this API.  Copy this to a text file because you will need it in a later step (see example screenshot below).

![](images/Invoke_url.png)

10.	Paste the invoke URL into a browser tab to ensure that the API is responding. For now it will just return an error saying “{"message":"Missing Authentication Token"}”, which is expected at this point, because the request has not gone through the expected end-to-end workflow.

## Approval Verification Website

### Create Cognito Identity Pool

_Cognito will be used to assign temporary credentials for securely accessing AWS resources used in this workshop. (For more information: https://aws.amazon.com/cognito/)_

1.	In the AWS Console, click on “Services” in the top, left-hand corner of the screen, and click on “Cognito” (you can find it by typing _cog_ into the search field at the top of the screen).
2.	Click “Manage Federated Identities”, and then click “Create new identity pool”.
3.	For “Identity pool name”, enter _ML_ID_Pool_
4.	Select “Enable access to unauthenticated identities” from the “Unauthenticated identities” collapsible section.
5.	Click “Create Pool”.
6.	In the screen that appears, click “Allow” (in the bottom, right-hand corner of the screen).
7.	Note the identity pool ID that is displayed in the center of the screen (please see the following example screenshot).

![](images/Cognito_ID_Pool_ID.png)

8.	Copy that into a text file because you will use it in a later step.

### Update the Cognito IAM Role to Allow Access to AWS Resources

1.	In the AWS Console, click on “Services” in the top, left-hand corner of the screen, and click on “IAM” (you can find it by typing _iam_ into the search field at the top of the screen).
2.	On the left-hand side of the screen, click "Roles".
3.	In your list of roles, click on “Cognito_ML_ID_PoolUnauth_Role”, and click “Attach policy”.
4.	In the Search field, type _s3_, and then select “AmazonS3FullAccess” (i.e. click the checkbox to the left of “AmazonS3FullAccess”).
5.	In the Search field, type _step_, and then select “AWSStepFunctionsFullAccess”.
6.	In the Search field, type _rek_, and then select “AmazonRekognitionFullAccess”.
7.	In the Search field, type _dyn_, and then select “AmazonDynamoDBFullAccess”.
8.	In the Search field, type _ses_, and then select “AmazonSESFullAccess”.
9.	In the Search field, type _api_, and then select “AmazonAPIGatewayInvokeFullAccess”.
10.	Click “Attach policy” (at the bottom, right-hand corner of the screen).
11.	Repeat steps 1 to 10 for the “Cognito_ML_ID_PoolAuth_Role”.

### Create S3 Bucket for Static Website Hosting

_We will use a static website to host a web-page that will be used for approving unrecognized faces to be added to our Rekognition collection._

1. In the AWS Console, click on “Services” in the top, left-hand corner of the screen, and click on “S3” (you can find it by typing _s3_ into the search field at the top of the screen).
2. Click "Create bucket", and enter the following details:
*	Bucket name: [Your name or username]-web
*	Region: US East-1 (Virginia)
2.	Click "Create".
3.	Now, in your list of S3 buckets, click on the bucket you just created (i.e. [Your name or username]-web).
4.	Click on the "Properties" tab and click on "Static website hosting".
5.	Select the "Use this bucket to host a website" option.
6.	In the "Index document" textbox, type index.html
7.	Click "Save".

### Create the Approval Static Web Page

_The document at the following link contains the HTML code for the static web page that will be used for allowing manual approval of images to be added to the Rekognition collection: [index.html](./index.html)_

1. Copy the _[index.html](./index.html)_ file to your computer, save it as index.html, and make the following substitution:
```
cognitoIdentityPoolId = 'YOUR-IdentityPoolId'
apiGatewayUrl = 'https://YOUR-APIGatewayEndPoint.execute-api.us-east-1.amazonaws.com/respond/succeed'
```
_Next, we will upload that file to S3 according to the following steps:_

1.	In the AWS Console, click on “Services” in the top, left-hand corner of the screen, and click on “S3” (you can find it by typing _s3_ into the search field at the top of the screen).
2.	In your list of S3 buckets, click on the bucket you created in section 1.2, step 1.(i.e. [Your name or username]-web), and click “Upload”.
3.	Either drag and drop your file into that space, or click “Add files” browse for the file on your computer.
4.	Click “Upload”.
5.	Now, in that bucket in S3, click on the file that you just uploaded, and then click “Make Public”.
6.	Ensure that you can access your website via your browser by clicking on the link that is displayed at the bottom of the screen for that file (see screenshot above for reference).
7.	You should see a web-page like this:

![](images/Approval_page.png)