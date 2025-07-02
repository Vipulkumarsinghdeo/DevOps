#!groovy
import groovy.json.JsonSlurperClassic

node {

    def BUILD_NUMBER = env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR = "tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY

    stage('Checkout Source') {
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {

        stage('Authorize Dev Hub Org') {
            int rc

            if (isUnix()) {
                rc = sh returnStatus: true, script: """
                    sfdx auth:jwt:grant \
                      --clientid ${CONNECTED_APP_CONSUMER_KEY} \
                      --username ${HUB_ORG} \
                      --jwtkeyfile ${jwt_key_file} \
                      --setdefaultdevhubusername \
                      --instanceurl ${SFDC_HOST}
                """
            } else {
                rc = bat returnStatus: true, script: """
                    sfdx auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --loglevel DEBUG --setdefaultdevhubusername --instanceurl ${SFDC_HOST}
                """
            }

            if (rc != 0) { 
                error 'Dev Hub org authorization failed'
            } else {
                println 'Dev Hub org authorized successfully'
            }
        }

        stage('Deploy Code') {
            def rmsg

            if (isUnix()) {
                rmsg = sh returnStdout: true, script: "sfdx force:source:deploy -x manifest/package.xml -u ${HUB_ORG}"
            } else {
                rmsg = bat returnStdout: true, script: "sfdx force:source:deploy -x manifest\\package.xml -u ${HUB_ORG}"
            }

            println rmsg
        }

        stage('Post Deploy') {
            println('Hello from the Job DSL script!')
        }
    }
}
