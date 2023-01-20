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
        HOST_DOMAIN = ""
        
    }
    stages {
        // stage('Git Checkout') {
        //     steps {
        //         git credentialsId: 'github-ssh-key', url: 'git@github.com:KISLAYA-SRI/Capstone-Project-Demo_Team-C.git'
        //     }
        // }
                        
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
            steps {
                script {
                    dir('helm/') {
                        sh''' 
                        HOST_NAME=$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName | tr -d '"')  
                        echo $HOST_NAME
                        HOST_DOMAIN=$DOMAIN.$HOST_NAME
                        echo $HOST_DOMAIN
                        helm upgrade $RELEASE_NAME . --set hostname=$HOST_DOMAIN --set image=$IMAGE_URI
                        '''
                    }
                }  
            }
                               
        }
        stage('Verify Deployments') {            
            steps{
                sh'''                               
                echo "Waiting for end point..."
                sleep 10                               
                echo "URL: http://$HOST_DOMAIN"
                '''
            }
        }
        

    }

    
}
