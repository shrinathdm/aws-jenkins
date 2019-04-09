pipeline {
  agent {
    docker {
      image 'node:10-alpine'
      args '-p 20001-20100:3000'
    }
  }
  stages {
    stage('Cache clean'){
      steps{
	sh 'mvn clean'
      }
    }
    stage('Test and Build') {
      parallel {
        stage('Run Tests') {
          steps {
            sh 'mvn test'
          }
        }
        stage('Create Build Artifacts') {
          steps {
            sh 'mvn package'
          }
        }
      }
    }
    stage('Deployment') {
      parallel {
        stage('Staging') {
          when {
            branch 'staging'
          }
          steps {
            withAWS(region:'us-east-1',credentials:'Jenkins-AWS') {
              s3Delete(bucket: 'jenkis-cicd', path:'**/*')
              s3Upload(bucket: 'jenkis-cicd', workingDir:'build', includePathPattern:'**/*');
            }
            mail(subject: 'Staging Build', body: 'New Deployment to Staging', to: 'shrinath.mangalore2@harman.com')
          }
        }
        stage('Production') {
          when {
            branch 'master'
          }
          steps {
            withAWS(region:'us-east-1',credentials:'Jenkins-AWS') {
              s3Delete(bucket: 'jenkis-cicd', path:'**/*')
              s3Upload(bucket: 'jenkis-cicd', workingDir:'build', includePathPattern:'**/*');;
            }
            mail(subject: 'Production Build', body: 'New Deployment to Production', to: 'shrinath.mangalore2@harman.com')
          }
        }
      }
    }
  }
}
