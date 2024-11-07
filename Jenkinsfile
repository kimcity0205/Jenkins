pipeline {
    agent none  // 전체 파이프라인에서 사용할 에이전트는 지정하지 않음

    environment {
        GITHUB_REPO = 'https://github.com/kimcity0205/Jenkins.git'
        PLAYBOOK_PATH = '/home/user1/jenkins/ansible/playbook.yml'
        REPORT_POD_PATH = '/home/user1/jenkins/ansible/report_pod.yml'
        PLAYBOOK_SRC = "${WORKSPACE}/ansible/playbook.yml"  // Jenkins Workspace 내에서 Playbook 경로 설정
        REPORT_POD_SRC = "${WORKSPACE}/report_pod.yml"  // Jenkins Workspace 내에서 report_pod 경로 설정
    }

    stages {
        stage('Clone GitHub Repo') {
            agent any  // Git 클론은 어떤 에이전트에서든 가능
            steps {
                script {
                    // GitHub에서 리포지토리 클론
                    git url: "${GITHUB_REPO}", branch: 'main'
                }
            }
        }

        stage('Deploy Report Pod') {
            agent any  // 이 스테이지는 어떤 에이전트에서든 실행 가능
            steps {
                script {
                    // report_pod.yml을 Kubernetes 클러스터에 배포
                    sh """
                    kubectl apply -f ${REPORT_POD_SRC}
                    """
                }
            }
        }

        stage('Run Ansible Playbook') {
            agent any  // 이 스테이지는 어떤 에이전트에서든 실행 가능
            steps {
                script {
                    // Ansible Playbook을 실행합니다.
                    echo "Running Ansible Playbook"
                    sh """
                        ansible-playbook -i /home/user1/my_hosts ${PLAYBOOK_SRC} \
                        -e playbook_src=${PLAYBOOK_SRC} \
                        -e report_pod=${REPORT_POD_SRC}
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // 빌드 후 워크스페이스 정리
        }
    }
}
