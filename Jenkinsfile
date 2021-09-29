pipeline {
    agent any
    stages {
        stage('Checkout') {
            environment {
                dockerHome = tool 'Docker'
            }
            steps {
                script {
                    env.PATH = "${dockerHome}/bin:${env.PATH}"
                    sh "echo $PATH"
                    sh "echo $dockerHome"
                }
            }
        }
        stage('Run test') {
            steps {
                sh "docker version"
                sh "docker run --mount type=bind,source=/var/lib/jenkins/workspace/JENKINS-BOOT,target=/usr/src/myapp/bin dinonel/inkathon"
            }
        }
        stage('Sonar') {
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }
            steps {
                echo 'Sonar Scanner'
                withSonarQubeEnv('SonarQube Server') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectVersion=${BUILD_ID}"
                }
            }
        }
        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
    }

    post {
        success {
            echo 'JENKINS PIPELINE SUCCESSFUL'
        }
        failure {
            emailext body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                recipientProviders: [[$class: 'RequesterRecipientProvider'], [$class:'DevelopersRecipientProvider']],
                subject: "Jenkins Build ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
        }
        unstable {
            echo 'JENKINS PIPELINE WAS MARKED AS UNSTABLE'
        }
        changed {
            echo 'JENKINS PIPELINE STATUS HAS CHANGED SINCE LAST EXECUTION'
        }
    }
}