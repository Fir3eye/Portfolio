# 📢Deploy Website on S3 Bucket Using Github Actions

## 📌**Note:- Change `Bucket Name` with your `bucket name` **
```
name: Portfolio Deployment

on:
  push:
    branches:
    - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v1

    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ap-south-1

    - name: Deploy static site to S3 bucket
      run: aws s3 sync . s3://tests-portfolio --delete
```

# 📢Deploy Website on EC2 Instance Using GitHub Actions and using ECR (Elastic Container Registry) 
## 📌 **Note:- Change `REPOSITORY` with your `REPOSITORY` **

```
name: Deploy to AWS
on:
  push:
    branches:
      - "main"
env:
  AWS_REGION: ap-south-1
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  PRIVATE_SSH_KEY: ${{ secrets.AWS_SSH_KEY }}
  SERVER_PUBLIC_IP: ${{ secrets.AWS_PUBLIC_KEY }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Login to AWS ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      - name: Build, push docker image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          REPOSITORY: aws-deploy
          IMAGE_TAG: ${{ github.sha }}
        run: |-
          docker build -t $REGISTRY/$REPOSITORY:$IMAGE_TAG .
          docker push $REGISTRY/$REPOSITORY:$IMAGE_TAG
      - name: Deploy docker image to EC2
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          REPOSITORY: aws-deploy
          IMAGE_TAG: ${{ github.sha }}
          AWS_DEFAULT_REGION: ap-south-1
        uses: appleboy/ssh-action@master
        with:
          host: ${{ env.SERVER_PUBLIC_IP }}
          username: ubuntu
          key: ${{ env.PRIVATE_SSH_KEY }}
          envs: PRIVATE_SSH_KEY,REGISTRY,REPOSITORY,IMAGE_TAG,AWS_ACCESS_KEY_ID,AWS_SECRET_ACCESS_KEY,AWS_DEFAULT_REGION,AWS_REGION
          script: |-
            sudo apt update
            sudo apt install docker.io -y
            sudo apt install awscli -y
            sudo $(aws ecr get-login --no-include-email --region ap-south-1);
            sudo docker stop myappcontainer || true
            sudo docker rm myappcontainer || true
            sudo docker pull $REGISTRY/$REPOSITORY:$IMAGE_TAG
            sudo docker run -d --name myappcontainer -p 8090:80 $REGISTRY/$REPOSITORY:$IMAGE_TAG

```
