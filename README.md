# The lambda infrastructure
## infrastructure.json
The infrastructure.json template creates the required infrastructure bits and their support resources
1. The S3 bucket: updates to files named .zip in this bucket invoke the handler function. Posted as a cloudformation output.
*. A handler function: accepts the event from the s3 bucket and then passes a JSON file to the updater function
*. The updater function: accepts a JSON file from the handler function then udpates and publishes the function
## Templates
The various templates lay out a CICD stack and a lambda function
1. Added a simple but valid .zip file with a simple valid index.py
*. s3 copy the zip file that goes to the lambda-infrastructure bucket with a prefix of the function name
*. now create the function. It creates a git repository, in codecommit, for your lambda function. Posted as a cloudformation output
*. Add a buildspec.yaml simliar to below.
*. create a lambda directory and include index.py and other data in the lambda directory
*. Push the repository to master

## Example Files
### basic index.py
```
def lambda_handler(event, context):
    print event
    return "Repository Test"
```
### buildspec.yaml
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
## Required improvements
### CICD Pipeline
The Pipeline needs to be a pipeline with testing and multiple Environments. There's several ways to accomplish this.
