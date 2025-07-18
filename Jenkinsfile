pipeline {
    agent any

    stages {
        stage('Init') {
            steps {
                sh 'docker rm -f $(docker ps -qa) || true'
                sh 'docker network create new-network || true'
            }
        }
        stage('Build') {
            steps {
                sh 'docker build -t flask-app .'
                sh 'docker build -t mynginx -f Dockerfile.nginx .'
            }
        }

        stage("Security Scan") {
            steps {
                sh "trivy fs --format json -o trivy-report.json ."
            }
            post {
                always {
                    // Archive the Trivy report
                    archiveArtifacts artifacts: 'trivy-report.json', onlyIfSuccessful: true
                }
            }
        }

        stage('Approval to continue') {
            steps {
                input message: 'Do you want to proceed?', ok: 'Yes'
            }
        }

         stage('Deploy') {
            steps {
                sh 'docker run -d --name flask-app --network new-network flask-app:latest'
                sh 'docker run -d -p 80:80 --name mynginx --network new-network mynginx:latest'
            }
        }

        stage('Execute Tests') {
         //   options {
        //allowFailure true
           // timeout(time: 30, unit: 'MINUTES')
            //disableConcurrentBuilds()
    //}
            steps {
                script {
                    catchError(buildResult: 'UNSTABLE', stageResult: 'UNSTABLE'/*failure/success/aborted*/){
                // sudo apt install pythjon3-venv on ec2 first!!!!
                        sh '''
                python3 -m venv .venv
                . .venv/bin/activate
                pip install -r requirements.txt
                python3 -m unittest discover -s tests .
                deactivate
                '''
                    }
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}

