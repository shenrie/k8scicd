

def POD_LABEL = "buildpod-${UUID.randomUUID().toString()}"
podTemplate(label: POD_LABEL, cloud: 'kubernetes', 
containers: [
    containerTemplate(name: 'build', image: 'golang', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'test', image: 'golang', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
volumes: [
    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
  ]) {

    node(POD_LABEL) {
        environment {
            registry = "sphenrie/k8scicd"
            GOPATH = "/tmp"
        }
        stage('build a go project') {
            container('build') {
                stage('Build') {

                    checkout scm

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

        stage('Test a Golang project') {
            container('test') {
                stage('Test') {
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

        stage('Publish a Golang project') {
            environment {
                registryCredential = '62149d3c-dc3d-4b01-a23c-d0c1cf9d0502'
            }
            container('docker') {
                stage('Publish') {

                    def appimage = docker.build registry + ":$BUILD_NUMBER"
                    docker.withRegistry( '', registryCredential ) {
                        appimage.push()
                        appimage.push('latest')
                    }
                }
            }
        }

        stage('Deploy a Golang project') {
            container('docker') {
                stage('Deploy') {

                    def image_id = registry + ":$BUILD_NUMBER"
                    sh "kubectl set image deployment hello-deployment go-app=${image_id} -n develop --record"
                }
            }
        }
    }
}


