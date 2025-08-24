// The Final, Definitive Jenkinsfile for your Git Repo

pipeline {
    agent any

    environment {
        IMAGE_NAME = "whoami-app:${env.BUILD_NUMBER}"
        HELM_RELEASE_NAME = "my-whoami-release"
        CHART_PATH = "."
        K3D_CLUSTER_NAME = "my-cluster"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${IMAGE_NAME}"
                    docker.build(IMAGE_NAME, "-f Dockerfile.whoami .")
                }
            }
        }

        stage('Import Image into k3d') {
            steps {
                echo "Importing ${IMAGE_NAME} into k3d cluster '${K3D_CLUSTER_NAME}'..."
                sh "k3d image import ${IMAGE_NAME} --cluster ${K3D_CLUSTER_NAME}"
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    echo "Deploying application using a temporary, corrected kubeconfig..."
                    def tempKubeconfig = "/var/jenkins_home/workspace/Deploy-Whoami-App/kubeconfig_job_${env.BUILD_NUMBER}"

                    withEnv(["KUBECONFIG=${tempKubeconfig}"]) {
                        sh "cp /var/jenkins_home/.kube/config ${tempKubeconfig}"
                        sh "chmod 600 ${tempKubeconfig}"
                        sh "sed -i 's/0.0.0.0/k3d-my-cluster-server-0/g' ${tempKubeconfig}"

                        sh """
                        helm upgrade --install ${HELM_RELEASE_NAME} ${CHART_PATH} \
                            --set image.tag=${env.BUILD_NUMBER} \
                            --set image.repository=whoami-app
                        """
                    }
                }
            }
        }
    }
}
