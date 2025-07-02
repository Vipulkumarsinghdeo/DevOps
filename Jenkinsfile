#!groovy
import groovy.json.JsonSlurperClassic
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

    stage('Debug PATH and CLI') {
        sh "echo PATH is \$PATH"
        sh "which sf || which sfdx"
        sh "sf --version || sfdx --version"
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

        stage('Install sfdx-git-delta plugin if not present') {
            sh "sf plugins --core | grep sfdx-git-delta || sf plugins install sfdx-git-delta"
        }

        stage('Generate Delta Changes') {
            // Generate delta between HEAD and previous commit HEAD^
            sh """
                sf sgd:source:delta --to HEAD --from HEAD^ --output .
            """
        }

        stage('Deploy Added/Modified Metadata') {
            // Deploy using generated package/package.xml
            sh """
                sf project deploy start --metadata-dir package --ignore-warnings
            """
        }

        stage('Deploy Destructive Changes if any') {
            // Deploy destructiveChanges if exists
            sh """
                if [ -f destructiveChanges/destructiveChanges.xml ]; then
                    sf project deploy start --metadata-dir destructiveChanges --ignore-warnings
                else
                    echo "No destructive changes to deploy"
                fi
            """
        }
    }
}
