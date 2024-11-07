pipeline {
    agent none  // 전체 파이프라인에서 사용할 에이전트는 지정하지 않음

    environment {
        GITHUB_REPO = 'https://github.com/your-username/your-repository.git'
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

        stage('Create Kubernetes Pod') {
            agent {
                kubernetes {
                    label 'jenkins-pod'  // Pod 레이블 설정
                    yaml """
                    apiVersion: v1
                    kind: Pod
                    metadata:
                      name: jenkins-pod
                    spec:
                      containers:
                        - name: jenkins-agent
                          image: 'jenkins/inbound-agent:latest'  // Jenkins Agent 이미지 사용
                          volumeMounts:
                            - name: playbook-volume
                              mountPath: /home/user1/jenkins/ansible  // playbook.yml 경로
                            - name: report-pod-volume
                              mountPath: /home/user1/jenkins/ansible/report_pod.yml  // report_pod.yml 경로
                              subPath: report_pod.yml
                      volumes:
                        - name: playbook-volume
                          hostPath:
                            path: ${WORKSPACE}  // Jenkins 워크스페이스
                            type: Directory
                        - name: report-pod-volume
                          hostPath:
                            path: ${REPORT_POD_SRC}  // report_pod.yml 경로
                            type: File
                    """
                }
            }
            steps {
                script {
                    echo "Kubernetes pod created with playbook and report_pod"
                }
            }
        }

        stage('Run Ansible Playbook') {
            agent {
                kubernetes {
                    label 'jenkins-pod'
                    yaml """
                    apiVersion: v1
                    kind: Pod
                    metadata:
                      name: jenkins-pod
                    spec:
                      containers:
                        - name: jenkins-agent
                          image: 'jenkins/inbound-agent:latest'  // Jenkins Agent 이미지
                          volumeMounts:
                            - name: playbook-volume
                              mountPath: /home/user1/jenkins/ansible
                            - name: report-pod-volume
                              mountPath: /home/user1/jenkins/ansible/report_pod.yml
                              subPath: report_pod.yml
                      volumes:
                        - name: playbook-volume
                          hostPath:
                            path: ${WORKSPACE}
                            type: Directory
                        - name: report-pod-volume
                          hostPath:
                            path: ${REPORT_POD_SRC}
                            type: File
                    """
                }
            }
            steps {
                script {
                    // Ansible Playbook을 Kubernetes Pod에서 실행
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
