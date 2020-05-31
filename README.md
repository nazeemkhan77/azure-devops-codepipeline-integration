# Azure DevOps Integration with AWS CodePipeline, SonarQube and JFrog

This documentation provides the steps to setup AWS CodePipeline, Azure DevOps, SonarQube and JFrog integrations.

## Azure DevOps Setup
1. Log into the Azure Devops account
2. Create an organization
3. Create a repo by importing existing repo from GitHub
4. Create a Personal Access Token for AWS CodePipeline to download the repo codebase as zip file
5. Setup WebHooks for the repo

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

### SonarQube Token Setup
1. Click on the `Profile` icon and select the `account name`
2. Click on the `Security` tab
3. Enter a friend name for the repo access token in the `Generate Token` field and click `Generate`.
4. Copy the token string by clicking the `copy` button and save it somwhere. You cannot retrieve this again.

## AWS Codepipeline

### CodePipeline Setup
1. Log into the AWS Account and select `CodePipeline` service
2. Click `Create Pipeline` button
3. Under the `Pipeline Settings`, enter the pipeline name.
4. Expand the `Advanced settings`
5. Make sure the `Default Location` option is selected under `Artifact Store` and `Default AWS Managed Key` option under the `Encryption key` section
6. 

## JFrog Setup

### Install JFrog Instance

1. Log into the AWS Account and select the Region where the AWS CodePipeline project will be created.
2. Goto AWS Marketplace and click Discover products
3. Search for 'JFrog Open Source' and select it from the list
4. Click `continue to subscribe` button
5. Click `continue to configuration` button
6. Leave the default selection under `Delivery Method` and `Software Version`
7. Select the desired `region` from the list and click `Contineu to Launch` button.


