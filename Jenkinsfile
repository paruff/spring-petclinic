def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'maven', image: 'maven:3.6.0-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true)
  ],
volumes: [
    hostPathVolume(mountPath: '/root/.m2/repository', hostPath: '/root/.m2/repository'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {

    node(label) {
        def myRepo = checkout scm
        def gitCommit = myRepo.GIT_COMMIT
        def gitBranch = myRepo.GIT_BRANCH
        def branchName = sh(script: "echo $gitBranch | cut -c8-", returnStdout: true)
        def shortGitCommit = "${gitCommit[0..10]}"
        def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
        def gitCommitCount = sh(script: "git rev-list --all --count", returnStdout: true)
        def regURL = "registry-sonatype-nexus.pipeline:8081/docker-internal"
        def regNamespace = "paruff"
        def artifactID = sh(script: "grep '<artifactId>' pom.xml | head -n 1 | sed -e 's/artifactId//g' | sed -e 's/\\s*[<>/]*//g' | tr -d '\\r\\n'", returnStdout: true)
        def POMversion = sh(script: "grep '<version>' pom.xml | head -n 1 | sed -e 's/version//g' | sed -e 's/\\s*[<>/]*//g' | tr -d '\\r\\n'", returnStdout: true)
 
        try {
        notifySlack()

        stage('Maven project') {
            container('maven') {

                stage('Validate project') {
                    sh 'mvn -B  validate'        
                }
                
                stage('Compile project') {
                    sh 'mvn -B  compile package'
                }
                
                stage('Unit Test and coverage project') {
                    sh 'mvn -B  test'
                }
                
               stage('Security Scan maven components') {
                   sh 'mvn -B dependency-check:check'
               }
            
                stage ('Package and Code Analysis') {
                    withSonarQubeEnv {
                        sh "mvn jdepend:generate pmd:pmd findbugs:findbugs checkstyle:checkstyle   package sonar:sonar"
                    }
                }
                stage("Quality Gate"){
          timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
          }
      }
                
                stage('Publish test results') {
                    junit 'target/surefire-reports/*.xml'
                } 
                
                
            }
        }
        stage('Create Docker images') {
          container('docker') {
         withCredentials([[$class: 'UsernamePasswordMultiBinding',
           credentialsId: 'dockerreg',
           usernameVariable: 'DOCKER_REG_USER',
           passwordVariable: 'DOCKER_REG_PASSWORD']]) {
          sh """
            docker login -u ${DOCKER_REG_USER}  -p ${DOCKER_REG_PASSWORD}
            docker build -t ${regNamespace}/${artifactID} .
            docker tag ${regNamespace}/${artifactID} ${regNamespace}/${artifactID}:${POMversion}.${shortGitCommit}
            echo $gitBranch
            echo $branchName
            if [ ${gitBranch} == "origin/master" ] ; then
                docker tag ${regNamespace}/${artifactID} ${regNamespace}/${artifactID}:${POMversion}.${gitCommitCount}
                docker tag ${regNamespace}/${artifactID} ${regNamespace}/${artifactID}:${POMversion}.${BUILD_NUMBER}
            else
                docker tag ${regNamespace}/${artifactID} ${regNamespace}/${artifactID}:${branchName}-${POMversion}.${gitCommitCount}
                docker tag ${regNamespace}/${artifactID} ${regNamespace}/${artifactID}:${branchName}-${POMversion}.${BUILD_NUMBER}
            fi
            docker push ${regNamespace}/${artifactID}
            """
         }
      }
    }
    stage('deploy 2 k8s') {
      container('kubectl') {
        sh "kubectl get pods"
        sh "kubectl set image deployments/${artifactID} ${artifactID}=${regNamespace}/${artifactID}:${POMversion}.${gitCommitCount}"
        sh "kubectl expose deployment ${artifactID} --type=LoadBalancer --port=8080"

      }
    }

    
    } catch (e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        notifySlack(currentBuild.result)
    }
    }
}

def notifySlack(String buildStatus = 'STARTED') {
    // Build status of null means success.
    buildStatus = buildStatus ?: 'SUCCESS'

    def color

    if (buildStatus == 'STARTED') {
        color = '#D4DADF'
    } else if (buildStatus == 'SUCCESS') {
        color = '#BDFFC3'
    } else if (buildStatus == 'UNSTABLE') {
        color = '#FFFE89'
    } else {
        color = '#FF9FA1'
    }

    def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}"

    slackSend(color: color, message: msg)
 }
