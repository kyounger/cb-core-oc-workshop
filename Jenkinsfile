library 'kypseli@master'
def podYaml = libraryResource 'podtemplates/kubectl.yml'
pipeline {
  agent {
    kubernetes {
      label 'workshop-oc-update'
      defaultContainer 'jnlp'
      yaml podYaml
    }
  }
  options { 
    buildDiscarder(logRotator(numToKeepStr: '5'))
    preserveStashes(buildCount: 5)
  }
  stages {
    stage('Update Config') {
      when { 
        branch 'master'
      }
      steps {
        container('kubectl') {
          sh('kubectl -n cje apply -f k8s/casc.yml')
          sh('kubectl -n cje apply -f k8s/kaniko.yml')
          sh('kubectl -n cje apply -f k8s/cb-core-psp.yml')
          sh('kubectl -n cje apply -f k8s/cb-oc.yml')
        } 
        sleep 600
      }
    }
    stage('Run Groovy Scripts') {
      agent { label 'master' }
      when { 
        branch 'master'
      }
      steps {
        echo "preparing Jenkins CLI"
        sh 'curl -O http://cjoc/cjoc/jnlpJars/jenkins-cli.jar'
        withCredentials([usernamePassword(credentialsId: 'cli-username-token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh """
            alias cli='java -jar jenkins-cli.jar -s \'http://cjoc/cjoc/\' -auth $USERNAME:$PASSWORD'
            cli groovy = < groovy-scripts/k8s-shared-cloud.groovy
            cli reload-jcasc-configuration
            cli build update-masters -f -v
          """
        }
      }
    }
  }
}