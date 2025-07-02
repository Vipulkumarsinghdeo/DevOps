#!groovy
import groovy.json.JsonSlurperClassic

node {

    def BUILD_NUMBER = env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR = "tests/${BUILD_NUMBER}"

    def HUB_ORG = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH

    println 'KEY IS' 
    println JWT_KEY_CRED_ID
    println HUB_ORG
    println SFDC_HOST
    println CONNECTED_APP_CONSUMER_KEY

    stage('Debug PATH and Shell') {
        echo "Checking PATH and shell environment"
        sh 'echo "Effective PATH is: $PATH"'
        sh 'which sh'
        sh 'which sfdx'
        sh 'sfdx --version'
    }

    stage('Checkout Source') {
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {

        stage('Authorize Dev Hub Org') {
            int rc = sh returnStatus: true, script: """
                sfdx auth:jwt:grant \
                  --clientid ${CONNECTED_APP_CONSUMER_KEY} \
                  --username ${HUB_ORG} \
                  --jwtkeyfile $jwt_key_file \
                  --setdefaultdevhubusername \
                  --instanceurl ${SFDC_HOST}
            """

            if (rc != 0) { 
                error 'Dev Hub org authorization failed'
            } else {
                println '✅ Dev Hub org authorized successfully'
            }
        }

        stage('Deploy Code') {
            def rmsg = sh returnStdout: true, script: """
                sfdx force:source:deploy -x manifest/package.xml -u ${HUB_ORG}
            """
            println "🚀 Deployment Output:"
            println rmsg
        }

        stage('Post Deploy') {
            println('✅ Pipeline completed successfully.')
        }
    }
}
