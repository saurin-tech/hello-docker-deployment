# About this project

This project chronicles my first attempt at running minikube with a simple [hello-docker](https://github.com/saurin-tech/hello-docker) container.

## My setup 

- Host OS: Ubuntu 18.04
- Docker: 18.06.0-ce
- Minikube: 0.28.2
- Kubectl: 
    - Client version: 1.11.2
    - Server version: 1.10.0

## Creating the container

There are several ways you can embark up on to create your first project using minikube.  You can choose from one of thousands of tutorials out on the internet.  Perhaps you happen to stumble upon this one. I hope you learn something from this little experiment of mine. If not you can always go back to professor google and ask him/her/it to help you solve your quest.  Before I start ranting I will start explaining.

- Before there was kubernetes there were containers or at least I hope there were. So I decided to build my own container using spring boot and gradle plugin called Palantir.  That project can be found [here](https://github.com/saurin-tech/hello-docker).  Download it and follow instructions and create a container.  But before I just tell you to download and and start executing those commands make sure you have all the necessary stuff like Java, docker, kubectl, and minikube installed.

- Make sure you run the `docker images` command and double check that you have built the image.

- Once the image is ready we will start the minikube and to do that you will execute `minikube start`.  If this is your first time and you have a slow internet connection this could take a little bit of time to get up and ready. So be patient my young Padawan.

## Deployment of container

- Once minikube has started go ahead and deploy the hello-docker-deployment by executing `kubectl apply -f hello-docker-deployment.yml`.  I know you are not gonna get any errors. Your terminal will tell you that everything is all good.  Life is peachy perfect!  Wrong!!!! Don't believe me yet.  Let's go ahead and check the pods and see if the project was deployed.  To do that go ahead and execute `kubectl get pods`. Oh, boy an error! READY shows 0/1.  STATUS tells me that `ErrImagePull`.  Image pull error?! What!  If you ran `docker images` command earlier then you are sure that you have the image on your host machine. What the heck is going on? Let me run another command `kubectl get pods --all-namespaces`.  This one is giving me a different STATUS `ImagePullBackOff`.  By this time things were getting annoying.

When Things start getting annoying consult professor Google.  After consulting with him a little bit I found this helpful little [guide](https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615).  But my setup was a bit different then the solutions provided in there.  Don't sweat it, I got you covered! After a bit more consulting with professor Google, I came across this [guy](https://stackoverflow.com/a/49151532) `docker save saurinp12/hello-docker:0.0.1-SNAPSHOT | pv | (eval $(minikube docker-env) && docker load)`.

If you didn't read the long article above, I got you!  Sparknotes version: basically your docker images repo is sitting on your host machine and you are running minikube inside a VM so therefor minikube can't find your docker images.  There are two ways of solving this:

- Push your images up to docker hub and pull them from there or you can type in the command above to push your images inside the minikube VM.  I choose the easy route and went with the command to save the images in the VM.

## Deleting the deployment

- Since the deployment wasn't working we have to delete it and redeploy it again.  

- To delete the deployment `kubectl delete deployment hello-docker-deployment`

- After deleting it deploy it again `kubectl apply -f hello-docker-deployment.yml`.

- Check again to see if it's deployed `kubectl get pods`

- Phew! It's working STATUS shows that the pod is Running.

## Creating Service

Without a Service you are not gonna be able to get to the running pod inside the minikube.  There are several ways of creating a service.  We will create a service via kubectl.

- To create service `kubectl expose deployment hello-docker-deployment --type=NodePort`

- To check if the service is running `kubectl get services`

- The above command will show you a two port numbers listed (8080:XXXXX/TCP) for our hello-docker-deployment under the ports column.  We are interested in the second port number.

- If you are following other tutorials on the internet you will come across another command which is `curl $(minikube service hello-docker-deployment --url)`.  Yeah, that one is not gonna work here buddy!

## Testing the deployment

- To test the deployment you are gonna need two things:

    - minikube ip - `minikube ip`
    - port number - `kubectl get services`  get the second port that I mentioned earlier.

- Now open up your favorite web browser your REST API testing tool like POSTMAN and generate url using minikube ip and the port number - `http://<minikubeIP>:<PortNumber>/hello`

- You will get `Hello Docker!` response back.

## Clean up

### Delete deployment

- `kubectl delete deployment hello-docker-deployment`

### Delete Service

- `kubectl delete service hello-docker-deployment`

### Stop minikube

- `minikube stop`

## Commands I ran trying to get this project running

Below is a list of commands I ran or materials I used to get this project up and running.

- To start minikube

    - `minikube start`

- To set docker environment

    - `eval $(minikube docker-env)`

- To deploy the pod/container

    - `kubectl apply -f hello-docker-deployment.yml`

- To check if the pod is deployed or not

    - `kubectl get pods`
    - `kubectl describe pod <pod-name>`
    - `kubectl get pods --all-namespaces`

- To describe the deployment

    - `kubectl get deployments`
    - `kubectl describe deployment hello-docker-deployment`

- To expose minikube deployment 

    - `kubectl expose deployment hello-docker-deployment --type=NodePort`

- To get services

    - `kubectl get services`

- To CURL minikube

    - `curl $(minikube service hello-docker-deployment --url)`

- To delete service

    - `kubectl delete service hello-docker-deployment`

- To access a service exposed via a node port, run this command in a shell after starting Minikube to get the address:

    - `minikube service [-n NAMESPACE] [--url] NAME`

- To delete deployment 

    - `kubectl delete deployment hello-docker-deployment`

## List of errors faced during deployment

- When running get pods command status comes up ImagePullBackOff.  To debug this run describe pod to look at the error.  The error I faced was that docker image was on the host machine rather then minikube.  Also there was no image on docker hub.  So therefore kubectl could not generate pod with a container. This is documented [here](https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615).

    - Another resolution for the above issue is [here](https://stackoverflow.com/questions/42564058/how-to-use-local-docker-images-with-minikube)
        - `docker save saurinp12/hello-docker:0.0.1-SNAPSHOT | pv | (eval $(minikube docker-env) && docker load)`

- Minikube username and password

    - username: docker
    - password: tcuser

- How to get IP of the minikube VM

    - `kubectl config view`
    - `minikube ip`