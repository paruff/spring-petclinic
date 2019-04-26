def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'maven', image: 'maven:3.6.0-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'maven', image: 'maven:3.6.0-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
volumes: [
    hostPathVolume(mountPath: '/root/.m2/repository', hostPath: '/root/.m2/repository'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {

    node(label) {
        try {
        notifySlack()

        def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
        
        stage('Get a Maven project') {
            checkout scm
            container('maven') {

                stage('Validate project') {
//                    slackSend (  message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>")
                    sh 'mvn -B  validate'
                }
                
                stage('Compile project') {
                    sh 'mvn -B  compile'
                }
                parallel (
                stage('Unit Test and coverage project') {
                    sh 'mvn -B  test'
                },
                
// TODO
//  sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target               
//                stage('Scan components Maven project') {
//                    sh 'mvn -B -Djavax.net.ssl.trustStore=/path/to/cacerts dependency-check:check'
//                }

               stage('Security Scan maven components') {
                   sh 'mvn -B dependency-check:check'
               },
            
                stage ('Package and Code Analysis') {
                    withSonarQubeEnv {
                        sh "mvn jdepend:generate pmd:pmd findbugs:findbugs checkstyle:checkstyle   package sonar:sonar"
                        // com.hello2morrow:sonargraph-maven-plugin:dynamic-report -Dsonargraph.sonargraphBuildVersion=newest -Dsonargraph.prepareForSonarQube=true
                    }
                },
                
                stage('Publish test results') {
                    junit 'target/surefire-reports/*.xml'
                } 
                )
                
            }
        }
        stage('Create Docker images') {
      container('docker') {
        withCredentials([[$class: 'UsernamePasswordMultiBinding',
          credentialsId: 'dockerreg',
          usernameVariable: 'DOCKER_REG_USER',
          passwordVariable: 'DOCKER_REG_PASSWORD']]) {
          sh """
            docker login ${registy-url} -u ${registry-user} -p ${registry-pw}
            docker build -t paruff/petclinic:${gitCommit} .
            docker push paruff/petclinic:${gitCommit}
            """
        }
      }
    }

    
       // Existing build steps.
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