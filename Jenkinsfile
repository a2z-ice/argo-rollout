pipeline {
    stages {
        stage('Deploy Green') {
            steps {
                // Update image in Git or via kubectl to start rollout
                sh "kubectl patch rollout my-app-rollout --type merge -p '{\"spec\":{\"template\":{\"spec\":{\"containers\":[{\"name\":\"my-app\",\"image\":\"my-repo/my-app:${BUILD_NUMBER}\"}]}}}}'"
            }
        }
        stage('Test Green') {
            steps {
                script {
                    // Run tests against the 'Green' version using the header
                    def response = sh(script: "curl -H 'user-x-test: true' http://my-app.example.com/health", returnStatus: true)
                    if (response == 0) {
                        env.TEST_RESULT = "SUCCESS"
                    } else {
                        env.TEST_RESULT = "FAILURE"
                    }
                }
            }
        }
        stage('Promote or Rollback') {
            steps {
                script {
                    if (env.TEST_RESULT == "SUCCESS") {
                        echo "Tests passed. Promoting Green to Stable..."
                        sh "kubectl argo rollouts promote my-app-rollout"
                    } else {
                        echo "Tests failed. Rolling back..."
                        sh "kubectl argo rollouts abort my-app-rollout"
                    }
                }
            }
        }
    }
}
