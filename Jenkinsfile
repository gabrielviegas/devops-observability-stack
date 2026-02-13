pipeline {
    agent any

    environment {
        SSH_CREDS = credentials('vm-ssh-credentials')
        VM_IP = '192.168.56.101'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/gabrielviegas/devops-observability-stack.git'
            }
        }

        stage('Provision VM') {
            steps {
                dir('ansible/playbooks') {
                    sh 'ansible-playbook create_vm.yml'
                }
            }
        }

        stage('Configure Docker') {
            steps {
                dir('ansible/playbooks') {
                    sh "ansible-playbook -i ${VM_IP}, -u ubuntu --private-key $SSH_CREDS_KEY install_docker.yml"
                }
            }
        }

        stage('Deploy Stack') {
            steps {
                sshagent(['vm-ssh-credentials']) {
                    sh "scp -o StrictHostKeyChecking=no docker-compose-stack.yml prometheus/prometheus.yml ubuntu@${VM_IP}:~/"
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${VM_IP} 'docker compose -f docker-compose-stack.yml up -d'"
                }
            }
        }

        stage('Health Check') {
            steps {
                sleep 20
                sh "curl -I http://${VM_IP}:3000"
            }
        }
    }
}