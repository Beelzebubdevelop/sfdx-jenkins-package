#!groovy

import groovy.json.JsonSlurperClassic

node {

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTALS_ID=env.SERVER_KEY_CREDENTALS_ID
    def TEST_LEVEL='RunLocalTests'
    def PACKAGE_NAME='0Ho1U000000CaUzSAK'
    def PACKAGE_VERSION


    def toolbelt = tool 'toolbelt'


    // -------------------------------------------------------------------------
    // Checkout code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }


    // -------------------------------------------------------------------------
    // Run all of the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------

    withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'server_key_file')]) {

        // -------------------------------------------------------------------------
        // Authorize dev hub with JWT key and give it an alias.
        // -------------------------------------------------------------------------

        stage('Authorize DevHub') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:auth:jwt:grant --clientid ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwtkeyfile ${server_key_file} --setdefaultdevhubusername --setalias HubOrg"
            if (rc != 0) {
                error 'Salesforce dev hub org authorization failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Create new scratch org to test code in.
        // -------------------------------------------------------------------------

        stage('Create Test Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias ciorg --wait 10 --durationdays 1"
            if (rc != 0) {
                error 'Salesforce test scratch org creation failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Display test scratch org info.
        // -------------------------------------------------------------------------

        stage('Display Test Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:org:display --targetusername ciorg"
            if (rc != 0) {
                error 'Salesforce test scratch org display failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Push source to test scratch org.
        // -------------------------------------------------------------------------

        stage('Push To Test Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:source:push --targetusername ciorg"
            if (rc != 0) {
                error 'Salesforce push to test scratch org failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Run unit tests in test scratch org.
        // -------------------------------------------------------------------------

        stage('Run Tests In Test Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:apex:test:run --targetusername ciorg --wait 10 --resultformat tap --codecoverage --testlevel ${TEST_LEVEL}"
            if (rc != 0) {
                error 'Salesforce unit test run in test scratch org failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Delete test scratch org.
        // -------------------------------------------------------------------------

        stage('Delete Test Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:org:delete --targetusername ciorg --noprompt"
            if (rc != 0) {
                error 'Salesforce test scratch org deletion failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Create package version.
        // -------------------------------------------------------------------------

        stage('Create Package Version') {
            output = sh returnStdout: true, script: "${toolbelt}/sfdx force:package:version:create --package ${PACKAGE_NAME} --installationkeybypass --wait 10 --json --targetdevhubusername HubOrg"
            printf output

            def jsonSlurper = new JsonSlurperClassic()
            def response = jsonSlurper.parseText(output)

            PACKAGE_VERSION = response.result.SubscriberPackageVersionId

            response = null
        }


        // -------------------------------------------------------------------------
        // Create new scratch org to install package to.
        // -------------------------------------------------------------------------

        stage('Create Package Install Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:org:create --targetdevhubusername HubOrg --setdefaultusername --definitionfile config/project-scratch-def.json --setalias installorg --wait 10 --durationdays 1"
            if (rc != 0) {
                error 'Salesforce package install scratch org creation failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Display install scratch org info.
        // -------------------------------------------------------------------------

        stage('Display Install Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:org:display --targetusername installorg"
            if (rc != 0) {
                error 'Salesforce install scratch org display failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Install package in package scratch org.
        // -------------------------------------------------------------------------

        stage('Install Package In Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:package:install --package ${PACKAGE_VERSION} --targetusername installorg --wait 10"
            if (rc != 0) {
                error 'Salesforce package install failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Run unit tests in package install scratch org.
        // -------------------------------------------------------------------------

        stage('Run Tests In Package Install Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:apex:test:run --targetusername installorg --resultformat tap --codecoverage --testlevel ${TEST_LEVEL} --wait 10"
            if (rc != 0) {
                error 'Salesforce unit test run in pacakge install scratch org failed.'
            }
        }


        // -------------------------------------------------------------------------
        // Delete package intall scratch org.
        // -------------------------------------------------------------------------

        stage('Delete Package Install Scratch Org') {
            rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:org:delete --targetusername installorg --noprompt"
            if (rc != 0) {
                error 'Salesforce package install scratch org deletion failed.'
            }
        }
    }
}