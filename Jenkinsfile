def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'maven', image: 'maven:3.6.0-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
    containerTemplate(name: 'gradle', image: 'gradle:5.2.1-jdk8', command: 'cat', ttyEnabled: true),
    containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true)
  ],
volumes: [
  hostPathVolume(mountPath: '/home/jenkins/.m2', hostPath: '/home/jenkins/.m2'),
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {

    node(label) {
        stage('Get a Maven project') {
            checkout scm
            container('maven') {
                
                
                stage 'Maven Static Analysis'
                    withSonarQubeEnv {
                        sh "mvn package sonar:sonar"
                    }
// TODO
//  sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target               
//                stage('Scan components Maven project') {
//                    sh 'mvn -B -Djavax.net.ssl.trustStore=/path/to/cacerts dependency-check:check'
//                }
                
                
                stage('Build a Maven project') {
                    sh 'mvn -B clean install'
                }
            
            }
        }

    }
}
