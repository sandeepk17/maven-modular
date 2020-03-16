pipeline {
    agent any
    //agent {
    //    docker {
    //        image "sdkman:local" 
    //    }
    //}
    
    //agent {
    //    dockerfile {
    //        filename DOCKERFILE
    //        additionalBuildArgs "--build-arg JAVA_VERSION=11.0.5-open -t maven:11.0.5-open"
    //    }
    //}
    options {
        timestamps()
        ansiColor("xterm")
    }
    
    tools {
		jdk "jdk8"
		maven "maven3"
	}

    environment {
        FOO = "bar"
        // Set env variables for Pipeline
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
        ARTIFACTORY_SERVER_ID = "Artifactory"
        ARTIFACTORY_URL = "https://artifactory.azure.dsb.dk/artifactory"
        ARTIFACTORY_CREDENTIALS = "Artifactory"
        CURRENT_BUILD_NO = "${currentBuild.number}"
        RELEASE_TAG = "${currentBuild.number}-${VERSION}"
        CURRENT_BRANCH = "${env.BRANCH_NAME}"
    }

    stages {
        stage ('Clone') {
            steps {
                git branch: 'master', url: "https://github.com/sandeepk17/maven-modular.git"
            }
        }

        //stage("openjdk-11.0.5") {
        //    agent {
        //        dockerfile {
        //            filename DOCKERFILE
        //            additionalBuildArgs "--build-arg JAVA_VERSION=11.0.5-open -t maven:11.0.5-open"
        //        }
        //    }
        //    steps {
        //        sh "${MVN_COMMAND} -P jdk11"
        //    }
        //    post {
        //        always {
        //            junit TEST_REPORTS
        //        }
        //    }
        //}

        stage ('Environment variables') {
            steps {
                sh 'printf "\\e[31mPrinting Environment Variables...\\e[0m\\n"'
                echo "The build number is ${env.BUILD_NUMBER}"
                echo "The build name is ${env.JOB_NAME}"
                echo "You can also use \${BUILD_NUMBER} -> ${BUILD_NUMBER}"
                sh 'echo "I can access $BUILD_NUMBER in shell command as well."'
                sh 'printenv'
            }
        }

        stage('Build/sonar Analysis'){
            steps{
                sh "mvn clean package"
            }
        }

        //stage ('Deploy to Artifactory') {
        //    steps {
        //        rtUpload (
        //            buildName: "${env.JOB_NAME}",
        //            buildNumber: "${env.BUILD_NUMBER}",
        //            serverId: "Artifactory", // Obtain an "Artifactory" server instance, defined in Jenkins --> Manage:
        //            specPath: 'resources/props-upload.json'
        //        )
        //    }
        //}
//
        //stage ('Download from Artifactory') {
        //    steps {
        //        rtDownload (
        //            buildName: "${env.JOB_NAME}",
        //            buildNumber: "${env.BUILD_NUMBER}",
        //            serverId: "Artifactory",
        //            specPath: 'resources/props-download.json'
        //        )
        //    }
        //}
//
        //stage ('Publish build info') {
        //    steps {
        //        rtPublishBuildInfo (
        //            buildName: "${env.JOB_NAME}",
        //            buildNumber: "${env.BUILD_NUMBER}",
        //            serverId: "Artifactory"
        //        )
        //    }
        //}

        stage ('Archive & Artifactory Upload') {
            steps {
                echo " Archive and Artifactory Deplyment stage"
                //junit '**/target/surefire-reports/TEST-*.xml'
                zip archive: true, dir: "$WORKSPACE/module2/target", glob: '', zipFile: "module2-snapshot.zip"
                zip archive: true, dir: "$WORKSPACE/module1/target", glob: '', zipFile: "module1-snapshot.zip"
                archiveArtifacts artifacts: "**/*.zip", fingerprint: true
                //rtServer (
                //    id: "${ARTIFACTORY_SERVER_ID}",
                //    url: "${ARTIFACTORY_URL}",
                //    credentialsId: "${ARTIFACTORY_CREDENTIALS}"
                //)

                rtUpload (
                    serverId: "${ARTIFACTORY_SERVER_ID}",
                    spec: '''{
                        "files": [
                            {
                            "pattern": "$WORKSPACE/*.zip",
                            "target": "salgskerne/Test-${RELEASE_TAG}/"
                            }
                        ]
                    }''',
                    buildName: "${env.JOB_NAME}",
                    buildNumber: "${currentBuild.number}"
                )
                
                rtPublishBuildInfo (
                    serverId: "${ARTIFACTORY_SERVER_ID}",
                    buildName: "${env.JOB_NAME}",
                    buildNumber: "${currentBuild.number}"
                )
            }
        }

        stage('Octopus Deploy') {
            steps {
                echo "Deployment to Octopus"

            }
        }

        stage ('Promotion to Release') {
            steps {
                rtPromote (
                    //Mandatory parameter
                    serverId: "Artifactory",
                    targetRepo: 'libs-release-local',

                    //Optional parameters
                    buildName: "${env.JOB_NAME}",
                    buildNumber: "${env.BUILD_NUMBER}",
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
