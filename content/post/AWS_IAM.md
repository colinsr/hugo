---
title: "AWS Identity & Access Management"
date: 2018-03-17T07:49:12-04:00
tags: ["aws", "amazon web services", "IAM"]
categories: ["cloud"]
---


# What is IAM?

* IAM is AWS' way of tracking permissions across all the resources assiciated with a given AWS cloud subscription.
* IAM controls access at the user, group or role level - and also handles federation to give temporary permission to external users.
* Inside IAM you are able to customize the login URL for the specific AWS account.
* You can configure console password policies, enforce MFA across all accounts.
* You're able to manage users, groups and roles as well as their associated permissions.

Here is what you'll be looking at if you are viewing IAM in the AWS Console:
  
![IAM Dashboard](/img/IAMdashboard.png)

## IAM Policies
  JSON definition which define what permissions a user/group/role have.  
  Can be very loosely defined giving access to all actions applied to all resources in the account, or could be extremely fine grained allowing a very specific set of actions on a single AWS resource.
  
  Here are a few examples of AWS IAM Policies:
  ```
  {
    "Version": "2012-10-17",
    "Statement": {
      "Effect": "Allow",
      "Action": "<SERVICE-NAME>:<ACTION-NAME>",    
      "Resource": "*"
    }
  }
  ```

  If you were to throw a `*` into the above policy for the Action property, you would be granting God mode to all AWS services to whatever resource (user/group/role) you apply this policy to.

  Some of the rules around IAM policies can be a little confusing.  For example, if you had a user who was part of a group that gave administrator access to all S3 buckets inside an AWS account, but you wanted to lock down one specific bucket you can apply a second policy with an explicit Deny policy which will override the Allow `*` policy.
  
  ```
  {
    "Version": "2012-10-17",
    "Statement": [
      // All actions allowed on all S3 buckets
      {
        "Effect": "Allow",
        "Action": "s3:*",
        "Resource": "*"
      },
      // Lock down top secret bucket - no actions allowed
      {
        "Effect": "Deny",
        "NotAction": "s3:*",
        "NotResource": [
          "arn:aws:s3:::top-secret-bucket",
          "arn:aws:s3:::top-secret-bucket/*"
        ]
      }
    ]
  }
  ```

  Once you understand the basics Policies aren't too bad to work with - but be aware in the absence of an explicit allow, there is an implicit deny.<br>
  If you were to add a new user he would have no access to any services until he/she gets added to a group or has a policy directly applied to his user account.

## IAM Users
  A "user" is similar to a local login account on an operating system.<br>
  You may have users who are system admins, endusers, or even service accounts that are used by different applications to access your machine.<br>
  IAM users accomplish the same goal, giving things access to resources and controlling access levels.<br>
  Each account has a root user, which can perform `*` Actions on `*` Resources - hence it's a best practice to restrict usage of the root account in order to minimize exposure of the root user's credentials.<br>
  The root account should be used to set up MFA (multi factor authentication) for the root user and create an Admin user which would be used on an ongoing basis in lieu of the root account.<br>
  Users can have either API keys, which would be used for programmatic access to that AWS account, and/or console access using a username/password/MFA combination.<br>
  If you were to add a user in IAM here is what you'd be looking at:
  ![IAM Dashboard](/img/IAMadduser.png)
  On the next page you can grant permission by assigning policies directly to the user account or add the user to a group - if you skip this step you'll get a warning from the kind folks at AWS:
    
  >`This user has no permissions`<br>
  >`You haven't given this user any permissions.`<br>
  >`This means that the user has no access to any AWS service or resource.`<br>
  >`Consider returning to the previous step and adding some type of permissions.`
  
  Keep in mind - you must either download the csv or store the API credentials, Access Key ID and Secret Access Key, upon creating the user or you will have to go through the process of generating a new set of credentials at a later point.

