pipeline {
    agent any

    parameters {
        string(name: 'DOCKERTAG', description: 'Docker image tag (e.g. BUILD_NUMBER)')
    }

    environment {
        IMAGE_NAME = "salimgalib/myflaskapp"
        GIT_REPO   = "https://github.com/salimgalib/kubernetesmanifest.git"
        GIT_BRANCH = "main"
        GIT_CREDS  = "github"
    }

    stages {

        stage('Clone Manifest Repository') {
            steps {
                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO}"
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${GIT_CREDS}",
                        usernameVariable: 'GIT_USERNAME',
                        passwordVariable: 'GIT_PASSWORD'
                    )
                ]) {
                    sh """
                      git config user.email "jenkins@local"
                      git config user.name "jenkins"

                      echo "Before update:"
                      cat deployment.yaml

                      sed -i 's|image: ${IMAGE_NAME}:.*|image: ${IMAGE_NAME}:${DOCKERTAG}|' deployment.yaml

                      echo "After update:"
                      cat deployment.yaml

                      git add deployment.yaml
                      git commit -m "Update image tag to ${DOCKERTAG}" || echo "No changes to commit"
                      git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/salimgalib/kubernetesmanifest.git HEAD:${GIT_BRANCH}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Manifest updated successfully 🎉"
        }
        failure {
            echo "Failed to update manifest ❌"
        }
    }
}

