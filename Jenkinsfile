pipeline {
    agent any

    parameters {
        string(name: 'ANSIBLE_INVENTORY_FILE', defaultValue: '/path/to/ansible/hosts', description: 'Ansible inventory file path')
        string(name: 'ANSIBLE_PLAYBOOK', defaultValue: 'site.yml', description: 'Ansible playbook to run')
    }

    environment {
        GITHUB_CREDENTIALS = credentials('github-ssh')
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/YOUR_USERNAME/YOUR_REPO.git',
                        credentialsId: 'github-ssh'
                    ]]
                ])
            }
        }

        stage('Detect Changes') {
            steps {
                script {
                    echo 'Detecting changes...'
                    sh '''
                        git diff HEAD~1 HEAD --name-only | grep -E "ansible|config" || echo "No ansible changes detected"
                    '''
                }
            }
        }

        stage('Validate Ansible') {
            steps {
                echo 'Validating Ansible syntax...'
                sh '''
                    ansible-playbook --syntax-check ${ANSIBLE_PLAYBOOK}
                '''
            }
        }

        stage('Run Ansible Playbook') {
            steps {
                echo 'Running Ansible playbook...'
                sh '''
                    ansible-playbook \
                        -i ${ANSIBLE_INVENTORY_FILE} \
                        -u ec2-user \
                        --private-key=/path/to/your/private/key.pem \
                        ${ANSIBLE_PLAYBOOK}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployment...'
                sh '''
                    curl -s http://$(cat ${ANSIBLE_INVENTORY_FILE} | grep -oP '\\d+\\.\\d+\\.\\d+\\.\\d+'):80 | head -n 20
                '''
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
