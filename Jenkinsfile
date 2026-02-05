pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "671905743549"
        ECR_REPO       = "hello-java-app"
        ECR_URI        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        CLUSTER_NAME   = "my-eks-cluster"
        IMAGE_TAG      = "${BUILD_NUMBER}"  
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'git-pat-token',
                    url: 'https://github.com/subhambehera12/hello-java-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                if [ ! -f target/hello-java-app-1.0.jar ]; then
                  echo "ERROR: JAR file not found!"
                  exit 1
                fi

                docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']
                ]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_REGION \
                    | docker login --username AWS --password-stdin $ECR_URI

                    docker tag ${ECR_REPO}:${IMAGE_TAG} $ECR_URI/${ECR_REPO}:${IMAGE_TAG}
                    docker push $ECR_URI/${ECR_REPO}:${IMAGE_TAG}
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-creds']
                ]) {
                    sh '''
                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

                    # Update deployment.yaml dynamically with the current build tag
                    sed -i "s|image: .*|image: ${ECR_URI}/${ECR_REPO}:${IMAGE_TAG}|g" deployment.yaml

                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml
                    '''
                }
            }
        }
    }
}
