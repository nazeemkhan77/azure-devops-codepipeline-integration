# Azure DevOps Integration with AWS CodePipeline, SonarQube and JFrog

This documentation provides the steps to setup AWS CodePipeline, Azure DevOps, SonarQube and JFrog integrations.

Pre-Requisites
- Azure DevOps Repository
- Sonarqube Project
- Sonarqube Configuration
- Azure DevOps WebHooks with AWS Services
- Configure Azure DevOps Repo WebHook Trigger

## Azure DevOps Setup
1. Log into the Azure Devops Portal https://dev.azure.com/
2. Click `New Organization` link to create an organization
3. Enter the oranization name and select `Central US` from the list under `We'll host your projects in`, and click `Next`
4. In the `Create a project to get started', enter the project name and click `+ Create Project`button. 
5. Import the existing [GitHub project](https://github.com/in28minutes/spring-unit-testing-with-junit-and-mockito) into the new repo following the steps in the [link](https://docs.microsoft.com/en-us/azure/devops/repos/git/import-git-repository?view=azure-devops)
6. Click the `Files` under the repo and select the `pom.xml` file.
7. Click `Edit` button and add the following xml snippet to add code coverage plugin configuration after line 24 (under <properties> section)

```XML
   <jacoco.version>0.8.3</jacoco.version>
   <sonar.java.coveragePlugin>jacoco</sonar.java.coveragePlugin>
   <sonar.dynamicAnalysis>reuseReports</sonar.dynamicAnalysis>
   <sonar.jacoco.reportPath>${project.basedir}/../target/jacoco.exec</sonar.jacoco.reportPath>  
   <sonar.language>java</sonar.language>
```
8. Similarly, add the following xml configuration after the line 65 (under <plugins> section) and click `Commit` button to save the changes

```XML
   <plugin>
       <groupId>org.jacoco</groupId>
       <artifactId>jacoco-maven-plugin</artifactId>
       <version>${jacoco.version}</version>
       <configuration>
      <skip>${maven.test.skip}</skip>
      <destFile>${basedir}/target/coverage-reports/jacoco-unit.exec</destFile>
      <dataFile>${basedir}/target/coverage-reports/jacoco-unit.exec</dataFile>
      <output>file</output>
      <append>true</append>
      <excludes>
          <exclude>*MethodAccess</exclude>
      </excludes>
       </configuration>
       <executions>
      <execution>
          <id>jacoco-initialize</id>
          <goals>
         <goal>prepare-agent</goal>
          </goals>
          <phase>test-compile</phase>
      </execution>
      <execution>
          <id>jacoco-site</id>
          <phase>verify</phase>
          <goals>
         <goal>report</goal>
          </goals>
      </execution>
       </executions>
   </plugin>
```
9. Update the following xml configuration
```XML
<artifactId>unit-testing</artifactId>
```
with
```XML
<artifactId>spring-unit-testing-with-junit-and-mockito</artifactId>
```
10. Similarly, update the below xml configuration
```XML
<name>unit-testing</name>
```
with
```XML
<name>spring-unit-testing-with-junit-and-mockito</name>
```

11. Click on the `User Settings` icon and select `Personal Access Token` to create the token for AWS CodePipeline to download the repo codebase as zip file
12. Click `+ New Token` button and provide a user-friendly name (for ex, aws-codepipeline-access-token)
13. Under the Scopes, select `Read` under Code section to provide read access to the token consumer.
14. Click `Create` button to complete the setup.


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

## Sonarqube Configuration
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
6. In the Allowed IPs, enter the `Azure DevOps Services IPs for the Regional Identity Service - Central United States` value 13.89.236.72,52.165.41.252,52.173.25.16,13.86.38.60,20.45.1.175,13.86.36.181,52.158.209.56 (Refer the [link](https://docs.microsoft.com/en-us/azure/devops/migrate/migration-import?view=azure-devops#azure-devops-services-ips) for different region) 
7. In the Git Personal Access Token, paste the AzureDevOps Personal Access Token value created in the section `Azure DevOps Setup` step 11.
8. In the Quick Start S3 Bucket Name, enter the value `Azure-DevOps-WebHooks`
9. In the Quick Start S3 Key Prefix, enter the value `Assets/`
10. Click `Next`
11. Click `Next`
12. In the Review screen, under Capabilities section select the checkbox for `I acknowledge that AWS CloudFormation might create IAM resources.`
13. Click `Create stack` to complete the setup
14. After the stack creation is completed, go to the `Outputs` tab and copy the value of key name `ZipDownloadWebHookApi`
15. Go to the section `Configure Azure DevOps Repo WebHook Trigger` and follow the steps 1-12.
16. Go to `Lambda` service and select the `AzureRepo-to-Amazon-S3-ZipDlLambda` lambda function to edit.
17. In the code editor, replace the following existing code

```JavaScript
    if 'X-Hub-Signature' in event['params']['header'].keys():
        hostflavour = 'githubent'
    elif 'X-Gitlab-Event' in event['params']['header'].keys():
        hostflavour = 'gitlab'
    elif 'User-Agent' in event['params']['header'].keys():
        if event['params']['header']['User-Agent'].startswith('Bitbucket-Webhooks'):
            hostflavour = 'bitbucket'
        elif event['params']['header']['User-Agent'].startswith('GitHub-Hookshot'):
            hostflavour = 'github'
        elif 'Bitbucket-' in event['params']['header']['User-Agent']:
            hostflavour = 'bitbucket-server'
    elif event['body-json']['publisherId'] == 'tfs':
        hostflavour='tfs'
