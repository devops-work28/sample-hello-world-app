pipeline {
    agent any
    environment {
        registry = "magalixcorp/k8scicd"
        GOCACHE = "/tmp"
    }
        stages {
            stage('Build') {
                agent { 
                    docker { 
                        image 'golang' 
                    }
                }
                steps {
                    // Create our project directory.
                    sh 'cd ${GOPATH}/src'
                    sh 'mkdir -p ${GOPATH}/src/hello-world'
                    // Copy all files in our Jenkins workspace to our project directory.                
                    sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
                    // Build the app.
                    sh 'go build main.go'               
                }     
            }
            stage('Test') {
                agent { 
                    docker { 
                        image 'golang' 
                    }
                }
                steps {                 
                    // Create our project directory.
                    sh 'cd ${GOPATH}/src'
                    sh 'mkdir -p ${GOPATH}/src/hello-world'
                    // Copy all files in our Jenkins workspace to our project directory.                
                    sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
                    // Remove cached test results.
                    sh 'go clean -cache'
                    // Run Unit Tests.            
                }
            }
        
            stages {
                parallel {
                    stage('SonarQube: Code Quality') {
                        steps {
                            echo "Running SQ for code quality"
                        }
                    }
                    stage('Checkmarx: Application Security Report') {
                        steps {
                            echo "Running Checkmarx for application security"
                        }
                    }
                    stage('Blackduck: Scan') {
                        steps {
                            echo "Scan applications and container images"
                        }
                    }
                }
            }
            stage('Publish') {
                environment {
                    registryCredential = 'dockerhub'
                }
                steps{
                    script {
                        def appimage = docker.build registry + ":$BUILD_NUMBER"
                        docker.withRegistry( '', registryCredential ) {
                            appimage.push()
                            appimage.push('latest')
                        }
                    }
                }
            }
            stage ('Deploy') {
                steps {
                    script{
                        def image_id = registry + ":$BUILD_NUMBER"
                        sh "ansible-playbook  playbook.yml --extra-vars \"image_id=${image_id}\""
                    }
                }
            }
        }
}