## IAM Groups
  Groups can be used to apply a set of permissions across a group of users. Go figure.<br>
  How this works is that you go in and either create a specific policy or policies and/or use one or a combination of canned AWS policies.<br>
  Once you've defined the permissions you'd like to grant access to a given group, you simply assign a user to that group and viola - you're ready to rock.<br>
  If we wanted to add our user `SilkyJohnson` to the `administrators` group from the console, we'd either navigate to the Groups section and select the appropriate group or go to the User directly and select `Add users to group`.

## IAM Roles
  Roles are used to grant permissions to AWS resources like EC2 (Elastic Compute Cloud), Lambda function, etc, or even external users after being authenticated, via Active Directory or Single Sign on, needing temporary permissions inside your AWS footprint.<br>
  Roles should always be used instead of actual credentials on an EC2 instance.
  In the past roles *had* to be assigned at EC2 instance launch time, but now you are able to add or change the role assigned to a given EC2 instance via AWS CLI or even in the AWS Console.
  An IAM User from a different AWS account can even assume a role and get access to your AWS services.<br>
  IAM Policies are attached to a role, and they work in a similar fashion to groups.<br>
  i.e. Grant a set of permissions to a specific set of resources, assume the role, and now you're in business.
  AWS has some canned role permissions - Service Roles, or you can roll your own. 

  Depending on what AWS services you are using, you'll need to get pretty comfortable with these roles and how they work.<br>
  For example, when you set up a lambda function that needs access to S3, you'll need to make sure to grant the permissions, via a role, for that interaction to take place.

## IAM STS (Security Token Service)
  STS provides short term temporary credentials used to grant access to AWS services in your account.<br>
  It can be used to grant non-AWS account holders or AWS accounts from a separate account permissions within your account.
  Credentials are returned from an STS API call and include the following:
  
```
  {
    "sessionId":"temporary access key ID",
    "sessionKey":"temporary secret access key",
    "sessionToken":"security token"
  }
```

  STS takes a lot of the headache away from having to manage temporary credentials.<br>
  Use cases of STS:
  
  * Identity Federation, using SAML or web identity federation
  * Roles for Cross-Account access
  * Roles for EC2 and other AWS services

  STS API Actions:
    
  * AssumeRole:<br>
    Returns a set of temporary security credentials (consisting of an access key ID, a secret access key, and a security token) that you can use to access AWS resources that you might not normally have access to. 
  
  + AssumeRoleWithSAML:<br>
    Returns a set of temporary security credentials for users who have been authenticated via a SAML authentication response. 
  + AssumeRoleWithWebIdentity:<br>
    Returns a set of temporary security credentials for users who have been authenticated in a mobile or web application with a web identity provider, such as Amazon Cognito, Login with Amazon, Facebook, Google, or any OpenID Connect-compatible identity provider.
  + DecodeAuthorizationMessage:<br>
    Decodes additional information about the authorization status of a request from an encoded message returned in response to an AWS request.
  + GetCallerIdentity:<br>
    Returns details about the IAM identity whose credentials are used to call the API.
  + GetFederationToken:<br>
    Returns a set of temporary security credentials (consisting of an access key ID, a secret access key, and a security token) for a federated user.
  + GetSessionToken:<br>
    Returns a set of temporary credentials for an AWS account or IAM user. 
    The credentials consist of an access key ID, a secret access key, and a security token.
  
  The process is a bit complicated and we can do a deep dive on this later on to see how all the pieces fit together.

## IAM API Keys
  These are used for programmatic access to AWS resources.<br>
  If you're planning to interact with the console only, you can skip over this, but if you plan to use the AWS CLI, powershell tools, any of the supported AWS SDKs or direct interaction using HTTP calls using the APIs for the different services you'll need to make sure to set these up.<br>
  Remember that the keys are only available one time, upon creation.<br>
  Should you misplace your keys, you'll have to go through the regeneration process.<br>
  Do not combine Roles and API keys, use Roles whenever possible!

There you have a quick summary of AWS IAM, feel free to post any questions, comment, concerns in the comments section below.<br>
:thumbsup: