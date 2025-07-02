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

    stage('Debug PATH and CLI') {
        sh 'echo PATH is $PATH'
        sh 'which sf'
        sh 'sf --version'
    }

    stage('Checkout Source') {
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Authorize Dev Hub Org') {
            sh """
                sf org login jwt \
                    --client-id ${CONNECTED_APP_CONSUMER_KEY} \
                    --jwt-key-file ${jwt_key_file} \
                    --username ${HUB_ORG} \
                    --instance-url ${SFDC_HOST} \
                    --set-default-dev-hub
            """
        }

        stage('Deploy Code') {
            sh """
                sf project deploy start \
                    --manifest manifest/package.xml \
                    --target-org ${HUB_ORG}
            """
        }
    }

    println('Pipeline completed successfully.')
}
