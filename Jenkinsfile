pipeline {
    agent any

    environment {
        // Force Ansible to use local config
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
        // Disable Host Key Checking (Safety net)
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        // Make output pretty
        // ANSIBLE_FORCE_COLOR = 'true'
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
                    // SECURE MODE: We inject the password from Jenkins
                    // Quoting "$ANSIBLE_PASS" handles special chars safely
                    sh '''
                    ansible-playbook \
                      -i inventory/hosts.yml \
                      playbooks/router_config.yml \
                      -e ansible_user="$ANSIBLE_USER" \
                      -e ansible_password="$ANSIBLE_PASS" \
                      -vvv
                    '''
                }
            }
        }
    }
}
