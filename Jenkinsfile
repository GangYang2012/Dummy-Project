pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "php-bucket11"
        ApplicationName = "personal-testing"
        EnvironmentName = "Personal-testing-env "
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh "cd ${env.WORKSPACE} && zip -j ${BuildName}.zip *"
                    sh "aws s3 cp ${BuildName}.zip s3://${BucketName} --region us-east-1"
                    sh "rm ${BuildName}.zip"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        aws elasticbeanstalk create-application-version \
                            --application-name "${ApplicationName}" \
                            --version-label "${BuildName}" \
                            --description "Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}" \
                            --source-bundle S3Bucket=${BucketName},S3Key=${BuildName}.zip \
                            --region us-east-2

                        aws elasticbeanstalk update-environment \
                            --environment-name "${EnvironmentName}" \
                            --version-label "${BuildName}" \
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
                        sh "aws s3 rm s3://${BucketName}/${versions[i]}.zip --region us-east-1"
                    }
                }
            }
        }
    }
}
