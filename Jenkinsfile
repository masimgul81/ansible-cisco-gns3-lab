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
		// Good practice: Check for YAML errors before running
                withCredentials([file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE')]) {
		   sh '''
		   ansible-playbook \
                     -i inventory/hosts.yml \
                     playbooks/router_config.yml \
                     --syntax-check --vault-password-file $VAULT_PASS_FILE
                   '''
		 }
            }
        }

        stage('Deploy Configuration') {
            steps {
                echo 'Deploying to Cisco Routers...'
                withCredentials([file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE')])
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
		      --vault-password-file $VAULT_PASS_FILE
                      -vvv
                    '''
                }
            }
        }
    }
}
