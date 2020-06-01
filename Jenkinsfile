// Jenkinsfile
String credentialsId = 'aws_test'

try {
  stage('checkout') {
    node {
      cleanWs()
      checkout scm
    }
  }
  // Run terraform init
  stage('init') {
    node {
      withCredentials([[
        $class: 'AmazonWebServicesCredentialsBinding',
        credentialsId: credentialsId,
        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
      ]]) {
        ansiColor('xterm') {
          sh 'terraform init'
        }
      }
    }
  }

  // Run terraform plan
  stage('plan') {
    node {
      withCredentials([[
        $class: 'AmazonWebServicesCredentialsBinding',
        credentialsId: credentialsId,
        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
      ]]) {
        ansiColor('xterm') {
          sh 'terraform plan'
        }
      }
    }
  }

  if (env.BRANCH_NAME == 'master') {

    // Run terraform apply
    stage('apply') {
      node {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: credentialsId,
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
          ansiColor('xterm') {
            sh 'terraform apply -auto-approve'
          }
        }
      }
    }

    // Run terraform show
    stage('show') {
      node {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: credentialsId,
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
          ansiColor('xterm') {
            sh 'terraform show'
          }
        }
      }
    }
  }
  currentBuild.result = 'SUCCESS'
}
catch (org.jenkinsci.plugins.workflow.steps.FlowInterruptedException flowError) {
  currentBuild.result = 'ABORTED'

  stage('notification'){
    slackSend baseUrl: 'https://hooks.slack.com/services/', 
    channel: '#jenkins_slack', 
    color: 'warning', 
    message: 'El pipeline fue cancelado.', 
    teamDomain: 'javahomecloud', 
    tokenCredentialId: 'slack_demo', 
    username: 'Jenkins'
  }

}
catch (err) {
  currentBuild.result = 'FAILURE'

  stage('notification'){
    slackSend baseUrl: 'https://hooks.slack.com/services/', 
    channel: '#jenkins_slack', 
    color: 'warning', 
    message: 'Una o varias instancias del pipeline fallaron.', 
    teamDomain: 'javahomecloud', 
    tokenCredentialId: 'slack_demo', 
    username: 'Jenkins'
  }


  throw err
}
finally {
  if (currentBuild.result == 'SUCCESS') {
    currentBuild.result = 'SUCCESS'

    stage('notification'){
      slackSend baseUrl: 'https://hooks.slack.com/services/', 
      channel: '#jenkins_slack', 
      color: 'good', 
      message: 'El pipeline jenkis-terraform-localstack corrio exitosamente!', 
      teamDomain: 'javahomecloud', 
      tokenCredentialId: 'slack_demo', 
      username: 'Jenkins'
    }

  }
}
