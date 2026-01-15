pipeline {
    agent any

    environment {
        // Disable host key checking for lab environment
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Syntax Check') {
            steps {
                sh '''
                ansible-playbook \
                  -i inventory/hosts.yml \
                  playbooks/router_config.yml \
                  --syntax-check
                '''
            }
        }

        stage('Deploy Configuration') {
            steps {
                echo 'Deploying to Cisco Routers...'

                withCredentials([usernamePassword(
                    credentialsId: 'cisco-ssh',
                    usernameVariable: 'ANSIBLE_USER',
                    passwordVariable: 'ANSIBLE_PASS'
                )]) {
                    sh '''
                    ansible-playbook \
                      -i inventory/hosts.yml \
                      playbooks/router_config.yml \
                      -e ansible_user=$ANSIBLE_USER \
                      -e ansible_password=$ANSIBLE_PASS
                    '''
                }
            }
        }
    }
}
