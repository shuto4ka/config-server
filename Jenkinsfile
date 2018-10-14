pipeline {
    agent any

    stages {
        stage('Setting env') {
            agent {
                docker {
                    image 'kozhenkov/gradle-agent:gradle-4.10.2-jdk-11'
                    reuseNode true
                    args '-u 1000:122 -v /var/run/docker.sock:/var/run/docker.sock:rw -v /usr/bin/docker:/usr/bin/docker:ro'
                }
            }
            steps {
                withEnv(['LC_ALL=ru_RU.UTF-8']) {
                    script {
                        env.PROJECT_VERSION =
                                sh script: 'gradle -P jenkins properties | grep ^version: | cut -d " " -f 2 | tr -d "\\n"',
                                        returnStdout: true
                        echo "PROJECT_VERSION is ${env.PROJECT_VERSION}"
                    }
                }
            }
        }

        stage('Build & Tests') {
            agent {
                docker {
                    image 'kozhenkov/gradle-agent:gradle-4.10.2-jdk-11'
                    reuseNode true
                    args '-u 1000:122 -v /var/run/docker.sock:/var/run/docker.sock:rw -v /usr/bin/docker:/usr/bin/docker:ro'
                }
            }
            steps {
                withEnv(['LC_ALL=ru_RU.UTF-8']) {
                    sh 'gradle -Dorg.gradle.daemon=false -P jenkins clean test build bootJar'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/build/test-results/**/*.xml'
                }
            }
        }

        stage('Build image') {
            steps {
                echo 'Copying artifact to docker location'
                fileOperations([fileCopyOperation(excludes: '', flattenFiles: true, includes: 'build/libs/*.jar', targetLocation: 'docker')])
                echo 'Building docker image'
                dir('docker') {
                    script {
                        def registryPath = "kozhenkov/config-server"

                        sh "docker build --build-arg app_vers=${env.PROJECT_VERSION} -t ${registryPath}:${BUILD_NUMBER} . "
                        sh "docker push ${registryPath}:${BUILD_NUMBER}"

                        echo 'Creating and pushing latest tag...'
                        sh "docker tag ${registryPath}:${BUILD_NUMBER} ${registryPath}:latest"
                        sh "docker push ${registryPath}:latest"

//                        if (env.IS_TAG_NEW == 'true') {
//                            echo "Creating and pushing ${env.PROJECT_VERSION} tag..."
//                            sh "docker tag ${registryPath}:${BUILD_NUMBER} ${registryPath}:${env.PROJECT_VERSION}"
//                            sh "docker push ${registryPath}:${env.PROJECT_VERSION}"
//                        } else {
//                            echo "Creating and pushing versioned tag skipped"
//                        }
                    }
                }
            }
        }

        stage('Downstream builds') {
            steps {
                build 'dev-config-server-deploy'
            }
        }
    }

//    post {
//        always {
//            sendNotifications(currentBuild.result)
//        }
//    }
}
