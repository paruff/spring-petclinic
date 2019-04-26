def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'maven', image: 'maven:3.6.0-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'maven', image: 'maven:3.6.0-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'kubectl', image: 'lachlanevenson/k8s-kubectl:v1.8.8', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'helm', image: 'lachlanevenson/k8s-helm:latest', command: 'cat', ttyEnabled: true)
  ],
volumes: [
    hostPathVolume(mountPath: '/root/.m2/repository', hostPath: '/root/.m2/repository'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {

    node(label) {
        def myRepo = checkout scm
        def gitCommit = myRepo.GIT_COMMIT
        def gitBranch = myRepo.GIT_BRANCH
        def shortGitCommit = "${gitCommit[0..10]}"
        def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
        def gitCommitCount = sh(script: "git rev-list --all --count", returnStdout: true)
    
        // try {
        // notifySlack()

        stage('Maven project') {
            container('maven') {

                stage('Validate project') {
                    sh 'mvn -B  validate'
                }
                
                stage('Compile project') {
                    sh 'mvn -B  compile'
                }
                
            //     stage('Unit Test and coverage project') {
            //         sh 'mvn -B  test'
            //     }
                
            //    stage('Security Scan maven components') {
            //        sh 'mvn -B dependency-check:check'
            //    }
            
            //     stage ('Package and Code Analysis') {
            //         withSonarQubeEnv {
            //             sh "mvn jdepend:generate pmd:pmd findbugs:findbugs checkstyle:checkstyle   package sonar:sonar"
            //         }
            //     }
                
                // stage('Publish test results') {
                //     junit 'target/surefire-reports/*.xml'
                // } 
                
                
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
            docker build -t paruff/petclinic:latest .
            docker tag paruff/petclinic:latest paruff/petclinic:${gitCommitCount}
            docker push paruff/${POM_ARTIFACTID}:${gitCommitCount}
            """
         }
      }
    }
    stage('Run kubectl') {
      container('kubectl') {
        sh "kubectl get pods"
      }
    }

    
    // } catch (e) {
    //     currentBuild.result = 'FAILURE'
    //     throw e
    // } finally {
    //     notifySlack(currentBuild.result)
    // }
    // }
}

// def notifySlack(String buildStatus = 'STARTED') {
//     // Build status of null means success.
//     buildStatus = buildStatus ?: 'SUCCESS'

//     def color

//     if (buildStatus == 'STARTED') {
//         color = '#D4DADF'
//     } else if (buildStatus == 'SUCCESS') {
//         color = '#BDFFC3'
//     } else if (buildStatus == 'UNSTABLE') {
//         color = '#FFFE89'
//     } else {
//         color = '#FF9FA1'
//     }

//     def msg = "${buildStatus}: `${env.JOB_NAME}` #${env.BUILD_NUMBER}"

//     slackSend(color: color, message: msg)
 }