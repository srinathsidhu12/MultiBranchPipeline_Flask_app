pipeline {
    agent any     //Run this pipeline on any available Jenkins agent/node
    environment {
        IMAGE="srinathsidhu12/multibranchpipeline_python_flask_app"
        TAG="${BUILD_NUMBER}"     //TAG will be the Jenkins build number (example: 1,2,3...)
    }
    stages {
        stage ('build') {
            steps {
                sh 'docker build -t "$IMAGE:$TAG" -t "$IMAGE:latest" ./python'     //Build a Docker image from Dockerfile in the current directory (.) and  Apply two tag
            }
        }
        stage ('push') {
            steps {
                //Use Jenkins stored credentials to login to Docker Hub
                //'my-dockerhub-creds' is the ID of credentials saved in Jenkins
                withCredentials([usernamePassword(credentialsId: 'my-dockerhub-creds', passwordVariable: 'DOCKERHUB_PWD', usernameVariable: 'DOCKERHUB_USER')]) {
                sh 'echo "$DOCKERHUB_PWD" | docker login -u "$DOCKERHUB_USER" --password-stdin'
                sh 'docker push "$IMAGE:$TAG"'
                sh 'docker push "$IMAGE:latest"'
                } 
                
            }
        }
stage('deploy') {
      steps {
        sh 'docker pull "$IMAGE:$TAG"'     //Pull the newly built version of the image
        sh 'docker rm -f multibranchpipeline_python_flask_app || true'     // Remove old container if exists
        sh 'docker run -d --name flask_app -p 5000:5000 "$IMAGE:$TAG"'    //Run the new container

        //Create a deployment info file,Helpful for debugging & tracking which version is deployed
        sh '''
          cat > deploy-info-$BUILD_NUMBER.txt <<EOF
build: $BUILD_NUMBER
image: $IMAGE:$TAG
commit: ${GIT_COMMIT}
branch: $GIT_BRANCH
time: $(date -u +"%Y-%m-%dT%H:%M:%SZ")
url: $BUILD_URL
EOF
        '''
        //Archive the deploy-info file in Jenkins
        //fingerprint:true -> Jenkins tracks this file uniquely
        archiveArtifacts artifacts: "deploy-info-${BUILD_NUMBER}.txt", fingerprint: true, followSymlinks: false
      }
    }
        stage ('test') {
            steps {
                // Simple check: wait for 2 seconds then print URL for testing
                sh 'sleep 2; echo "Hit http://localhost:5000 to see the app."'
            }
        }      
            stage ('cleanup') {
            // Remove workspace files to keep Jenkins clean
            steps {
                cleanWs()
            }
        }
    }
    // POST-BUILD NOTIFICATIONS
    // ------------------------------
    post {

        // If pipeline is successful
        success {
            echo "Build ${env.BUILD_NUMBER} succeeded"
        }

        // If pipeline fails
        failure {
            echo "Build ${env.BUILD_NUMBER} failed"
        }

        // Always printed whether success/failure
        always {
            echo "Build ${env.BUILD_NUMBER} finished"
        }
    }
}
