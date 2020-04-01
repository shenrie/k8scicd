def POD_LABEL = "build-${UUID.randomUUID().toString()}"
podTemplate(label: POD_LABEL, cloud: 'kubernetes', containers: [
    containerTemplate(name: 'build', image: 'golang', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'test', image: 'golang', ttyEnabled: true, command: 'cat')
  ]) {

    node(POD_LABEL) {

        environment {
            registry = "sphenrie/k8scicd"
            GOCACHE = "/tmp"
        } 

        stage('Build') {
            container('build') {
              stage('build image') {
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
            }  
        }
        stage('Test') {
            container('test') {
              stage('test image') {
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
            }
        }
        stage('Publish') {
          container('build') {
            environment {
                registryCredential = '62149d3c-dc3d-4b01-a23c-d0c1cf9d0502'
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
        }
        stage ('Deploy') {
          container('build') {           
            steps {
                script{
                    def image_id = registry + ":$BUILD_NUMBER"
                    sh "kubectl set image deployment hello-deployment go-app=${image_id} -n develop --record"
                }
            }
          }
        }
    }
}

