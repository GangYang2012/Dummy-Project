pipeline {
    agent any

    environment {
        BUILD_NAME = "version-${BUILD_NUMBER}"
        BUCKET_NAME = "php-bucket11"
        ApplicationName = "php-testing-app"
        EnvironmentName = "Php-testing-app-env"
        S3Key = "path/to/${BUILD_NAME}.zip" // Specify the S3 Key with a relative path
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Change to the workspace directory
                    dir("/var/lib/jenkins/workspace/php-pipeline/") {
                        sh "zip -r ${BUILD_NAME}.zip ."
                        sh "ls -l ${BUILD_NAME}.zip"
                        sh "aws s3 cp ${BUILD_NAME}.zip s3://${BUCKET_NAME}/${S3Key} --region us-east-1"
                        sh "rm -rf ./*"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        aws elasticbeanstalk create-application-version \
                            --application-name "${ApplicationName}" \
                            --version-label "${BUILD_NAME}" \
                            --description "Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}" \
                            --source-bundle S3Bucket=${BUCKET_NAME},S3Key=${S3Key} \
                            --region us-east-1

                        aws elasticbeanstalk update-environment \
                            --environment-name "${EnvironmentName}" \
                            --version-label "${BUILD_NAME}" \
                            --region us-east-1
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Specify the number of versions to keep
                    def versionsToKeep = 2

                    // Get the list of application versions
                    def versions = sh(script: "aws elasticbeanstalk describe-application-versions --application-name ${ApplicationName} --region us-east-1 --query 'ApplicationVersions[*].VersionLabel' --output text", returnStdout: true).trim().split()

                    // Sort versions in descending order
                    versions.sort { a, b -> b.compareTo(a) }

                    // Remove excess versions and corresponding artifacts from S3
                    for (int i = versionsToKeep; i < versions.size(); i++) {
                        sh "aws elasticbeanstalk delete-application-version --application-name ${ApplicationName} --version-label ${versions[i]} --region us-east-1"
                        sh "aws s3 rm s3://${BUCKET_NAME}/path/to/${versions[i]}.zip --region us-east-1"
                    }
                }
            }
        }
    }
}