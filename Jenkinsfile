pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh 'chmod +x gradlew'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/reactApp'
            }
        }
        stage('Build Docker Image') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                script {
                    app = docker.build("quick5ilv3r/react-app")
                    app.inside {
                        sh 'echo $(curl localhost:1233)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'droopa@hotmail.com') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('Deploy To Staging') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                script {
                    sh "docker pull quick5ilv3r/react-app:${env.BUILD_NUMBER}"
                    try {
                        sh "docker stop react-app"
                        sh "docker rm react-app"
                    } catch (err) {
                        echo 'caught error: $err'
                    }
                    sh "docker run --restart always --name react-app -p 1233:80 -d quick5ilv3r/react-app:${env.BUILD_NUMBER}"
                }
            }
        }
        stage('Check HTTP Response') {
            steps {
                script {
                    final String url = "http://localhost:1233"
                    final String response = sh(script: "curl -o /dev/null -s -w '%{http_code}\\n' $url", returnStdout: true).trim()
                    if (response == "200") {
                        echo response
                        println "Successful Response Code" 
                    } else {
                        echo response
                        println "Error Response Code" 
                    }
                }
            }
        }
        stage('Deploy To Production') {
            when {
                expression { env.BRANCH_NAME == 'main' }
            }
            steps {
                input 'Does the staging environment look OK? Did you get a 200 response?'
                milestone(1)
                script {
                    sh "docker pull quick5ilv3r/react-app:${env.BUILD_NUMBER}"
                    try {
                        sh "docker stop react-app"
                        sh "docker rm react-app"
                    } catch (err) {
                        echo 'caught error: $err'
                    }
                    sh "docker run --restart always --name react-app -p 1233:80 -d quick5ilv3r/react-app:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}
