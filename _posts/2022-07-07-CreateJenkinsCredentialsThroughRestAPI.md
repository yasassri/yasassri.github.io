---
title: Create Jenkins Credentials Through Rest API
description: Create Jenkins Credentials Through Rest API
date: '2022-07-10'
categories: [CICD]
keywords: []
tags: [jenkins, cicd, devops, groovy, automation]
image:
  path: /assets/img/posts/jenkins.jpg
  width: 800
  height: 500
  alt:
---

In this post I'll explain how you can create Jenkins credentials through the Jenkins Rest API. Let's get into it straight away.

### How To Authenticate Jenkins API

Inorder to Authenticate the API call you have to pass an API token and the Jenkins Crump with the API call. Jenkins Crunb was introduced to prevent CSRF attacks. Let's see how exactly you can get all these details.

$JENKINS_URL : This variable refers to Jenkins URL with the custom context if you have any.
$JENKINS_USER : Username of the user used to generate the access token
$JENKINS_PASSWORD : Password of the user used to generate the access token
$API_ACCESS_TOKEN : The access token.
$JENKINS_CRUMB ; Jenkis Crumb. 

1. First we need to get the Jenkins `crumb` by passing the `Basic Auth` header. We also need to save the Cookies so we can use the same Cookies when doing the API request for this I'm using `--cookie-jar` option with `curl`.

```sh
curl -s --cookie-jar /tmp/cookies -u $JENKINS_USER:$JENKINS_PASSWORD $JENKINS_URL/crumbIssuer/api/json
```
The above will give you a response like the below.

```json
{
  "_class": "hudson.security.csrf.DefaultCrumbIssuer",
  "crumb": "e6aa50dfdda70b3db256d27a1effe7e0be5033b94d9edeaa9e108c212e91f4c2",
  "crumbRequestField": "Jenkins-Crumb"
}
```

From the above, you can extract the `crumb` value and pass it with the header `Jenkins-Crumb` to generate a token. 

2. Send the following `curl` request to Generate an Access Token. 

```sh
curl -u "$JENKINS_USER:$JENKINS_USER_PASS" -H $JENKINS_CRUMB -s \
          --cookie /tmp/cookies $JENKINS_URL'/me/descriptorByName/jenkins.security.ApiTokenProperty/generateNewToken' \
          --data 'newTokenName=GlobalToken'
```
You will get the following response for the above call. Extract the value for `tokenValue` and use it as the Access Token on consecutive API calls. 
```json
{
  "status": "ok",
  "data": {
    "tokenName": "GlobalToken",
    "tokenUuid": "cef3f33d-5e61-4d5e-a966-44d52546f5aa",
    "tokenValue": "1135b180fcc6ba2cbc0d3fb04621d8700a"
  }
}
```
#### Summary
Following are all of the above commands together. You can execute all of the following commands and generate an Access token. 

Note: Following commands need `curl` and `jq`. Execute in the same session. 
```sh
# Change the following appropriately
JENKINS_URL="http://localhost:8080"
JENKINS_USER=admin
JENKINS_USER_PASS=admin

# Get the Crumb**

JENKINS_CRUMB=$(curl -u "$JENKINS_USER:$JENKINS_USER_PASS" -s --cookie-jar /tmp/cookies $JENKINS_URL'/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)')

#Get the Access token**
ACCESS_TOKEN=$(curl -u "$JENKINS_USER:$JENKINS_USER_PASS" -H $JENKINS_CRUMB -s \
                    --cookie /tmp/cookies $JENKINS_URL'/me/descriptorByName/jenkins.security.ApiTokenProperty/generateNewToken' \
                    --data 'newTokenName=GlobalToken' | jq -r '.data.tokenValue')
```

### Creating credentials

You need to generate the approprite payload to create Credentials depending on the type of the credentials. You can refer the following for this. 

#### Payload for Create Username Password Credential

```xml
<com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
  <id>TestCredentials</id>
  <description>This is sample</description>
  <username>admin2</username>
  <password>admin2</password>
</com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl>
```

#### Payload for SSH User Private Key

```xml
<com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey>
  <id>SSHCredential</id>
  <description></description>
  <username>ubuntu</username>
  <usernameSecret>false</usernameSecret>
  <privateKeySource class="com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey$DirectEntryPrivateKeySource">
    <privateKey>PRIVATEKEY_HERE</privateKey>
  </privateKeySource>
</com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey>
```
#### Payload for Github App Credentials
```xml
<org.jenkinsci.plugins.github__branch__source.GitHubAppCredentials>
  <id>GIthubApp_YCR</id>
  <description></description>
  <appID>GitAppID</appID>
  <privateKey>PRIVATE_KEY</privateKey>
  <apiUri></apiUri>
  <owner>OWNER</owner>
</org.jenkinsci.plugins.github__branch__source.GitHubAppCredentials>
```

#### Payload for Secret Text Content Credential

```xml
<org.jenkinsci.plugins.plaincredentials.impl.StringCredentialsImpl>
  <id>SecretTextYcr</id>
  <description></description>
  <secret>SECRET_TEXT</secret>
</org.jenkinsci.plugins.plaincredentials.impl.StringCredentialsImpl>
```

#### Payload for X509 Cert/Docker Server Credential

```xml
<org.jenkinsci.plugins.docker.commons.credentials.DockerServerCredentials>
  <id>X509Cert</id>
  <description></description>
  <clientKey>CLIENT_KEY</clientKey>
  <clientCertificate>CLIENT_CERTIFICATE</clientCertificate>
  <serverCaCertificate>SERVER_CERT</serverCaCertificate>
</org.jenkinsci.plugins.docker.commons.credentials.DockerServerCredentials>
```

Add the content of the payload to a file named `credentials.xml`

#### Constructing the API URL for creating credentials

The credential create URL format is `JENKINS_URL/credentials/store/CREDENTIALS_STORE_NAME/domain/DOMAIN_NAME/` You need to change this appropriately based on the location and the domain you are creating the credentials under. The easiest way to get this URL is by navigating to an existing credential from the UI and copying the URL.

![](/assets/img/posts/JenkinsURL.png){: w="300" h="500" }

Once you figure out the correct URL(context path) for the API call execute the following command.

```sh
curl -u $JENKINS_USER:$ACCESS_TOKEN \
    -H $JENKINS_CRUMB \
    -H 'content-type:application/xml' \
    "$JENKINS_URL/credentials/store/system/domain/_/createCredentials" \
    -d @credentials.xml
```

Hope the above helps!! Happy Coding. 