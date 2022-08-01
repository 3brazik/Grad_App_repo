# Graduation Project

![Untitled](Graduation%20Project%20d2f3965c56884900ba9fdfc0b26b4a47/Untitled.png)

# Docker file to build our app image.

```docker
FROM node:12
COPY nodeapp /nodeapp
WORKDIR /nodeapp
RUN npm install
CMD ["node", "/nodeapp/app.js"]
```

## Then we create our app deployment and load-balancer yaml file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-deploy
  namespace: application
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs
  template:
    metadata:
      labels:
        app: nodejs
    spec:
      containers:
        - name: node-container
          image: 3brazik/app_img
          ports:
            - name: http
              containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: app-load-balancer
  namespace: application
spec:
  type: LoadBalancer
  selector:
    app: nodejs
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```

# Then, configure the bastion instance as a slave

go to manage jenkins > Manage nodes and cloud > New Node

![Untitled](Graduation%20Project%20d2f3965c56884900ba9fdfc0b26b4a47/Untitled%201.png)

![Untitled](Graduation%20Project%20d2f3965c56884900ba9fdfc0b26b4a47/Untitled%202.png)

## Then, create jenkins pipeline script

```groovy
pipeline {
    agent { label'slave' }

    stages {
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                git 'https://github.com/3brazik/newww.git'

            }
        }
        stage('ci') {
            steps {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){

                    sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                    sh "docker build .  -t 3brazik/app_img"
                    sh "docker push 3brazik/app_img"
                }
            }    
        }
         stage ('deploy app'){
            steps {
                sh """
                kubectl apply -f app.yaml
                echo done
            """
            }       
        }
    }
}
```

## Build pipeline and go to console logs to see the results

![Untitled](Graduation%20Project%20d2f3965c56884900ba9fdfc0b26b4a47/Untitled%203.png)

# Then, ssh to the instance and get the IP

![Untitled](Graduation%20Project%20d2f3965c56884900ba9fdfc0b26b4a47/Untitled%204.png)

# [By Mohamed Abdel-Razik](https://www.linkedin.com/in/mohamed-abdelrazik-devops/)