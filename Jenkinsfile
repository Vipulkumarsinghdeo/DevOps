#!groovy
import groovy.json.JsonSlurperClassic

node {

    def BUILD_NUMBER = env.BUILD_NUMBER
    def SFDC_USERNAME

    def HUB_ORG = env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH

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
            sh """
                sf plugins --core | grep sfdx-git-delta || sf plugins install sfdx-git-delta --no-verify
            """
        }

        stage('Generate Delta Changes') {
            sh """
                sf sgd:source:delta --to HEAD --from HEAD^ --output .
            """
        }

        stage('Ensure Meta XML files are included') {
            sh '''
                echo "Ensuring meta.xml files are included in delta package..."

                copy_meta() {
                    folder=$1
                    extension=$2

                    echo "Processing $folder for *.$extension files"

                    cd force-app/main/default/$folder || exit

                    for f in *.$extension; do
                        if [ -f "../../../package/$folder/$f" ]; then
                            meta="${f}-meta.xml"
                            if [ -f "$meta" ] && [ ! -f "../../../package/$folder/$meta" ]; then
                                echo "Copying missing $meta for $f"
                                mkdir -p ../../../package/$folder
                                cp "$meta" ../../../package/$folder/
                            fi
                        fi
                    done

                    cd - > /dev/null
                }

                # Copy for Apex classes
                copy_meta classes cls

                # Copy for Apex triggers
                copy_meta triggers trigger

                # Copy for Aura components
                copy_meta aura cmp

                # Copy for Lightning Web Components
                copy_meta lwc js

                # Copy for Visualforce pages
                copy_meta pages page

                echo "Meta XML copy completed."
            '''
        }

        stage('Deploy Added/Modified Metadata') {
            sh """
                sf project deploy start --metadata-dir package --ignore-warnings --target-org ${HUB_ORG}
            """
        }

        stage('Deploy Destructive Changes if any') {
            sh """
                if [ -f destructiveChanges/destructiveChanges.xml ]; then
                    sf project deploy start --metadata-dir destructiveChanges --ignore-warnings --target-org ${HUB_ORG}
                else
                    echo "No destructive changes to deploy"
                fi
            """
        }
    }
}
