pipeline {
    agent any
    environment {
        IMAGE="cloudazi/maven-prj"
        TAG="${BUILD_NUMBER}"
    }
    stages{
        stage('checkout'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Iziik/python-jenkins.git']])
            }
        }
        stage('Build docker image'){
            steps{
                script{
                    sh 'docker build -t "$IMAGE:$TAG" -t "$IMAGE:latest" .'
                }
            }
        }
        stage('Push'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKERHUB_PWD', usernameVariable: 'DOCKERHUB_USER')]) {
                sh 'echo "Logging in to Docker Hub..."'
                sh 'echo "$DOCKERHUB_PWD" | docker login -u "$DOCKERHUB_USER" --password-stdin '
                sh 'docker push "$IMAGE:$TAG"'
                sh 'docker push "$IMAGE:latest"'
}
                
            }
        }
        stage('Deploy'){
            steps{
                sh 'docker rm -f iziik-maven || true'
                sh 'docker run -d --name iziik-maven -p 5001:5000 "$IMAGE:$TAG"'
                
                
                // write deploy info with build number in the filename
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

        // fingerprint: true tells Jenkins to compute and store a unique hash (a “fingerprint”) for the archived file.
        archiveArtifacts artifacts: "deploy-info-${BUILD_NUMBER}.txt", fingerprint: true, followSymlinks: false
    }
}
        stage ('test') {
            steps {
                sh 'sleep 2; echo "Hit http://localhost:5001 to see the app."'
            }
        }
        
        stage ('cleanup') {
            steps {
                cleanWs()
            }
        }
  }
  
  post {
    success {
        echo "Build ${env.BUILD_NUMBER} succeeded"
        }
    failure { echo "Build ${env.BUILD_NUMBER} failed" }
    always  { echo "Build ${env.BUILD_NUMBER} finished" }
  }
}
