# k8scicd


## Notes

docker build -t shenrie/go-app-example:1.0 .

docker run -d -p 8080:8080 --name go-app shenrie/go-app-example:1.0



kubectl create namespace develop

kubectl set image deployment hello-deployment go-app=sphenrie/k8scicd:11 -n develop


Using docker for delarative pipeline
agent { 
    docker {
        image 'registry.az1:5043/maven-proto'
        registryUrl 'https://registry.az1'
        registryCredentialsId 'credentials-id'
        args '-v /var/jenkins_home/.m2:/root/.m2'
    }
}