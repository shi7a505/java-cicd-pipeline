def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK21"
    }

    stages {

        stage('Test Slack') {
            steps {
                sh 'echo "Slack test step"'
            }
        }

        stage('Fetch Code') {
            steps {
                git branch: 'main', url: 'https://github.com/shi7a505/java-cicd-pipeline.git'
            }
        }

        stage('Build') {
            steps {
                dir('vprofile-app') {
                    sh 'mvn install -DskipTests'
                }
            }

            post {
                success {
                    dir('vprofile-app') {
                        echo 'Archiving WAR file...'
                        archiveArtifacts artifacts: '**/target/*.war'
                    }
                }
            }
        }

        stage('Unit Test') {
            steps {
                dir('vprofile-app') {
                    sh 'mvn test'
                }
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                dir('vprofile-app') {
                    sh 'mvn checkstyle:checkstyle'
                }
            }
        }

        stage('Sonar Code Analysis') {
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
                dir('vprofile-app') {
                    withSonarQubeEnv('sonarserver') {
                        sh '''${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=vprofile \
                          -Dsonar.projectName=vprofile \
                          -Dsonar.projectVersion=1.0 \
                          -Dsonar.sources=src/ \
                          -Dsonar.java.binaries=target/ \
                          -Dsonar.junit.reportsPath=target/surefire-reports/ \
                          -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                          -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                    }
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Upload Artifact to Nexus") {
            steps {
                dir('vprofile-app') {
                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '172.31.25.14:8081',
                        groupId: 'QA',
                        version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                        repository: 'vprofile-repo',
                        credentialsId: 'nexuslogin',
                        artifacts: [
                            [artifactId: 'vproapp',
                             classifier: '',
                             file: 'target/vprofile-v2.war',
                             type: 'war']
                        ]
                    )
                }
            }
        }
    }

    post {
        always {
            echo 'Sending Slack Notification...'
            slackSend channel: '#devopscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\nMore info: ${env.BUILD_URL}"
        }
    }
}
