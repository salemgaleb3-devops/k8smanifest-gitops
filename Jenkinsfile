pipeline {
    agent any

    parameters {
        string(name: 'DOCKERTAG', description: 'Docker image tag (e.g. BUILD_NUMBER)')
    }

    environment {
        IMAGE_NAME = "salimgalib/myflaskapp"
        GIT_REPO   = "https://github.com/salemgaleb3-devops/k8smanifest-gitops.git"
        GIT_BRANCH = "main"
        GIT_CREDS  = "github-creds"
    }

    stages {

        stage('Validate Parameters') {
            steps {
                script {
                    if (!params.DOCKERTAG?.trim()) {
                        error "DOCKERTAG is required and cannot be empty ❌"
                    }
                }
            }
        }

        stage('Clone Manifest Repository') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
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
                        set -e
                        git config user.email "jenkins@local"
                        git config user.name "jenkins"

                        echo "Before update:"
                        grep image deployment.yaml

                        sed -i 's|\\(image: ${IMAGE_NAME}:\\).*|\\1${params.DOCKERTAG}|' deployment.yaml

                        echo "After update:"
                        grep image deployment.yaml

                        git add deployment.yaml
                        git commit -m "Update image tag to ${params.DOCKERTAG}"
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/salemgaleb3-devops/k8smanifest-gitops.git HEAD:${GIT_BRANCH}
                    """
                }
            }
        }
    }
}
