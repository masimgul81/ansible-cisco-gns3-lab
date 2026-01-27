pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
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
                withCredentials([file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE')]) {
                    sh '''
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_config.yml \
                        --syntax-check \
                        --vault-password-file $VAULT_PASS_FILE
                    '''
                }
            }
        }

        // --- NEW STAGE STARTS HERE ---
        stage('Backup Current State') {
            steps {
                echo 'Taking backup of Cisco Routers before changes...'
                
                withCredentials([
                    file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                    usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                ]) {
                    sh '''
                        # Run the backup playbook
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_backup.yml \
                        -e ansible_user="$NET_USER" \
                        -e ansible_password="$NET_PASS" \
                        --vault-password-file $VAULT_PASS_FILE
                    '''
                }
            }
        }
        // --- NEW STAGE ENDS HERE ---

        stage('Deploy Configuration') {
            steps {
                echo 'Deploying to Cisco Routers...'
                withCredentials([
                    file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                    usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                ]) {
                    sh '''
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_config.yml \
                        -e ansible_user="$NET_USER" \
                        -e ansible_password="$NET_PASS" \
                        --vault-password-file $VAULT_PASS_FILE \
                        -v
                    '''
                }
            }
        }
    }

    post {
        always {
            // CRITICAL: Save the backup files to Jenkins UI before deleting the workspace
            archiveArtifacts artifacts: 'backups/*.cfg', allowEmptyArchive: true
            
            cleanWs()
        }
    }
}pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
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
                withCredentials([file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE')]) {
                    sh '''
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_config.yml \
                        --syntax-check \
                        --vault-password-file $VAULT_PASS_FILE
                    '''
                }
            }
        }

        // --- NEW STAGE STARTS HERE ---
        stage('Backup Current State') {
            steps {
                echo 'Taking backup of Cisco Routers before changes...'
                
                withCredentials([
                    file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                    usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                ]) {
                    sh '''
                        # Run the backup playbook
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_backup.yml \
                        -e ansible_user="$NET_USER" \
                        -e ansible_password="$NET_PASS" \
                        --vault-password-file $VAULT_PASS_FILE
                    '''
                }
            }
        }
        // --- NEW STAGE ENDS HERE ---

        stage('Deploy Configuration') {
            steps {
                echo 'Deploying to Cisco Routers...'
                withCredentials([
                    file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                    usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                ]) {
                    sh '''
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_config.yml \
                        -e ansible_user="$NET_USER" \
                        -e ansible_password="$NET_PASS" \
                        --vault-password-file $VAULT_PASS_FILE \
                        -v
                    '''
                }
            }
        }
    }

    post {
        always {
            // CRITICAL: Save the backup files to Jenkins UI before deleting the workspace
            archiveArtifacts artifacts: 'backups/*.cfg', allowEmptyArchive: true
            
            cleanWs()
        }
    }
}
