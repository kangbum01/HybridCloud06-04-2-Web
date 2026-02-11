pipeline {
  agent any

  environment {
    AWS_REGION     = "ap-northeast-2"
    AWS_ACCOUNT_ID = "120337556186"
    ECR_REPO_NAME  = "beat-dev-web"
    ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    ECR_IMAGE      = "${ECR_REGISTRY}/${ECR_REPO_NAME}"
  }

  stages {
    stage('Checkout') {
      steps {
        // Jenkins Job 설정에서 SCM 연동해도 되고, 여기서 checkout scm 해도 됨
        checkout scm
        sh 'ls -al'
      }
    }

    stage('ECR Login') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-cred']]) {
          sh '''
            set -euo pipefail
            aws ecr get-login-password --region ${AWS_REGION} \
              | docker login --username AWS --password-stdin ${ECR_REGISTRY}
          '''
        }
      }
    }

    stage('Build & Push') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-cred']]) {
          sh '''
            set -euo pipefail

            # 태그: build 번호 + latest 같이 푸시 추천
            TAG="${BUILD_NUMBER}"

            # Dockerfile이 레포에 있어야 함 (없으면 여기서 생성하는 fallback도 가능)
            test -f Dockerfile

            docker build -t ${ECR_IMAGE}:${TAG} .
            docker tag  ${ECR_IMAGE}:${TAG} ${ECR_IMAGE}:latest

            docker push ${ECR_IMAGE}:${TAG}
            docker push ${ECR_IMAGE}:latest
          '''
        }
      }
    }
  }
}
