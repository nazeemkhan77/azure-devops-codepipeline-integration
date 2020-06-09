# Azure DevOps Integration with AWS CodePipeline, SonarQube and JFrog

This documentation provides the steps to setup AWS CodePipeline, Azure DevOps, SonarQube and JFrog integrations.

Pre-Requisites
- Azure DevOps Repository
- Sonarqube Project
- Sonarqube Configuration
- Azure DevOps WebHooks with AWS Services
- Configure Azure DevOps Repo WebHook Trigger

## Azure DevOps Setup

## Create Azure Repo (Importing existing GitHub project)
1. Log into the Azure Devops Portal https://dev.azure.com/
2. Click `New Organization` link to create an organization
3. Enter the oranization name and select `Central US` from the list under `We'll host your projects in`, and click `Next`
4. In the `Create a project to get started', enter the project name and click `+ Create Project`button. 
5. Import the existing [GitHub project](https://github.com/nazeemkhan77/spring-unit-testing-with-junit-and-mockito) into the new repo following the steps in the [link](https://docs.microsoft.com/en-us/azure/devops/repos/git/import-git-repository?view=azure-devops)
6. Click the `Files` under the repo and you should see project files listed


## Generate Personal Access Token
7. Click on the `User Settings` icon and select `Personal Access Token` to create the token for AWS CodePipeline to download the repo codebase as zip file
8. Click `+ New Token` button and provide a user-friendly name (for ex, aws-codepipeline-access-token)
9. Under the Scopes, select `Read` under Code section to provide read access to the token consumer.
10. Click `Create` button to complete the setup.

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
7. In the Git Personal Access Token, paste the AzureDevOps Personal Access Token value created in the sub-section `Generate Personal Access Token` step 7.
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

### Install JFrog Artifactory

In this section, follow the steps to configure JFrog instance using the Open Source JFrog Artifactory from the AWS Marketplace and start the artifactory instance.

1. Log into the AWS Account and select the Region where the AWS CodePipeline project will be created.
2. Goto AWS Marketplace and click Discover products
3. Search for 'JFrog Open Source' and select it from the list
4. Click `continue to subscribe` button
5. Click `continue to configuration` button
6. Leave the default selection under `Delivery Method` and `Software Version`
7. Select the desired `region` from the list and click `Continue to Launch` button. Wait for the instance to status change to READY state.
8. Select the ec2 instance and copy the private ip address and host name.
9. click Connect button and follow the chmod and ssh commands (use ubuntu instead of root) to remote log into the ec2 instance.
9. Type the command `sudo vi /etc/hosts` to open the hosts file
10. After the below first line, type the private ip and host name separated by tab space to the hosts file, save and exit from the vi.
11. Start the artifactory by entering the following commands
```
sudo su
cd /home/ubuntu/artifactory-oss-6.8.2/bin
./artifactory.sh
```
12. After the `Artifactory successfully started` message, open the browser and type the URL http://ec2-instance-public-hostname:8081/artifactory. (Replace the ec-instance-public-hostname with the actual value)
13. Login with the admin credentials, admin/password.
14. Open the ec2 instance security group and check the inbound rules for ports 22 and 8081 must be opened. Port 22 must be allowed from specific ip and 8081 must be opened for all (0.0.0.0).

### Setup Maven Repository

After the JFrog Artifactory is setup and running, a Maven repostiory needs to setup to resolve and deploy artifacts and plugins. 

1. Login with the admin credentials, admin/password.
2. Click the link `Welcome, admin` and select `Quick Setup` to create maven repository
3. Select `Maven` from the repositories and click `create` button.
4. Under `Set Me Up` section, you should see the following repository keys for snapshot and release
   * lib-snapshot
   * lib-snapshot-local
   * lib-release
   * lib-release-local
5. Now the Maven repository setup is completed.

### Configure Maven Project to publish artifacts to JFrog Artifactory

Maven project configurations are stored in pom.xml file. In this section, we will configure the maven-artifactory sections to integrate with JFrog Artifactory.

First, we will configure the plugin and dependency for the artifactory-maven
1. Add the following xml configuration under the <dependencies> section for the `artifactory-maven-plugin`
   ```XML
	<dependency>
		<groupId>org.jfrog.buildinfo</groupId>
		<artifactId>artifactory-maven-plugin</artifactId>
		<version>2.7.0</version>
		<type>pom</type>
	</dependency>
   ```
2. Similarly, add the configuration for the `artifactory-maven-plugin` with artifactory connection details
   ```XML
   <plugin>
       <groupId>org.jfrog.buildinfo</groupId>
       <artifactId>artifactory-maven-plugin</artifactId>
       <version>2.7.0</version>
       <inherited>false</inherited>
       <executions>
           <execution>
               <id>snapshots</id>
               <goals>
                   <goal>publish</goal>
               </goals>
               <configuration>
                   <publisher>
                       <contextUrl>${artifactory.url}</contextUrl>
                       <username>${artifactory.username}</username>
                       <password>${artifactory.password}</password>
                       <repoKey>libs-release-local</repoKey>
                       <snapshotRepoKey>libs-snapshot-local</snapshotRepoKey>							
                   </publisher>
               </configuration>
           </execution>
       </executions>
   </plugin>
   ```
3. Add the following XML configuration to configure the snapshot build artifact publishing artifactory details

```XML
<distributionManagement>
	<snapshotRepository>
		<uniqueVersion>false</uniqueVersion>
		<id>snapshots</id>
		<name>Spring Unit Testing With Junit and Mockito</name>
		<url>${artifactory.url}/artifactory/libs-snapshot-local</url>
	</snapshotRepository>
