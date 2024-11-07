pipeline {
    agent any

    environment {
        GITHUB_REPO = 'https://github.com/kimcity0205/Jenkins.git'
        PLAYBOOK_PATH = '/home/user1/jenkins/ansible/playbook.yml'
        REPORT_POD_PATH = '/home/user1/jenkins/ansible/report_pod.yml'
        PLAYBOOK_SRC = "${WORKSPACE}/ansible/playbook.yml"  // Jenkins Workspace 내에서 Playbook 경로 설정
        REPORT_POD_SRC = "${WORKSPACE}/report_pod.yml"  // Jenkins Workspace 내에서 report_pod 경로 설정
        KUBECTL_PATH = '/home/user1/bin/kubectl'  // 권한이 있는 디렉토리 사용
    }

    stages {
        stage('Clone GitHub Repo') {
            agent any
            steps {
                script {
                    git url: "${GITHUB_REPO}", branch: 'main'
                }
            }
        }

        stage('Download and Install kubectl') {
            steps {
                script {
                    // kubectl 다운로드 및 실행 파일 권한 부여
                    sh 'curl -LO https://dl.k8s.io/release/v1.24.0/bin/linux/amd64/kubectl'
                    sh 'chmod +x kubectl'
                    // 권한이 있는 디렉토리로 이동
                    sh "mv kubectl ${KUBECTL_PATH}"
                    // kubectl 위치를 PATH에 추가
                    sh "echo 'export PATH=${KUBECTL_PATH}:\$PATH' >> ~/.bashrc"
                    sh "source ~/.bashrc"
                }
            }
        }

        stage('Deploy Report Pod') {
            agent any
            steps {
                script {
                    // kubectl 버전 확인 후 실행
                    sh "${KUBECTL_PATH} version --client"
                    sh "kubectl apply -f ${REPORT_POD_SRC}"
                }
            }
        }

        stage('Run Ansible Playbook') {
            agent any
            steps {
                script {
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
            cleanWs()
        }
    }
}
