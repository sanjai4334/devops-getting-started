## Installing Docker
  - Enter the following commands to inatall and verify installation of Docker
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker --now
docker
```
![image](https://github.com/user-attachments/assets/e0a899be-cb40-40cf-9231-e35ce45a6e4f)
![image](https://github.com/user-attachments/assets/cbfdbef3-ad47-488b-89f5-0b450c27e3af)

## Download Docker plugins in Jenkins
 - Go to Jenkins `Dashboard` -> `Manage Jenkins` -> `Available Plugins` -> Search `Docker`
 - Select these plugins and Install
    - Docker
    - Docker Commons
    - Docker Pipeline
    - docker-build-step
    - CoudBees Docker Build and Publish

![image](https://github.com/user-attachments/assets/5eef535f-2b65-44ca-9ac2-2a19548477c5)
![image](https://github.com/user-attachments/assets/3f4bda54-a3dc-4811-a546-b027d83a679f)
![image](https://github.com/user-attachments/assets/f27a51c0-f7d0-4ea2-808e-161a16d296d4)

 - In this page Check the `Restart Jenkins` after installation this will restart Jenkins

## Connectin

![image](https://github.com/user-attachments/assets/023e655e-e8e7-4b3b-9b74-317f9f4484f2)

 - Go to Jenkins `Dashboard` > `Create a Job`

![image](https://github.com/user-attachments/assets/cd3138ce-07b3-4070-97b9-0a8d76df4a36)

 - Enter a project name 
 - Select `pipeline`
 - Click `Ok`

![image](https://github.com/user-attachments/assets/3b2b76ed-f099-4257-8669-cf269e4d10a3)

 - Go to `pipeline`
 - Paste this script below:
```groovy
pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node20'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                script {
                    echo "Cleaning workspace..."
                    deleteDir() // Deletes everything in the Jenkins workspace before starting
                }
            }
        }
        

        stage('Git Checkout') {
            steps {
                script {
                    git branch: 'main', 
                        credentialsId: 'github_seccred', 
                        url: 'https://github.com/justicesecops-10/day2-app-depployement-docker.git'
                }
            }
        }

        stage('Install Node') {
            steps {
                script {
                    sh '''
                    if [ -f package.json ]; then
                        echo "package.json found. Running npm install..."
                        npm install
                    else
                        echo "ERROR: package.json is missing. Skipping npm install."
                        exit 1
                    fi
                    '''
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        def imageName = "sanjai4334/docker"
                        def tag = "latest"
                        
                        sh "docker build -t ${imageName} ."
                        sh "docker tag ${imageName} ${imageName}:${tag}"
                        sh "docker push ${imageName}:${tag}"
                    }
                }
            }
        }
   stage('Deploy Docker Container') {
    steps {
        script {
            // Stop and remove the existing container if it's running
            sh "docker stop my_container || true"
            sh "docker rm my_container || true"
            
            // Run the new container
            sh "docker run -d --name my_container -p 3001:3000 sanjai4334/docker:latest"
        }
    }
}

    }
}
```
