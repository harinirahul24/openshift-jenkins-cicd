
      pipeline
      {
       agent {

            label 'maven'

      }


        stages
        {
          stage('Build App')
          {
            steps
             {
              git branch: 'main', url: 'https://github.com/kuldeepsingh99/openshift-jenkins-cicd.git'
              script {
                  def pom = readMavenPom file: 'pom.xml'
                  version = pom.version
              }
              sh "mvn install"
            }
          }
          

          stage('Create Image Builder') {

            when {
              expression {
                openshift.withCluster() {
                  openshift.withProject('kuldeepfr99-dev') {
                    return !openshift.selector("bc", "sample-app-jenkins-new").exists();
                  }
                }
              }
            }
            steps {
              script {
                openshift.withCluster() {
                  openshift.withProject('kuldeepfr99-dev') {
                    openshift.newBuild("--name=sample-app-jenkins-new", "--image-stream=kuldeepfr99-dev/openjdk18-openshift:1.14-3", "--binary=true")
                  }
                }
              }
            }
          }
          stage('Build Image') {
            steps {
              sh "rm -rf ocp && mkdir -p ocp/deployments"
              sh "pwd && ls -la target "
              sh "cp target/openshiftjenkins-0.0.1-SNAPSHOT.jar ocp/deployments"

              script {
                openshift.withCluster() {
                  openshift.withProject('kuldeepfr99-dev') {
                    openshift.selector("bc", "sample-app-jenkins-new").startBuild("--from-dir=./ocp","--follow", "--wait=true")
                  }
                }
              }
            }
          }
          stage('Create DEV') {
            when {
              expression {
                openshift.withCluster() {
                  openshift.withProject('kuldeepfr99-dev') {
                    return !openshift.selector('dc', 'sample-app-jenkins-new').exists()
                  }
                }
              }
            }
            steps {
              script {
                openshift.withCluster() {
                  openshift.withProject('kuldeepfr99-dev') {
                    def app = openshift.newApp("sample-app-jenkins-new", "--as-deployment-config")
                    app.narrow("svc").expose();
                  }
                }
              }
            }
          }
        }
      }
