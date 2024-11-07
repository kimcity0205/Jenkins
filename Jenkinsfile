pipeline {
    agent any  // 전체 파이프라인에서 사용할 에이전트는 지정하지 않음

    environment {
        GITHUB_REPO = 'https://github.com/kimcity0205/Jenkins.git'
        PLAYBOOK_PATH = '/home/user1/jenkins/ansible/playbook.yml'
        REPORT_POD_PATH = '/home/user1/jenkins/ansible/report_pod.yml'
        PLAYBOOK_SRC = "${WORKSPACE}/ansible/playbook.yml"  // Jenkins Workspace 내에서 Playbook 경로 설정
        REPORT_POD_SRC = "${WORKSPACE}/report_pod.yml"  // Jenkins Workspace 내에서 report_pod 경로 설정
    }

    stages {
        stage('Clone GitHub Repo') {
            agent any
            steps {
                script {
                    // GitHub에서 리포지토리 클론
                    git url: "${GITHUB_REPO}", branch: 'main'
                }
            }
        }

        stage('Install kubectl and Ansible') {
            agent any
            steps {
                script {
                    // kubectl 설치
                    sh '''
                    if ! command -v kubectl &> /dev/null
                    then
                        echo "kubectl not found, installing..."
                        curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl
                        chmod +x kubectl
                        mv kubectl /usr/local/bin/
                    else
                        echo "kubectl already installed"
                    fi
                    '''

                    // Ansible 설치
                    sh '''
                    if ! command -v ansible &> /dev/null
                    then
                        echo "Ansible not found, installing..."
                        sudo apt update
                        sudo apt install -y ansible
                    else
                        echo "Ansible already installed"
                    fi
                    '''
                }
            }
        }

        stage('Deploy Report Pod') {
            agent any
            steps {
                script {
                    // kubectl이 올바르게 설치되어 있는지 확인
                    sh 'kubectl version --client'  // kubectl 버전 확인
                    // report_pod.yml을 Kubernetes 클러스터에 배포
                    sh """
                    kubectl apply -f ${REPORT_POD_SRC}
                    """
                }
            }
        }

        stage('Run Ansible Playbook') {
            agent any
            steps {
                script {
                    // Ansible이 설치되어 있는지 확인
                    sh 'ansible --version'  // Ansible 버전 확인
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
