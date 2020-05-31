# Azure DevOps Integration with AWS CodePipeline, SonarQube and JFrog

This documentation provides the steps to setup AWS CodePipeline, Azure DevOps, SonarQube and JFrog integrations.

## Azure DevOps Repository
## Sonarqube Project
## Configuration
## Azure DevOps WebHooks with AWS Services
## Configure Azure DevOps Repo WebHook Trigger

## Azure DevOps Setup
1. Log into the Azure Devops account
2. Create an organization
3. Create a repo by importing existing repo from GitHub
4. Create a Personal Access Token for AWS CodePipeline to download the repo codebase as zip file


## Sonarqube Setup

### Account Setup
1. Go to https://sonarcloud.io
2. Login using any of the GitHub, BitBucket etc credentials

### Sonarqube Project Setup
1. Click the `+` icon in top right corner
2. Select `Analyse New Project` option
3. Select the repo from the list and Click `Setup` button
4. Under `Configure` tab, select the `With Other CI Tools` option from the Choose another analysis method options.
5. Select appropriate codebase `language` under the 'What option best describes your build?'
6. Select the `Operation System` name from the list under the 'What is your OS?'
7. Click icon next to `+` and select organization name under `My Organizations`
8. Click `Administration` and select `Organization settings`
9. Right corner, copy the value of the label `Key:` and store it somewhere.

### SonarQube Token Setup
1. Click on the `Profile` icon and select the `account name`
2. Click on the `Security` tab
3. Enter a friend name for the repo access token in the `Generate Token` field and click `Generate`.
4. Copy the token string by clicking the `copy` button and save it somwhere. You cannot retrieve this again.

## Configuration
1. Log into AWS Management Console and select `Secrets Manager`
2. Click on the `Store a new secret` button
3. Select `Other types of secrets` in the Select secret type
4. Under the secret key/value, add the following key/value pairs
   a. key: token, value: <paste the sonarqube token value from the section `SonarQube Token Setup` step 4.
   b. key: host, value: https://sonarcloud.io
   c. key: organization, value: <Sonarqube Organization Key> (obtained from the section Sonarqube Project Setup step 9)
5. Click `Next` button
6. Enter the secret name value `dev/sonarcloud` and click `Next` button
7. Click `Next` button
8. Click `Store` button to complete the setup

