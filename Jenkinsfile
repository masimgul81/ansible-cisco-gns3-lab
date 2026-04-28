pipeline {
    agent any

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        // Initialize variable for the dynamic backup filename
        BACKUP_NAME = ""
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Cisco Operations') {
            when {
                anyOf {
                    // Triggers if any router playbook is modified
                    changeset "**/playbooks/router_.*"
                    // Triggers if the main Cisco inventory is modified
                    changeset "**/inventory/hosts.yml"
                    // Triggers if this Jenkinsfile itself is modified
                    changeset "**/Jenkinsfile"
                }
            }
            stages {
                stage('Backup Current State') {
                    steps {
                        script {
                            // 1. Generate a filename with a timestamp
                            def timeStamp = sh(returnStdout: true, script: 'date +%Y%m%d%H%M%S').trim()
                            env.BACKUP_NAME = "rollback_point_${timeStamp}.cfg"
                        }

                        echo "Creating rollback point: ${env.BACKUP_NAME}"

                        withCredentials([
                            file(credentialsId: 'ansible-vault-pass', variable: 'VAULT_PASS_FILE'),
                            usernamePassword(credentialsId: 'cisco-ssh', usernameVariable: 'NET_USER', passwordVariable: 'NET_PASS')
                        ]) {
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
        }
    }

    post {
        always {
            // Archives any .cfg files generated in the backups folder
            archiveArtifacts artifacts: 'backups/*.cfg', allowEmptyArchive: true
            cleanWs()
        }
    }
}
