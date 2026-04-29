pipeline {
    agent any
    tools {
        nodejs "node"
    }
        stage('Run tests') {
            steps {
                script {
                    dir("app") {
                        sh "npm install"
                        sh "npm run test"
                    }
                }
            }
        }
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "docker build -t miron163/my_new_node_app:1.0 ."
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push miron163/my_new_node_app:1.0"
                }
            }
        }

        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "git remote set-url origin https://${USER}:${PASS}@github.com/miron1631/jenkins-exercises.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:feature/solutions'
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    echo 'deploying Docker image to EC2 server...'
                    def dockerCmd = "docker run -d -p 3000:3000 miron163/my_new_node_app:1.0"
                    sshagent(['ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no miron@89.169.186.237 ${dockerCmd}"
                    }
                }
            }
        }
    }
}