pipeline { 
    environment { 
        registry = "theja27/python-flask-app1" 
        registryCredential = 'docker_id' 
        dockerImage = '' 
    }
    agent any
    stages {
        stage('Cloning the Code') {
            steps {
                echo 'Cloning the Code from Git'
                git branch:'main', url: 'https://github.com/Theja27/python-flask-app.git'
            }
        }
        stage('install req') {
            steps {
                echo 'installing'
                sh 'pip install flake8==4.0.1'
                sh 'pip install flake8-json==21.7.0'
            }
        }   
        stage('SCA using Bandit') {
            steps {
                echo 'Scanning the Source Code using Bandit'
                sh 'docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm secfigo/bandit bandit -r /src -f json -o /src/bandit-output.json | exit 0'
            }
        }    
        stage('Testing') {
            parallel {
                stage('SCA using Bandit') {
                    steps {
                        echo 'Scanning the Source Code using Bandit'
                        sh 'docker run --user $(id -u):$(id -g) -v $(pwd):/src --rm secfigo/bandit bandit -r /src -f json -o /src/bandit-output.json | exit 0'
                    }
                }
                stage('Git Secret using TruffleHog') {
                    steps {
                        echo 'Scan the git repo using Trufflehog'
                        sh 'docker run --user $(id -u):$(id -g) --rm -v "$(pwd):/proj" dxa4481/trufflehog file:///proj --json | tee trufflehog-output.json'
                    }
                }
                stage('Flake8') {
                    steps {
                        echo 'Flake8 Scaning'
                        sh 'docker pull alpine/flake8'
                        sh 'docker run -ti --rm -v $(pwd):/apps alpine/flake8:3.5.0 | tee flask_op.txt'
                    }
                }
                
            }
        }        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('',registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Delete Docker Image from Local Computer') {
            steps {
                echo 'Deleting the Docker Image'
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }
        stage('Deploy to Kubernetes Dev Environment') {
            steps {
                echo 'Deploy the App using Kubectl'
                //sh "sed -i 's/BUILDNUMBER/$BUILD_NUMBER/g' python-flask-deployment.yml"
                sh "sed -i 's/DEPLOYMENTENVIRONMENT/development/g' python-flask-deployment.yml"
                sh "sed -i 's/TAG/$BUILD_NUMBER/g' python-flask-deployment.yml"
                sh "kubectl apply -f python-flask-deployment.yml"
            }
        }
        stage('Promote to Production') {
            steps {
                echo "Promote to production"
            }
            input {
                message "Do you want to Promote the Build to Production"
                ok "Ok"
                submitter "pathinishant@gmail.com"
                submitterParameter "whoIsSubmitter"
                
            }
        }
        stage('Deploy to Kubernetes Production Environment') {
            steps {
                echo 'Deploy the App using Kubectl'
                sh "sed -i 's/development/production/g' python-flask-deployment.yml"
                sh "sed -i 's/TAG/$BUILD_NUMBER/g' python-flask-deployment.yml"
                sh "kubectl apply -f python-flask-deployment.yml"
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'bandit-output.json',onlyIfSuccessful: true
        }
    }
}