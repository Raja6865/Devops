pipeline {
    agent any
    tools {
        maven 'maven'
    }
    stages {
        stage ("scm") {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/Raja6865/Devops.git']]])
            }
        }
        stage ("sonar analasys") {
            environment {
                scannerHome = tool 'sonarscanner'
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage ("quality gate check") {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ("Maven Build in Pipelinemode") {
            steps {
                sh 'mvn package'
            }
        }
        stage ("Nexus store Artifact") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'simple-web-app', classifier: '', file: 'target/simple-web-app.war', type: 'war']], credentialsId: 'NexusID', groupId: 'org.mitre', nexusUrl: '54.80.23.150:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'Maven_Hosted', version: '3.1.14'
                
            }
            
        }
        stage ("Deployment in Tomcat") {
            steps {
                sh "curl http://54.80.23.150:8081/repository/Maven_Hosted/org/mitre/simple-web-app/3.1.14/simple-web-app-3.1.14.war /opt/apache-tomcat-8.5.51/webapps/"
            }
        }
    }
    post {
    success {
        emailext (
            to: 'rajasekhar.raja50@gmail.com',
            subject: "JOB: ${env.JOB_NAME} - SUCCESS",
            body: "JOB SUCCESS - \"${env.JOB_NAME}\" Build No: ${env.BUILD_NUMBER}\n\nClick on the below link to view the logs:\n ${env.BUILD_URL}\n"
        )
    }
    failure {
		emailext (
            to: 'rajasekhar.raja50@gmail.com',
            subject: "JOB: ${env.JOB_NAME} - FAILURE",
            body: "JOB FAILURE - \"${env.JOB_NAME}\" Build No: ${env.BUILD_NUMBER}\n\nClick on the below link to view the logs:\n ${env.BUILD_URL}\n"
        )
    }
    }
}
