pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: "5"))
        disableConcurrentBuilds()
    }

    tools {
        dotnetsdk "DOTNET_CORE_3.1.24"
    }

    environment {
        USERNAME      = credentials('esigner_demo')
        PASSWORD      = credentials('Admin@123')
        CREDENTIAL_ID = credentials('es-crendential-id')
        TOTP_SECRET   = credentials('es-totp-secret')

        ENVIRONMENT_NAME = 'TEST' //PROD
    }

    stages {
        stage('Create artifacts directory') {
            steps {
                sh 'mkdir -p ${WORKSPACE}/artifacts'
                sh 'mkdir -p ${WORKSPACE}/packages'
            }
        }

        stage('Docker Pull Image') {
            steps {
                sh 'docker pull ghcr.io/bayrakmustafa/codesigner:latest'
            }
        }

        stage ('Build Dotnet') {
            steps {
                sh 'dotnet build HelloWorld.csproj -c Release'
                sh 'cp bin/Release/netcoreapp3.1/HelloWorld-0.0.1.dll ${WORKSPACE}/packages/HelloWorld.dll'
            }
        }

        stage ('Sign Dotnet Artifact') {
            steps {
                sh 'docker run -i --rm --dns 8.8.8.8 --network host --volume ${WORKSPACE}/packages:/codesign/examples --volume ${WORKSPACE}/artifacts:/codesign/output -e USERNAME=${USERNAME} -e PASSWORD=${PASSWORD} -e CREDENTIAL_ID=${CREDENTIAL_ID} -e TOTP_SECRET=${TOTP_SECRET} -e ENVIRONMENT_NAME=${ENVIRONMENT_NAME} ghcr.io/bayrakmustafa/codesigner:latest sign -input_file_path=/codesign/examples/HelloWorld.dll -output_dir_path=/codesign/output'
            }
            post {
                always {
                    archiveArtifacts artifacts: "artifacts/HelloWorld.dll", onlyIfSuccessful: true
                }
            }
        }
    }
}
