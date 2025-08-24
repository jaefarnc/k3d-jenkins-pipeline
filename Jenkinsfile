// my-first-chart/Jenkinsfile

pipeline {
	agent any

	environment {
		//Define a unique name using the job name and build number
		IMAGE_NAME = "whoami-app:${env.BUILD_NUMBER}"
		// The name of our Helm release
		HELM_RELEASE_NAME = "my-whoami-release"
		// The path to our chart
		CHART_PATH = "."
	}

	stages {
		stage('Build Docker Image') {
			steps {
				script {
					echo "Building Docker image ${IMAGE_NAME}"
					docker.build(IMAGE_NAME, "-f Dockerfile.whoami .")
				}
			}
		}

		stage('Import Image into k3d') {
			steps {
				echo "Importing ${IMAGE_NAME} into k3d cluster..."
				sh "k3d image import ${IMAGE_NAME}"
			}
		}

		stage('Deploy with Helm') {
			steps {
				echo "Deploying the application with Helm..."
				// Run the helm upgrade --install command
				// This will install the release if it doesn't exist, or upgrade it if it does.
				// We override teh image tag to use the one we just built.
				sh """
				helm upgrade --install ${HELM_RELEASE_NAME} ${CHART_PATH} \
					--set image.tag=${env.BUILD_NUMBER} \
					--set image.repository=whoami-app
				"""
			}
		}
	}

	post {
		always {
			echo 'Pipeline finished.'
		}
	}
}
