pipeline {
  agent any
  stages {
    stage('Clone Repo') {
      steps {
        git 'https://github.com/DaChe01/Task6.gitt'
      }
    }
    stage('Run Ansible') {
      steps {
        sh 'ansible-playbook deploy-httpd.yaml -i inventory.ini'
      }
    }
  }
}
