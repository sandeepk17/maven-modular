pipeline {
    agent any
    environment {
        FOO = "bar"
    }

    stages {
        stage ('Clone') {
            steps {
                git branch: 'master', url: "https://github.com/sandeepk17/maven-modular.git"
            }
        }

        stage ('Environment variables') {
            steps {
                echo "The build number is ${env.BUILD_NUMBER}"
                echo "The build name is ${env.JOB_NAME}"
                echo "You can also use \${BUILD_NUMBER} -> ${BUILD_NUMBER}"
                sh 'echo "I can access $BUILD_NUMBER in shell command as well."'
                sh 'printenv'
            }
        }

        stage('Build/sonar Analysis'){
            steps{
                echo "Build and static code analysis"
            }
        }

        stage ('Deploy to Artifactory') {
            steps {
                rtUpload (
                    buildName: 'MK',
                    buildNumber: '48',
                    serverId: "Artifactory", // Obtain an "Artifactory" server instance, defined in Jenkins --> Manage:
                    specPath: 'resources/props-upload.json'
                )
            }
        }

        stage ('Download from Artifactory') {
            steps {
                rtDownload (
                    buildName: 'MK',
                    buildNumber: '48',
                    serverId: "Artifactory",
                    specPath: 'resources/props-download.json'
                )
            }
        }

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    buildName: 'MK',
                    buildNumber: '48',
                    serverId: "Artifactory"
                )
            }
        }

        stage ('Promotion to Release') {
            steps {
                rtPromote (
                    //Mandatory parameter
                    serverId: "Artifactory",
                    targetRepo: 'libs-release-local',

                    //Optional parameters
                    buildName: 'MK',
                    buildNumber: '48',
                    comment: 'this is the promotion comment',
                    sourceRepo: 'libs-snapshot-local',
                    status: 'Released',
                    includeDependencies: true,
                    failFast: true,
                    copy: true
                )
            }
        }
    }
}