## Azure DevOps WebHooks with AWS Services
1. Log into AWS Management Console
2. Click on this [link](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/create/template?stackName=Azure-DevOps-to-Amazon-S3&templateURL=https://aws-quickstart.s3.amazonaws.com/quickstart-git2s3/templates/git2s3.template)
3. Click `Next` button
4. Enter the Output S3 Bucket Name as `azure-repo-codebase'
6. In the Allowed IPs, enter the value 13.89.236.72,52.165.41.252,52.173.25.16,13.86.38.60,20.45.1.175,13.86.36.181,52.158.209.56
7. In the Git Personal Access Token, paste the AzureDevOps Personal Access Token value created in the section `Azure DevOps Setup` step 5.
8. In the Quick Start S3 Bucket Name, enter the value `Azure-DevOps-WebHooks`
9. In the Quick Start S3 Key Prefix, enter the value `Assets/`
10. Click `Next`
11. Click `Next`
12. In the Review screen, under Capabilities section select the checkbox for `I acknowledge that AWS CloudFormation might create IAM resources.`
13. Click `Create stack` to complete the setup
14. After the stack creation is completed, go to the `Outputs` tab and copy the value of key name `ZipDownloadWebHookApi`
15. Go to the section `Configure Azure DevOps Repo WebHook Trigger` and follow the steps 1-12.

## Configure Azure DevOps Repo WebHook Trigger

1. Log into the Azure DevOps portal and select the Repo
2. Click the `Project Settings` bottom of the left navigatiojn
3. Select `service hooks`
4. Click `+` to add a new webhook
5. Select `Web Hooks` from the list of Service and click `Next`
6. Select `Code Pushed` from the list under `Trigger on this type of event`
7. Under the `Repository`, select the repo name from the list
8. Select `Master` from the list under the branch
9. Leave the default value `[Any]` for the Pushed by member of group and click `Next` button
10. In the `Action` screen, pase the value obtained from the section Azure DevOps WebHooks with AWS Services step 14.
11. Click `Test` button for test
12. Click `Finish` button to complete the setup

## AWS Codepipeline

### CodePipeline Setup
1. Log into the AWS Account and select `CodePipeline` service
2. Click `Create Pipeline` button
3. Under the `Pipeline Settings`, enter the pipeline name.
4. Expand the `Advanced settings`
5. Make sure the `Default Location` option is selected under `Artifact Store` and `Default AWS Managed Key` option under the `Encryption key` section
6. Under the `Source`, select Amazon S3 and select the bucket name (step TBD)
7. Under the `Build`, select `AWS CodeBuild` option.
8. Click `create project` button (follow the steps under the section `CodeBuild Project Setup for Unit Test`)
9. Click the `Next` button
10. Click `skip deploy stage` button to skip the deployment step
11. Review the pipeline details and click `Create Pipeline` button to complete the initial pipeline step
12. Select the pipeline name link from the list
13. In the pipeline screen, select `Edit` to add additional stages
14. Below Unit Test stage, click the `+ Add Stage` button to add `Quality-Gate` stage
15. In the Add Stage, enter the stage name `Quality-Gate`.
16. Click `+ Add action group` button to add the steps
17. Enter `code-quality` in the `Action Name`
18. Select `AWS CodeBuild` in the `Action provider`
19. Select the region where the pipline s3 bucket is located
20. Select `OutputArtifact` fromt the list under `Input Artifacts`
21. Click `Create project` to create `CodeBuild Project Setup for Quality Gate`

### CodeBuild Project Setup for Unit Test
1. Enter the build project name (preferrably, prefix `-unit-test` with the pipeline name)
2. Under `Environment` section, select `Managed Image`
3. Select `Ubuntu` for the operation system
4. Select `Standard` for the runtime
5. Select `aws/codebuild/standard:3.0` for the image. (Refer the [link](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html) for which OS and image should be selected based on the lanaguge version.)
6. Under buildsepc, select `insert build commands` option and click `switch to editor` link
7. In the `build commands` text editor, update with the following code


```YAML
version: 0.2

env:
  variables:
      Project: "<Sonarqube project name goes here>"
  secrets-manager:
      LOGIN: dev/sonarcloud:token
      HOST: dev/sonarcloud:host
      Organization: dev/sonarcloud:organization
phases:
  install:
    #If you use the Ubuntu standard image 2.0 or later, you must specify runtime-versions.
    #If you specify runtime-versions and use an image other than Ubuntu standard image 2.0, the build fails.
    runtime-versions:
       java: openjdk8
  pre_build:
    commands:
       - apt-get update
       - wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.3.0.2102-linux.zip
       - unzip ./sonar-scanner-cli-4.3.0.2102-linux.zip
       - export PATH=$PATH:/sonar-scanner-cli-4.3.0.2102-linux/bin/
  build:
    commands:
       - mvn clean install
       - mvn sonar:sonar -Dsonar.login=$LOGIN -Dsonar.host.url=$HOST -Dsonar.projectKey=$Project -Dsonar.organization=$Organization -Dsonar.jacoco.reportPath=target/coverage-reports/jacoco-unit.exec
       
artifacts:
  files:
    - '**/*'
  base-directory: 'target'
```
8. Click `continue to pipeline` button. It will take you back to the section `CodePipeline Setup` step 21.

### CodeBuild Project Setup for Quality Gate
1. Enter the build project name (pipeline name prefix with -quality-gate)
2. Under `Environment` section, select `Managed Image`
3. Select `Ubuntu` for the operation system
4. Select `Standard` for the runtime
5. Select `aws/codebuild/standard:4.0` for the image. (Refer the [link](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html) for which OS and image should be selected based on the lanaguge version.)
6. Under buildsepc, select `insert build commands` option and click `switch to editor` link
7. In the `build commands` text editor, update with the following code and update the Project variable with the sonarqube project name

```
version: 0.2

env:
  variables:
      Project: "<Sonarqube project name goes here>"
phases:
  install:
    #If you use the Ubuntu standard image 2.0 or later, you must specify runtime-versions.
    #If you specify runtime-versions and use an image other than Ubuntu standard image 2.0, the build fails.
    runtime-versions:
       java: corretto8
  build:
    commands:
       - curl https://sonarcloud.io/api/qualitygates/project_status?projectKey=$Project >result.json
       - if [ $(jq -r '.projectStatus.status' result.json) = ERROR ] ; then $CODEBUILD_BUILD_SUCCEEDING -eq 0 ;fi
```
8. Click `continue to pipeline` button. It will take you back to the section `CodePipeline Setup` step 8.

## JFrog Setup

### Install JFrog Instance

1. Log into the AWS Account and select the Region where the AWS CodePipeline project will be created.
2. Goto AWS Marketplace and click Discover products
3. Search for 'JFrog Open Source' and select it from the list
4. Click `continue to subscribe` button
5. Click `continue to configuration` button
6. Leave the default selection under `Delivery Method` and `Software Version`
7. Select the desired `region` from the list and click `Contineu to Launch` button.


