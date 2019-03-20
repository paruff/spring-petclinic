def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, containers: [
    containerTemplate(name: 'maven', image: 'maven:3.6.0-jdk-8', ttyEnabled: true, command: 'cat'),
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
                
                stage('Scan components Maven project') {
                    sh 'mvn -B -Ddownloader.quick.query.timestamp=false dependency-check:check'
                }
                
                stage 'Maven Static Analysis'
                    withSonarQubeEnv {
                        sh "mvn  sonar:sonar"
                    }
                
                stage('Build a Maven project') {
                    sh 'mvn -B clean install'
                }
            
            }
        }

    }
}
