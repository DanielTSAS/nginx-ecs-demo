pipeline {
    agent any
    parameters {
        string(name: 'AWS_REGION', defaultValue: 'sa-east-1', description: 'Región de AWS donde están tus recursos ECS y ECR.')
        string(name: 'ECR_REPO', defaultValue: 'nginx-ecs-demo', description: 'Nombre de tu repositorio en AWS ECR.')
        string(name: 'ECS_CLUSTER', defaultValue: 'ecs-lab-cluster', description: 'Nombre de tu cluster de ECS.')
        string(name: 'ECS_SERVICE', defaultValue: 'nginx-lab-svc', description: 'Nombre de tu servicio dentro del cluster de ECS.')
        string(name: 'TASK_FAMILY', defaultValue: 'nginx-lab-task', description: 'Nombre de la familia de tu Task Definition en ECS.')
        string(name: 'ACCOUNT_ID', defaultValue: 'TU_ACCOUNT_ID', description: 'Tu ID de cuenta de AWS de 12 dígitos.')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Push') {
            steps {
                script {
                    // Construimos la imagen Docker
                    sh "docker build -t ${ECR_REPO} ."
                    
                    // Definimos la URL completa del repositorio ECR
                    def ecrRepoUrl = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
                    
                    // Iniciamos sesión en ECR
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
                    
                    // Etiquetamos la imagen con la URL del repo y la etiqueta 'latest'
                    sh "docker tag ${ECR_REPO}:latest ${ecrRepoUrl}:latest"
                    
                    // Subimos la imagen a ECR
                    sh "docker push ${ecrRepoUrl}:latest"
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Este paso requiere 'jq' instalado en el agente de Jenkins [cite: 88]
                    sh """
                    aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} --force-new-deployment --region ${AWS_REGION}
                    """
                }
            }
        }
    }
}