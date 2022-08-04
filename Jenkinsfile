pipeline {
  agent {
    label 'devops'
  }
  environment {
    AWS_ACCESS_KEY_ID = credentials('aws_key_access')
    AWS_SECRET_ACCESS_KEY = credentials('aws_secret_key')
  }
  stages {
    stage('Create Project Folder') {
      steps {
        sh 'mkdir -p -m a=rwx ecs_basic'
      }
    }
    stage('Initialize CDK Project') {
      steps {
        dir('ecs_basic') {
          sh 'cdk init app --language python'
          sh 'export PYTHONPATH=$PWD/.venv/bin/python3.9'
        }
      }
      post {
        success {
          echo 'CDK project has been initialized successfully'
        }
        failure {
          cleanWs()
          echo 'Project initialization failed'
        }
      }
    }
    stage('Copy python code files into project folder') {
      steps {
        dir('ecs_basic') {
          sh 'cp ../app.py .'
          sh 'cp ../ecs_basic_stack.py ecs_basic/'
        }
      }
      post {
        success {
          echo 'Files copied successfully!'
        }
        failure {
          echo 'Error copying python files'
        }
      }
    }
    stage('Activate Python Virtual Environment and Deploy AWS Resources') {
      steps {
        dir('ecs_basic') {
          sh '''
          . .venv/bin/activate
          pip3 install -r requirements.txt
          cdk deploy --require-approval never
          '''
        }
      }
      post {
        success {
          echo 'waiting 5 minutes until destroy the AWS resources'
          sleep(time: 5, unit: 'MINUTES')
        }
        failure {
          sh 'cdk destroy --force'
          cleanWs()
        }
      }
    }
    stage('Destroy AWS Resources & Clean Jenkins Workspace') {
      steps {
        dir('ecs_basic') {
          sh 'cdk destroy --force'
          echo 'AWS resoruces destroyed'
        }
      }
      post {
        success {
          cleanWs()
          echo 'Jenkins workspace has been wipeout'
        }
      }
    }
  } 
}  
