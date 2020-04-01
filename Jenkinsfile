pipeline {
    agent {
      kubernetes {
        label 'build'  // all your pods will be named with this prefix, followed by a unique id
        idleMinutes 1  // how long the pod will live after no jobs have run on it
        yamlFile 'build-pod.yaml'  // path to the pod definition relative to the root of our project 
        defaultContainer 'jnlp'
      }
    }
    environment {
        registry = "sphenrie/k8scicd"
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
                sh 'go build'               
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
                sh 'go test ./... -v -short'            
            }
        }
        stage('Publish') {
            agent { 
                docker { 
                    image 'docker' 
                }
            }
            environment {
                registryCredential = '62149d3c-dc3d-4b01-a23c-d0c1cf9d0502'
            }
            steps{
		echo 'PUSHING IMAGE IN CONATAINER'
                sh "/usr/local/bin/docker images sphenrie/k8scicd"
            }
        }
        stage ('Deploy') {
	    agent any
            steps {
                script{
                    def image_id = registry + ":$BUILD_NUMBER"
                    sh "kubectl set image deployment hello-deployment go-app=${image_id} -n develop --record"
                }
            }
        }
    }
}

