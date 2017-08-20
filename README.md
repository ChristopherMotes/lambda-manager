# Overview
Here I try to unify and simplify lambda and it's deployment. Building lambdas is tedious expeically if you want everything in an s3bucket/DML. This is a basic framework, that requires some re-engineering from shop to shop.  The infrastructure is an S3 bucket with Lambda notification that alerts a lambda handler. The handler strips the basename from the s3 key that is the lambda zip file. The handler passes that to a lambda updater that updates the function mentioned in that basename. Initially, this was much more complicated. Thus, it made a ton of sense to have a handler and updater function. In a more complex environment, separating these functions will be more self-evident. There's a chiken egg problem here. The functions exist as code in the infrastructure.json cloudformation template. We need this to build the initial infrastructure. It wouldn't be glarling complicated to have these as S3 templates. Since they're simple, they can stay as code.

The templates build the infrastructure for each lambda. The cloudformation grabs the the bucket names (for lambdas and for the pipeline) from the infrastructure stack as exported resources. Then it build the Pipeline, codebuild, and that lambda function itself. Cloudformation won't let you build a lambda without a valid zip file in place. Thus, you have to manually put it there first. The template also creates the the CodeCommit/Git repository and exports that resources name and HTTPS URL. From there it's staight code pipleline. 
## Required improvements
### CICD Pipeline
1. The Pipeline needs to be a pipeline with testing and multiple Environments. There's several ways to accomplish this.
2. Enventually the s3 cp command should use another step, perhaps a lambda, that that grabs the artifact from the ArtifactBucket and deploys that. Or the notification should just come from the Artifact bucket itself. 
# Cloduformation Stuff
## infrastructure.json
The infrastructure.json template creates the required infrastructure bits and their support resources
1. The S3 bucket: updates to files named .zip in this bucket invoke the handler function. Posted as a cloudformation output.
2. A handler function: accepts the event from the s3 bucket and then passes a JSON file to the updater function
3. The updater function: accepts a JSON file from the handler function then udpates and publishes the function
## Templates
The various templates lay out a CICD stack and a lambda function
1. Added a simple but valid .zip file with a simple valid index.py
2. s3 copy the zip file that goes to the lambda-infrastructure bucket with a prefix of the function name
3. now create the function. It creates a git repository, in codecommit, for your lambda function. Posted as a cloudformation output
4. Add a buildspec.yaml simliar to below.
5. create a lambda directory and include index.py and other data in the lambda directory
6. Push the repository to master

# Example Files
## basic index.py
```
def lambda_handler(event, context):
    print event
    return "Repository Test"
```
## buildspec.yaml
```
version: 0.1

environment_variables:
  plaintext:
            
phases:
  build:
    commands:
      - export Environment
      - echo "creating lambda.zip"
      - ls -l
      - cd lambda && zip ../lambda.zip *
  post_build:
    commands:
      - ls -l 
      - aws s3 cp ./lambda.zip "s3://$LambdaInfrastructureBucket/$FunctionName/"
artifacts:
  files:
    - lambda.zip
```