</distributionManagement>
```

## Add Publish Artifacts stage
In this section, new stage for building and publishing artifacts to JFrog Artifactory will be added in the existing pipeline.
1. Log into the AWS Management Console and select Code Pipeline service,
2. Select the pipeline name link from the pipelines list and click `Edit` button.
3. Goto the Code-Quality step and click the below button `+ Add Stage` to add new build step. 
4. Enter the action name `Publish-Artifacts' and click `Add Stage` button
5. Click `+ Add Action Group` button.
6. Enter the Action name `publish-artifacts`, select `AWS CodeBuild` under Action Provider, select `Source Artifact` under Input Artifacts and click `Create Project` button to create `CodeBuild Project Setup for Publish Artifacts`

### CodeBuild Project Setup for Publish Artifacts
1. Enter the build project name (pipeline name prefix with -publish-artifacts)
2. Under `Environment` section, select `Managed Image`
3. Select `Ubuntu` for the operation system
4. Select `Standard` for the runtime
5. Select `aws/codebuild/standard:3.0` for the image. (Refer the [link](https://docs.aws.amazon.com/codebuild/latest/userguide/build-env-ref-available.html) for which OS and image should be selected based on the lanaguge version.)
6. Under buildsepc, select `insert build commands` option and click `switch to editor` link
7. In the `build commands` text editor, update with the following code and update the Project variable with the sonarqube project name

```YAML
version: 0.2

env:
  variables:
      Project: "spring-unit-testing-with-junit-and-mockito"
  secrets-manager:
      USER: dev/artifactory:username
      PASSWORD: dev/artifactory:password
      URL: dev/artifactory:url
phases:
  install:
    #If you use the Ubuntu standard image 2.0 or later, you must specify runtime-versions.
    #If you specify runtime-versions and use an image other than Ubuntu standard image 2.0, the build fails.
    runtime-versions:
       java: openjdk8
  build:
    commands:
       - mvn deploy -e -X -Dartifactory.url=$URL -Dartifactory.username=$USER -Dartifactory.password=$PASSWORD
artifacts:
  files:
    - '**/*'
  base-directory: 'target'
```
## JFrog Artifactory Configuration
In this section, we create secret manager configuration to store the JFrog Artifactory configurtion.

### Artifactory SecretManager
1. Log into AWS Management Console and select `Secrets Manager`
2. Click on the `Store a new secret` button
3. Select `Other types of secrets` in the Select secret type
4. Under the secret key/value, add the following key/value pairs
   * key: url, value: <paste the JFrog Artifactory URL`.
   * key: username, value: admin
   * key: password, value: <encrypted password>
5. Click `Next` button
6. Enter the secret name value `dev/artifactory` and click `Next` button
7. Click `Next` button
8. Click `Store` button to complete the setup
	
### Grant ServiceRole SecretManager Permission

1. Goto IAM service and search for the service role associated with `Publish Artifacts` build project.
2. Click + Add inline policy to add inline policy to give access to read the secret manager key
3. Click JSON tab and paste the following json snippet:

```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "secretsmanager:GetSecretValue",
            "Resource": "<dev/artifactory secret ARN>"
        }
    ]
}
```
