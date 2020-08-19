# Running Spark job on local kubernetes cluster (Minikube)
#### Contains steps to perform spark jobs in Minikube cluster 

###### 1. Install docker -> for Win **[Docker-desktop](https://www.docker.com/products/docker-desktop)**  and check: 
```shell script
 docker version
```
###### 2. Create folder C:/minikube_home

###### 3. Create folder C:/kube

###### 4. Set env variable MINIKUBE_HOME=C:/minikube_home and add C:/kube to Path variable

###### 5. Install kubectl -> **[Kubectl-for-win](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)**  download, add to C:/kube and check:
 ```shell script
 kubectl version --client
 ```
###### 6. Install minikube -> **[Minikube-for-win](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-minikube-via-direct-download)** download, add to C:/kube and check:
 ```shell script
 minikube version
 ``` 
###### 7. Run minikube cluster, for windows:
  * 1 Create EXTERNAL switch in Hiper-V manager 'MinikubeSwitch' (set Ethernet adapter)
  * 2 Run command:	
 ```shell script
     minikube start --vm-driver=hyperv --memory=6144 --cpus=3 --hyperv-virtual-switch='MinikubeSwitch' --kubernetes-version='v1.15.1'
``` 
  * 3 Check cluster:
   ```sh
      kubectl get nodes
   ``` 
![minikube_get_nodes](https://user-images.githubusercontent.com/42671888/90618315-7c41c900-e218-11ea-81ee-062941fdd52e.PNG)

##### ATTENTION! Use compatible versions spark and k8s  ->  in my case spark 2.4.6 (jdk 8 252) work without issues with k8s v.1.16 and lower. For example, for k8s v.1.18.3 with spark 2.4.6, you get issue:  
```shell script
ERROR SparkContext: Error initializing SparkContext.
  org.apache.spark.SparkException: External scheduler cannot be instantiated
```


##### ATTENTION!! INSTEAD of hyperv driver on Windows we can run Minikube with 'docker' driver, it's more simply -> don't create an external switch, just run:
 ```sh
 minikube start --vm-driver=docker --memory=6144 --cpus=3 --kubernetes-version='v1.15.1'
``` 
###### 8. 8-10 steps are optional ->  to perform use RAM memory dynamically by Minikube cluster, Run:
  ```sh
  minikube stop
 ``` 
###### 9. Open Hyper-V manager -> open Minikube Virtual machine settings -> choose checkbox 'Enable dynamic memory' in the memory section and set range

###### 10. Run :
  ```sh
  minikube start
 ``` 
###### 11. Install Spark locally -> download and extract archive to C:/spark -> add to Path and create HADOOP_HOME and SPARK_HOME env variables (naturally install Java)

###### 12. Create docker image for k8s, run:
  ```sh
  cd /c/spark
  eval $(minikube docker-env)  // set docker env values for Minikube's docker daemon
  ./bin/docker-image-tool.sh -m -t spark-docker build   // -m Use Minikube's Docker daemon, -t set tag 'spark-docker'
 ``` 
 Using minikube when building images will do so directly into minikube's Docker daemon.
 There is no need to push the images into minikube in that case, they'll be automatically
 available when running applications inside the minikube cluster.
###### ATTENTION! For this step on Windows - USE BASH SHELL AND RUN as ADMINISTRATOR (need to perform docker env settings) 
 
###### 13. Check spark examples jar version cd /c/spark/examples/jars
In my case, for example: spark-examples_2.11-2.4.6.jar
Take this correct version to run spark job later

###### 14. Get correct $KUBERNETES_MASTER IP address (192.168.100.12:8443 for example):
  ```sh
  kubectl cluster-info
 ``` 
![minikube_cluster_info](https://user-images.githubusercontent.com/42671888/90618554-d8a4e880-e218-11ea-9f7f-8b9402164ab0.PNG)
###### 15. Get list of Minikube addons
 ```sh
  minikube addons list
 ``` 
###### 16. Enable Dashboard:
 ```sh
  minikube addons enable dashboard
  minikube addons enable metrics-server // in some cases required for dashboard
 ``` 
###### 17. Open Minikube dashboard:
 ```sh
  minikube dashboard
 ``` 
###### 18. Create 'spark' service account and role with 'edit' permissions (to perform spark job in a pod using RBAC -> read more K8S RBAC)
 ```sh
  kubectl create serviceaccount spark 
  kubectl create clusterrolebinding spark-role --clusterrole=edit --serviceaccount=default:spark --namespace=default 
 ``` 
###### 19. Submit spark job on Minikube:
 ```sh
  spark-submit --master k8s://https://$KUBERNETES_MASTER --deploy-mode cluster --name spark-pi-OR-YOU-CUSTOM-NAME --class org.apache.spark.examples.SparkPi --conf spark.executor.instances=2 --conf spark.kubernetes.container.image=spark:spark-docker --conf spark.kubernetes.authenticate.driver.serviceAccountName=spark  local:///opt/spark/examples/jars/spark-examples_2.11-2.4.6.jar
 ``` 
###### ATTENTION! Check your example jar version and set $KUBERNETES_MASTER for example: 192.168.100.12:8443

###### 20. Open Dashboard verify driver pod (read logs) or run 
 ```sh
  kubectl get pods
  kubectl logs 'POD_NAME' // for example: kubectl logs spark-pi-test-1597770716703-driver
 ``` 

###### 21. Also, you may watch the execution, run after the spark-submit command in another window:
```sh
  kubectl get pods --watch
 ``` 
###### You will see something output like this:

![minikube_spark_execution](https://user-images.githubusercontent.com/42671888/90609424-abead400-e20c-11ea-9ae9-87422172606a.PNG)
