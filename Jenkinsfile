pipeline {
    agent any    
    environment {
        RELEASE_NAME = 'chatapp'  
        DOMAIN = 'chatapp'   
        RESOURCE_GROUP = 'capstone-rg'   
        CLUSTER_NAME = 'capstone-aks'
        IMAGE_REPO_NAME='capstoneprojectdemoacr'
        IMAGE_NAME='nginx'
        IMAGE_TAG='v1'
        IMAGE_URI="${IMAGE_REPO_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}"
        
    }
    stages {
        stage('Git Checkout') {
            steps {
                git credentialsId: 'git-ssh-key', url: 'git@github.com:KISLAYA-SRI/Capstone-Project-Demo_Team-C.git'
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
        stage('Deploying Helm Charts') {  
            script {
                    dir('helm/') {
                        sh''' 
                        HOST_NAME=$DOMAIN.$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName -o table)  
                        helm install $RELEASE_NAME . --set hostname=$HOST_NAME image=$IMAGE_URI
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