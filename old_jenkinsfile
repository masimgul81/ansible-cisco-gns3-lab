pipeline {
    agent any

    environment {
        // Disable Host Key Checking for Lab environment
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // Jenkins automatically pulls your code from GitHub here
                checkout scm
            }
        }

        stage('Syntax Check') {
            steps {
                sh 'ansible-playbook -i inventory/hosts.yml playbooks/router_config.yml --syntax-check'
            }
        }

        stage('Deploy Configuration') {
            steps {
                echo 'Deploying to Cisco Routers...'
                // Run the playbook
                sh 'ansible-playbook -i inventory/hosts.yml playbooks/router_config.yml'
            }
        }
    }
}
