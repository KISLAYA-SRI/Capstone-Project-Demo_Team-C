pipeline {
    agent any    
    environment {
        RELEASE_NAME = 'nginx-ingress'  
        DOMAIN = 'chatapp'   
        RESOURCE_GROUP = 'capstone-rg'   
        CLUSTER_NAME = 'capstone-aks'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git credentialsId: '', url: ''
            }
        }
                        
        stage('Configure Kubectl') {
           
            steps {
                script {
                    
                    sh'''
                    az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME         
                    kubectl get nodes
                    '''                    
                }
            }
            
        }        
        stage('Ansible: Deployment') {            
            steps{                
                ansiblePlaybook colorized: true, disableHostKeyChecking: true, playbook: 'ansible/deployment.yaml'                                
            }
        }
        stage('Installing Helm Chart') {  
            script {
                    dir('helm/') {
                        sh''' 
                        HOST_NAME=$DOMAIN.$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName -o table)  
                        helm install $RELEASE_NAME . --set hostname=$HOST_NAME
                        '''
                    }
                }                     
        }
        stage('Verify Deployments') {            
            steps{
                sh'''                               
                echo "Waiting for end point..."
                sleep 10
                EXTERNAL_IP=$(kubectl get svc $RELEASE_NAME -o yaml | grep -oP '(?<=ip: )[0-9].+')
                echo 'End point ready:' && echo $EXTERNAL_IP
                echo "URL: http://$HOST_NAME"
                '''
            }
        }
        

    }

    
}