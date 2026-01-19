pipeline {
    agent any

    environment {
        // Force Ansible to use local config
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
        // Disable Host Key Checking (Safety net)
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        // Make output pretty (Enabled this for better Jenkins logs)
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
                withCredentials([file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE')]) {
                    sh '''
                        echo "Running Syntax Check..."
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_config.yml \
                        --syntax-check \
                        --vault-password-file $VAULT_PASS_FILE
                    '''
                }
            }
        }

        stage('Deploy Configuration') {
            steps {
                echo 'Deploying to Cisco Routers...'
                
                // FIX: Combined both File and Username/Password into ONE list
                withCredentials([
                    file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                    usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                ]) {
                    // Using shell environment variables (safer than Groovy interpolation)
                    sh '''
                        echo "Starting Playbook..."
                        
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_config.yml \
                        -e ansible_user="$NET_USER" \
                        -e ansible_password="$NET_PASS" \
                        --vault-password-file $VAULT_PASS_FILE \
                        -vvv
                    '''
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}

