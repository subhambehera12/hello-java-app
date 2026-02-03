pipeline {
    

    environment {
        AWS_REGION      = "ap-south-1"
        AWS_ACCOUNT_ID  = "123456789012"
        ECR_REPO        = "hello-java-app"
        IMAGE_TAG       = "latest"
        ECR_URI         = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        CLUSTER_NAME    = "my-eks-cluster"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Cloning GitHub repository..."
                git url: 'https://github.com/<your-github>/hello-java-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "Building Java application with Maven..."
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Creating Dockerfile and building Docker image..."
                sh '''
                cat <<EOF > Dockerfile
                FROM openjdk:11-jre-slim
                WORKDIR /app
                COPY target/hello-java-app-1.0.jar app.jar
                CMD ["java","-jar","app.jar"]
                EOF

                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                echo "Logging in to AWS ECR and pushing image..."
                sh '''
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin $ECR_URI

                docker tag $ECR_REPO:$IMAGE_TAG $ECR_URI/$ECR_REPO:$IMAGE_TAG
                docker push $ECR_URI/$ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                echo "Updating kubeconfig and deploying to EKS..."
                sh '''
                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

                cat <<EOF > deployment.yaml
                apiVersion: apps/v1
                kind: Deployment
                metadata:
                  name: hello-java
                spec:
                  replicas: 2
                  selector:
                    matchLabels:
                      app: hello-java
                  template:
                    metadata:
                      labels:
                        app: hello-java
                    spec:
                      containers:
                      - name: hello-java
                        image: ${ECR_URI}/${ECR_REPO}:${IMAGE_TAG}
                        ports:
                        - containerPort: 8080
                EOF

                kubectl apply -f deployment.yaml
                '''
            }
        }
    }
}
