pipeline {  
    triggers {      
        pollSCM('* * * * *')  
    }
    agent {    
        kubernetes {
            // this label will be the prefix of the generated pod's name      
            label 'kubeagent'
            cloud "kubernetes"      
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    job: build-pod
  component: ci
spec:
  containers:
    - name: python
      command:
        - cat
      image: python:3.7
      tty: true

    - name: docker
      image: docker
      command:        
        - cat
      tty: true
      volumeMounts:        
        - mountPath: /var/run/docker.sock
          name: docker-sock
    - name: kubectl
      image: lachlanevenson/k8s-kubectl:v1.17.2 # use a version that match
      command:        
        - cat
      tty: true

  volumes:    
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
'''    
                }  
    }
    stages {    
        stage('Test python') {      
            steps {        
                container('python') {          
                    sh "pip install -r requirements.txt"          
                    sh "python test.py"        
                }      
            }    
        }  
        stage('Build image') {      
            steps {        
                container('docker') {          
                    sh "docker build -t localhost:4000/pythontest:latest ."         
                    
                }      
            }
        }
        stage("Push Docker Image"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_HUB_USERNAME',passwordVariable:"DOCKER_HUB_PASSWORD")]) {
                    sh 'docker login -u $DOCKER_HUB_USERNAME -p $DOCKER_HUB_PASSWORD'
                }
               sh "docker push jeanpcr94/pythontest:latest"        
                
            }
        }
        stage('Deploy') {      
            steps {        
                container('kubectl') {          
                    sh "kubectl apply -f ./kubernetes/deployment.yaml"          
                    sh "kubectl apply -f ./kubernetes/service.yaml"        
                }      
            }    
        }  
    }
    
}