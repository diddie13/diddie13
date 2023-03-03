#### Preamble

Assume you’re running on your favorite cloud (Azure, AWS, GCP) - you don’t have to make this work specifically for GCP GKE.
Create a README.md that outlines your line of thinking for the solution.
Create plain Kubernetes resources (yaml or json). Please return this file in your response with any other materials you want to share with us.

##### Cluster criado:
"cluster-thinkon" 

![alt text](https://user-images.githubusercontent.com/68625361/222587868-0eb68673-e686-4976-979f-42cdf75d3cd8.png)

![alt text](https://user-images.githubusercontent.com/68625361/222567944-c5ea79fa-8a18-46f4-9ccf-b4910dfc7c0d.png
)

##### Arquivos yml criados:

- __pod_users.yml__ (File that creates a pod using the image "diddie/container-db_users")

```apiVersion: v1
kind: Pod
metadata:
  name: pod-db-users
  labels:
    app: pod-db-users
spec:
  containers:
    - name: container-db-users
      image: diddie/container-db_users:v1.03
      ports:
        - containerPort: 80
``` 
![alt text](https://user-images.githubusercontent.com/68625361/222583191-f03c3953-2d7e-4b99-916b-d5809989ae98.png)


- __pod_shifts.yml__ (File that creates a pod using the image "diddie/container-db-shifts")

```apiVersion: v1
kind: Pod
metadata:
  name: pod-db-shifts
  labels:
    app: pod-db-shifts
spec:
  containers:
    - name: container-db-shifts
      image: diddie/container-db-shifts:v1
      ports:
        - containerPort: 84
```
![alt text](https://user-images.githubusercontent.com/68625361/222583202-b39ec08e-0b00-4ad8-a756-f1a9879efcf1.png)


- __autoscaling_users.yml__ (Auto scale file for users container)

```apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: autoscaling-pod-db-users
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: pod-db-users
  minReplicas: 1
  maxReplicas: 2
  targetCPUUtilizationPercentage: 70
```
![alt text](https://user-images.githubusercontent.com/68625361/222585339-8c0e3c99-5dfd-44c9-9284-bed6472f9d2b.png)


- __autoscaling_shift.yml__ (Auto scale file for shifts container)

```apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: autoscaling-pod-db-shifts
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: pod-db-shifts
  minReplicas: 1
  maxReplicas: 2
  targetCPUUtilizationPercentage: 70
```

![alt text](https://user-images.githubusercontent.com/68625361/222585353-e4c51d4a-0530-40e1-a6ac-d7d04b29f7fe.png)

- __lb_users.yml__ (Load balancer file for users container)

```apiVersion: v1
kind: Service
metadata:
  name: lb-db-users
spec:
  type: LoadBalancer
  ports:
    - port: 80
      nodePort: 30000
  selector:
    app: pod-db-users
```

![alt text](https://user-images.githubusercontent.com/68625361/222586648-1d547483-83a3-48b0-a97c-2aa3f498c4aa.png)

- __lb_shifts.yml__ (Load balancer file for shifts container)

```apiVersion: v1
kind: Service
metadata:
  name: lb-db-shifts
spec:
  type: LoadBalancer
  ports:
    - port: 84
      nodePort: 30004
  selector:
    app: pod-db-shifts
```
![alt text](https://user-images.githubusercontent.com/68625361/222586672-a5e69d9e-2b8d-4e6e-8611-cc406a0ceca5.png)


You can make the following assumptions:

Each system you’re deploying has its own isolated database. You don’t have to worry about the type. You can assume the database is in the same region as your k8s.
    You can use any docker image you’d like for your containers. It’s just an example and does not have to work. Any, say, default php docker you can deploy on a pod. What the container it is, does not matter - but we’ll be talking about two different containers in the exercise, one for users, and one for shifts.
    Assume daily bell-curve scaling. High traffic during the day, low traffic during the night

Foram criadas as seguintes imagens:
###### Container for users: 
- diddie/container-db_users 
     (image with mysql and it was uploaded to the Docker hub)

![alt text](https://user-images.githubusercontent.com/68625361/222567984-39bb8ba0-0842-4b31-a509-5c30e4cbd972.png)

###### Container for shifts:
- diddie/container-db-shifts 
    (image with mysql and it was uploaded to the Docker hub)

![alt text](https://user-images.githubusercontent.com/68625361/222567973-508169c0-7c33-40fa-b539-09a90a8ef360.png)

Exercise
1.) We want to deploy two containers that scale independently from one another.
- Container 1: This container runs code that runs a small API that returns users from a database.

    - The image "diddie/container-db_users" was created with has mysql already installed with data inserted from the file "db_users.sql". The .sql file was used in the Dockerfile. 
    - The database could have been created directly on the cloud provider.

- Container 2: This container runs code that runs a small API that returns shifts from a database.

    - The image "diddie/container-db-shifts" was created with has mysql already installed with data inserted from the file "db_shifts.sql". The .sql file was used in the Dockerfile.
    - The database could have been created directly on the cloud provider.
    
2.) For the best user experience auto scale this service when the average CPU reaches 70%.
##### NOTE 1: As the type of scaling was not mentioned, was created an hpa for both.
##### NOTE 2:For both, cronjob can also be used to help scale high traffic during the day and low traffic during the night.

- The "autoscaling_users.yml" file creates the auto scale service for the users container.
  ![alt text](https://user-images.githubusercontent.com/68625361/222585339-8c0e3c99-5dfd-44c9-9284-bed6472f9d2b.png)
  
- The "autoscaling_shift.yml" file creates the auto scale service for the shifts container.
![alt text](https://user-images.githubusercontent.com/68625361/222585353-e4c51d4a-0530-40e1-a6ac-d7d04b29f7fe.png)  


NOTE 3: A load balance service was also created for both.
- __lb_users.yml__ (Load balancer file for users container)
![alt text](https://user-images.githubusercontent.com/68625361/222586648-1d547483-83a3-48b0-a97c-2aa3f498c4aa.png)
![alt text](https://user-images.githubusercontent.com/68625361/222605210-ed3e3a80-e5b1-459f-9fbf-358121b392c2.png)

- __lb_shifts.yml__ (Load balancer file for shifts container)
![alt text](https://user-images.githubusercontent.com/68625361/222586672-a5e69d9e-2b8d-4e6e-8611-cc406a0ceca5.png)
![alt text](https://user-images.githubusercontent.com/68625361/222605221-78f7d515-3714-4377-a9c9-8fddf68ae4d1.png)

3.) Ensure the deployment can handle rolling deployments and rollbacks.
- Use the "replicaSet"

4.) Your development team should not be able to run certain commands on your k8s cluster, but you want them to be able to deploy and roll back. What types of IAM controls do you put in place?
- Creating a new custom group and policy with the necessary settings for deployment and rollback and some "extra features", deploy and rollback is too vague and leaves no room for post-deployment analysis, among other types of features such as logging.
Example are the permissions of the image below, creation and rollback.

![alt text](https://user-images.githubusercontent.com/68625361/222567994-43e0d8e4-7566-4aab-a99e-8d316e8e082e.png)

Bonus
·    How would you apply the configs to multiple environments (staging vs production)?
- In my opinion, the best way would be to create configmaps by environments, like: dev, dev2, qa... And for each configmap to have a namespace by environment, like: dev, dev, qa... And if possible, create pipelines for perform the deployment in its respective environment. Ideally, each file/environment should have its own configurations so as not to depend on another resource/configuration, making it easier to manage the independent configurations.

·How would you auto-scale the deployment based on network latency instead of CPU?
- Creating an entry controller (ingress) that manages access to services in the cluster by HTTP/HTTPS