```
with this new code snippet

```JavaScript
    if event['body-json']['publisherId'] == 'tfs':
        hostflavour='tfs'
    elif 'X-Hub-Signature' in event['params']['header'].keys():
        hostflavour = 'githubent'
    elif 'X-Gitlab-Event' in event['params']['header'].keys():
        hostflavour = 'gitlab'
    elif 'User-Agent' in event['params']['header'].keys():
        if event['params']['header']['User-Agent'].startswith('Bitbucket-Webhooks'):
            hostflavour = 'bitbucket'
        elif event['params']['header']['User-Agent'].startswith('GitHub-Hookshot'):
            hostflavour = 'github'
        elif 'Bitbucket-' in event['params']['header']['User-Agent']:
            hostflavour = 'bitbucket-server'
```
18. Similarly, replace the following line

```JavaScript
        archive_url = event['body-json']['resourceContainers']['account']['baseUrl'] + 'DefaultCollection/' + event['body-json']['resourceContainers']['project']['id'] + '/_apis/git/repositories/' + event['body-json']['resource']['repository']['id'] + '/items'
```

with this code

```JavaScript
        archive_url = event['body-json']['resourceContainers']['account']['baseUrl'] + event['body-json']['resourceContainers']['project']['id'] + '/_apis/git/repositories/' + event['body-json']['resource']['repository']['id'] + '/items'
```
19. Click the `Save` button to complete the code change

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

### Add Source Stage
6. Under the `Source`, select Amazon S3 and enter the bucket name `azure-repo-codebase` (specified in the step 4 under the section `Azure DevOps WebHooks with AWS Services`) and paste the S3 object key as `<Azure Repo Organization Name>/<repo name>/master/<repo name>.zip`

### Add Unit Test stage
7. Under the `Build`, select `AWS CodeBuild` option.
8. Click `create project` button (follow the steps under the section `CodeBuild Project Setup for Unit Test`)
9. Click the `Next` button
10. Click `skip deploy stage` button to skip the deployment step
11. Review the pipeline details and click `Create Pipeline` button to complete the initial pipeline step
12. Goto IAM service and search for the service role associated with Unit Test build project.
13. Click + Add inline policy to add inline policy to give access to read the secret manager key
14. Click JSON tab and paste the following json snippet:

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "secretsmanager:GetSecretValue",
            "Resource": "<secret ARN>"
        }
    ]
}
```
16. To verify the Unit Test setup, goto azure repo, edit and update readme.md file and click commit.
17. Check the pipeline must be triggered per above code change.


### Add Quality Gate stage
18. Select the pipeline name link from the list
19. In the pipeline screen, select `Edit` to add additional stages
20. Below Unit Test stage, click the `+ Add Stage` button to add `Quality-Gate` stage
21. In the Add Stage, enter the stage name `Quality-Gate`.
22. Click `+ Add action group` button to add the steps
23. Enter `code-quality` in the `Action Name`
24. Select `AWS CodeBuild` in the `Action provider`
25. Select the region where the pipline s3 bucket is located
26. Select `OutputArtifact` fromt the list under `Input Artifacts`
27. Click `Create project` to create `CodeBuild Project Setup for Quality Gate`

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

### Trigger CodePipeline
1. Log into the Azure DevOps portal and select the Repo
2. Select any file (for ex, readme.md) and click `Edit` button
3. Make some changes (doesn't matter what change it is)
4. Click `Save` button
5. Click `Commit` button in the commit dialog screen to trigger the pipeline
5. Goto AWS CodePipeline and select the pipeline name
6. Check for the build is triggered

## JFrog Setup

### Install JFrog Instance

1. Log into the AWS Account and select the Region where the AWS CodePipeline project will be created.
2. Goto AWS Marketplace and click Discover products
3. Search for 'JFrog Open Source' and select it from the list
4. Click `continue to subscribe` button
5. Click `continue to configuration` button
6. Leave the default selection under `Delivery Method` and `Software Version`
7. Select the desired `region` from the list and click `Contineu to Launch` button.


