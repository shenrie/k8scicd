# k8scicd


## Notes

docker build -t shenrie/go-app-example:1.0 .

docker run -d -p 8080:8080 --name go-app shenrie/go-app-example:1.0



kubectl create namespace develop

kubectl set image deployment hello-deployment go-app=sphenrie/k8scicd:11 -n develop