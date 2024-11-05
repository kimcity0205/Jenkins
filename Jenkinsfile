pipeline {
  agent any
  stages {
    stage('git scm update') {
      steps {
        git url: 'https://github.com/kimcity0205/jenkinstest.git', branch: 'main'
      }
    }
    stage('delivery and deployment') {
      steps {
        sh '''
        ansible master -m copy -a "src=report_pod.yml dest=/root/jenkins/report_pod.yml" --become
        now=$(date +%y%m%d%H%M)
        sudo docker build -t kimcity0205/report:${now} .
        sudo docker push kimcity0205/report:${now}
        ansible node -m shell -a "sudo docker pull kimcity0205/report:${now}"
        ansible master -m shell -a "sudo kubectl create deploy web-${now} --replicas=3 --port=80 --image=kimcity0205/report:${now}"
        ansible master -m shell -a "sudo kubectl expose deploy web-${now} --type=LoadBalancer --port=80 --target-port=80 --name=web-${now}-svc"
        '''
      }
    }
  }
}