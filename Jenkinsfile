pipeline {
  agent any
  stages {
    stage('git scm update') {
      steps {
        git url: 'https://github.com/kimcity0205/Jenkins.git', branch: 'main'
      }
    }

    stage('docker build and push') {
      steps {
        sh '''
        sudo docker build -t kimcity0205/report:image1 .
        sudo docker push kimcity0205/report:image1
        '''
      }
    }
    stage('deploy and service') {
      steps {
        sh '''
        sudo kubectl apply -f report_pod.yml
        '''
      }
    }
  }
}
