# Features

This plugins adds Jenkins pipeline steps to interact with the AWS API.

* Invalidating CloudFront distributions
* Creating, updating and deleting CloudFormation stacks
* Up- and downloading files to/from S3

## Upcoming features

Planned features for upcoming versions include:

* Login to ECR
* S3 sync

# Usage / Steps

## withAWS

the `withAWS` step provides authorization for the nested steps. 
You can provide region and profile information or let Jenkins 
assume a role in another or the same AWS account. 
You can mix all parameters in one `withAWS` block.

Set region information:

```
withAWS(region:'eu-west-1') {
    // do something
}
```

Use Jenkins UsernamePassword credentials information (Username: AccessKeyId, Password: SecretAccessKey):

```
withAWS(credentials:'nameOfSystemCredentials') {
    // do something
}
```

Use profile information from `~/.aws/config`:

```
withAWS(profile:'myProfile') {
    // do something
}
```

Assume role information (account is optional and uses current account as default):

```
withAWS(role:'admin', roleAccount:'123456789012') {
    // do something
}
```

## awsIdentity

Print current AWS identity information to the log.

```
awsIdentity()
```

## cfInvalidate

Invalidate given paths in CloudFront distribution.

```
cfInvalidate(distribution:'someDistributionId', paths:['/*'])
```

## s3Upload

Upload a file/folder from the workspace to an S3 bucket.
If the `file` parameter denotes a directory, the complete directory including all subfolders will be uploaded.

```
s3Upload(file:'file.txt', bucket:'my-bucket', path:'/path/to/target/file.txt')
s3Upload(file:'someFolder', bucket:'my-bucket', path:'/path/to/targetFolder/')
```

## s3Download

Download a file/folder from S3 to the local workspace. 
Set optional parameter `force` to `true` to overwrite existing file in workspace.
If the `path` end with a `/` the complete virtual directory will be downloaded.

```
s3Download(file:'file.txt', bucket:'my-bucket', path:'/path/to/source/file.txt', force:true)
s3Download(file:'targetFolder/', bucket:'my-bucket', path:'/path/to/sourceFolder/', force:true)
```

## cfnValidate

Validates the given CloudFormation template.

```
cfnValidate(file:'template.yaml')
```

## cfnUpdate

Create or update the given CloudFormation stack using the given template from the workspace.
You can specify an optional list of parameters. 
You can also specify a list of `keepParams` of parameters which will use the previous value on stack updates.

If you have many parameters you can specify a `paramsFile` containing the parameters. The format is either a standard 
JSON file like with the cli or a YAML file for the [cfn-params](https://www.npmjs.com/package/cfn-params) command line utility. 

Additionally you can specify a list of tags that are set on the stack and all resources created by CloudFormation.
The step returns the outputs of the stack as a map.

```
def outputs = cfnUpdate(stack:'my-stack', file:'template.yaml', params:['InstanceType=t2.nano'], keepParams:['Version'], tags:['TagName=Value'])
```

## cfnDelete

Remove the given stack from CloudFormation.

```
cfnDelete(stack:'my-stack')
```

## cfnDescribe

The step returns the outputs of the stack as map.

```
def outputs = cfnDescribe(stack:'my-stack')
```

## cfnExports

The step returns the global CloudFormation exports as map.

```
def globalExports = cfnExports()
```

## snsPublish

Publishes a message to SNS.

```
snsPublish(topicArn:'arn:aws:sns:us-east-1:123456789012:MyNewTopic', subject:'my subject', message:'this is your message')
```

# Versions

## 1.7 (master)
* fix environment for withAWS step
* add support for recursive S3 upload/download

## 1.6
* fix #JENKINS-42415 causing S3 errors on slaves
* add paramsFile support for cfnUpdate
* allow the use of Jenkins credentials for AWS access #JENKINS-41261

## 1.5
* add cfnExports step
* add cfnValidate step
* change how s3Upload works to use the aws client to guess the correct content type for the file.

## 1.4
* add empty checks for mandatory strings
* use latest AWS SDK
* add support for CloudFormation stack tags 

## 1.3
* add support for publishing messages to SNS
* fail step on errors during CloudFormation actions 

## 1.2
* add proxy support using standard environment variables
* add cfnDescribe step to fetch stack outputs

## 1.1
* fixing invalidation of CloudFront distributions
* add output of stack creation, updates and deletes
* Only fetch AWS environment once 
* make long-running steps async

## 1.0
* first release containing multiple pipeline steps
