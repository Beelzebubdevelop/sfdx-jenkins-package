# sfdx-jenkins-package

For a fully guided walk through of setting up and configuring this sample, see the [Continuous Integration Using Salesforce DX](https://trailhead.salesforce.com/modules/sfdx_travis_ci) Trailhead module.

This repository shows one way you can successfully setup Salesforce DX to create new package versions with Jenkins. We make a few assumptions in this README:

- You know how to get your repository setup with Jenkins. (Here's their [Getting Started guide](https://jenkins.io/doc/pipeline/tour/getting-started/).)
- You have properly setup JWT-Based Authorization Flow (i.e. headless). We recommended using [these steps for generating your Self-Signed SSL Certificate](https://devcenter.heroku.com/articles/ssl-certificate-self). 

If any any of these assumptions aren't true, the following steps won't work.

## Getting Started

1) Make sure you have the Salesforce CLI installed. Check by running `sfdx force --help` and confirm you see the command output. If you don't have it installed you can download and install it from [here](https://developer.salesforce.com/tools/sfdxcli).

2) Setup a [custom tool](https://wiki.jenkins.io/display/JENKINS/Custom+Tools+Plugin) in Jenkins for the Salesforce CLI. Name the custom tool `toolbelt` and set its installation directory to the path where the `sfdx` executable is (e.g., `/usr/local/bin/sfdx`).

3) [Fork](http://help.github.com/fork-a-repo/) this repo into your GitHub account using the fork link at the top of the page.

4) Clone your forked repo locally: `git clone https://github.com/<git_username>/sfdx-jenkins-package.git`

5) Setup a JWT-based auth flow for the target orgs that you want to deploy to. This step will create a server.key file that will be used in subsequent steps.
(https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_jwt_flow.htm)

6) Confirm you can perform a JWT-based auth: `sfdx force:auth:jwt:grant --clientid <your_consumer_key> --jwtkeyfile server.key --username <your_username> --setdefaultdevhubusername`

   **Note:** For more info on setting up JWT-based auth see [Authorize an Org Using the JWT-Based Flow](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_jwt_flow.htm) in the [Salesforce DX Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev).

7) From your JWT-Based connected app on Salesforce, retrieve the generated `Consumer Key`.

8) Setup Jenkins [environment variables](https://jenkins.io/doc/book/using/using-credentials/) for your Salesforce `Consumer Key` and `Username`. Note that this username is the username that you use to access your Salesforce org.

    Create an environment variable named `SF_CONSUMER_KEY`.

    Create an environment variable named `SF_USERNAME`.

9) Store your `server.key` file that you generated previously as a Jenkins Secret File using the [Jenkins Admin Credentials interface](https://wiki.jenkins.io/display/JENKINS/Credentials+Binding+Plugin). Make note of the new entry's ID.

10) Setup Jenkins [environment variable](https://gitlab.com/help/ci/variables/README#variables) to store the ID of the secret file you created.

    Create an environment variable named `SERVER_KEY_CREDENTALS_ID`.

11) Create the sample package running this command:

    `sfdx force:package:create --path force-app/main/default/ --name "Jenkins" --description "Jenkins Package Example" --packagetype Unlocked`

12) Create the first package version.

    `sfdx force:package:version:create --package "Jenkins" --installationkeybypass --wait 10 --json --targetdevhubusername HubOrg`

13) In the `Jenkinsfile` file update the value in the `PACKAGENAME` variable to be Package Id in your sfdx-project.json file.  This id will start with `0Ho`.

14) Commit the updated `sfdx-project.json` and `Jenkinsfile` files.

15) Create a Jenkins pipline with the `Jenkinsfile` included in the root directory of the git repository.

And you should be ready to go! Now when you commit and push a change, your change will kick off a Jenkins build.

Enjoy!

## Contributing to the Repository ###

If you find any issues or opportunities for improving this repository, fix them!  Feel free to contribute to this project by [forking](http://help.github.com/fork-a-repo/) this repository and make changes to the content.  Once you've made your changes, share them back with the community by sending a pull request. Please see [How to send pull requests](http://help.github.com/send-pull-requests/) for more information about contributing to Github projects.

## Reporting Issues ###

If you find any issues with this demo that you can't fix, feel free to report them in the [issues](https://github.com/forcedotcom/sfdx-jenkins-package/issues) section of this repository.
