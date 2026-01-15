pipeline {
    agent any

    environment {
        // 1. FORCE ANSIBLE TO READ YOUR CONFIG
        // This ensures the "Magic Fix" SSH args are actually used
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible.cfg"
        
        // 2. Disable Host Key Checking (Safety net)
        ANSIBLE_HOST_KEY_CHECKING = 'False'
        
        // 3. Make output pretty in Jenkins Console
        ANSIBLE_FORCE_COLOR = 'true'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Environment Check') {
            steps {
                // Debugging: Verify we are the jenkins user and have the collection
                sh '''
                whoami
                pwd
                ls -la
                echo "Using Config: $ANSIBLE_CONFIG"
                ansible-galaxy collection list cisco.ios || echo "WARNING: Collection might be missing"
                '''
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
                    // We use the variables passed by Jenkins credential manager
                    sh '''
                    ansible-playbook \
                      -i inventory/hosts.yml \
                      playbooks/router_config.yml \
                      -e ansible_user=$ANSIBLE_USER \
                      -e ansible_password=$ANSIBLE_PASS \
                      -vvv
                    '''
                    // -vvv gives verbose output so you can see exactly why SSH fails if it does
                }
            }
        }
    }
}
