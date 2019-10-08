def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'gradle', image: 'keeganwitt/docker-gradle-root', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
],
volumes: [
  hostPathVolume(mountPath: '/home/gradle/.gradle', hostPath: '/tmp/jenkins/.gradle'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) 
{
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
    sh "whoami"
 
     stage('Run kubectl') {
      container('kubectl') {
        echo "Inside kubectl user!"
        sh "whoami"
        sh "kubectl get pods"
      }
    }
    stage('Build Project') {
      try {
        container('gradle') {
            sh "pwd"
            echo "Inside Gradle container!"
            sh "whoami"
            sh "chmod +x gradlew"
            sh "./gradlew build -xtest"
           // sh "./gradlew bootRun"
           // """
        }
      }
      catch (exc) {
        println "Failed to test - ${currentBuild.fullDisplayName}"
        throw(exc)
      }
    }
    stage('Contract Test') {
      container('gradle') {
      sh "chmod +x gradlew"
      sh "./gradlew test pactPublish"
      //sh "./gradlew pactVerify"
      }
    }
    
    stage('Create Docker image') {
      container('docker') {
          sh """
            docker login -u 'contactsai123' -p 'xxxx'
            docker build -t contactsai123/userms .
            docker push contactsai123/userms
            """
        }
      }

    
  }
}
