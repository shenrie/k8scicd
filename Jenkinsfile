def POD_LABEL = "testpod-${UUID.randomUUID().toString()}"
podTemplate(label: POD_LABEL, cloud: 'kubernetes', containers: [
        containerTemplate(name: 'build', image: 'jfeng45/k8sdemo-backend:1.0', ttyEnabled: true, command: 'cat')
    ],
    volumes: [
        hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
    ]) {

    node(POD_LABEL) {
      environment {
        registry = "sphenrie/k8scicd"
        GOCACHE = "/tmp"
      }
        stage('Build') {
          container('build') {
            stage('Build a go project') {
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
        stage('Test') {
          container('build') {
            stage('Test a go project') {               
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
        stage('Publish') {
            environment {
                registryCredential = '62149d3c-dc3d-4b01-a23c-d0c1cf9d0502'
            }
            stage('Publish a go project') {  
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
            stage('Deploy a go project') {  
                script{
                    def image_id = registry + ":$BUILD_NUMBER"
                    sh "kubectl set image deployment hello-deployment go-app=${image_id} -n develop --record"
                }
            }
        }
    }
}

