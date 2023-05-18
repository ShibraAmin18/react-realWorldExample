pipeline {
     agent {
    kubernetes {
        label 'jenkinsrun'
        defaultContainer 'dind'
        yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: dind
    image: eeacms/jenkins-slave-dind:latest
    securityContext:
      privileged: true
    volumeMounts:
      - name: dind-storage
        mountPath: /var/lib/docker
  volumes:
    - name: dind-storage
      emptyDir: {}
"""
        }
      }

    environment {
        AWS_ACCOUNT_ID = 271251951598
        AWS_REGION = "us-east-2"
        ECR_REPO_NAME = '271251951598.dkr.ecr.us-east-2.amazonaws.com/web/stg'
        ECR_URL = 'https://271251951598.dkr.ecr.us-east-1.amazonaws.com'
        GIT_CREDENTIALS_ID = 'jenkins-demo-shibra'
    }

    stages {
        stage('Unit Test') {
            when { changeRequest target: 'develop' }
            steps {
                sh 'npm test'
            }
        }

        stage('Docker Build and Push') {
            when { anyOf { branch 'develop'; branch 'master' } }
            steps {
                script {
                    docker.build("${ECR_REPO_NAME}:test")
                    docker.withRegistry(ECR_URL, "ecr:us-east-2:${env.CREDENTIALS_ID}") {
                        docker.push("${ECR_REPO_NAME}:test")
                    }
                }
            }
        }

        stage('Approval') {
            when { anyOf { branch 'develop'; branch 'master' } }
            steps {
                input 'Deploy to environment?'
            }
        }

        stage('Deploy') {
            when { anyOf { branch 'develop'; branch 'master' }}
            steps {
                script {
                    // Determine the environment based on the branch
                    def env = (env.BRANCH_NAME == 'develop') ? 'development' : 'production'

                    // Clone the Helm chart repo
                    git credentialsId: GIT_CREDENTIALS_ID, url: 'https://github.com/your-helm-chart-repo'

                    // Update the Docker image tag in the values.yaml file
                    sh """
                        sed -i 's|image: .*|image: ${ECR_URL}/${ECR_REPO_NAME}:${env.BUILD_NUMBER}|' ${env}/values.yaml
                    """

                    // Commit and push the changes
                    sh """
                        git add ${env}/values.yaml
                        git commit -m "Update Docker image tag for ${env} environment"
                        git push origin HEAD
                    """
                }
            }
        }
    }
}
