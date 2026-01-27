pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        // Define a variable to hold our dynamic backup filename
        BACKUP_FILE_PATH = "" 
    }

    stages {
        stage('Checkout SCM') {
            steps { checkout scm }
        }

        stage('Backup Current State') {
            steps {
                script {
                    // 1. Generate a filename with a timestamp
                    def timeStamp = sh(returnStdout: true, script: 'date +%Y%m%d%H%M%S').trim()
                    // Set the global environment variable
                    env.BACKUP_NAME = "rollback_point_${timeStamp}.cfg"
                }

                echo "Creating rollback point: ${env.BACKUP_NAME}"

                withCredentials([
                    file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                    usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                ]) {
                    // Pass the specific filename to Ansible
                    sh '''
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_backup.yml \
                        -e ansible_user="$NET_USER" \
                        -e ansible_password="$NET_PASS" \
                        -e "custom_backup_filename=${BACKUP_NAME}" \
                        --vault-password-file $VAULT_PASS_FILE
                    '''
                }
            }
        }

        stage('Deploy Configuration') {
            steps {
                withCredentials([
                    file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                    usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                ]) {
                    sh '''
                        ansible-playbook -i inventory/hosts.yml \
                        playbooks/router_config.yml \
                        -e ansible_user="$NET_USER" \
                        -e ansible_password="$NET_PASS" \
                        --vault-password-file $VAULT_PASS_FILE
                    '''
                }
            }
            
            // --- ROLLBACK LOGIC ---
            post {
                failure {
                    echo "Deployment Failed! Initiating Rollback..."
                    
                    withCredentials([
                        file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                        usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                    ]) {
                        sh '''
                            echo "Rolling back to: backups/${BACKUP_NAME}"
                            
                            ansible-playbook -i inventory/hosts.yml \
                            playbooks/router_restore.yml \
                            -e ansible_user="$NET_USER" \
                            -e ansible_password="$NET_PASS" \
                            -e "restore_file=${WORKSPACE}/backups/${BACKUP_NAME}" \
                            --vault-password-file $VAULT_PASS_FILE
                        '''
                    }
                    echo "Rollback Complete. Please check device state."
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'backups/*.cfg', allowEmptyArchive: true
            cleanWs()
        }
    }
}
