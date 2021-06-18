pipeline {
    agent any
    triggers { pollSCM('* * * * *') }
    stages {
        stage('Clone') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'main', url: 'https://github.com/berktoprakci/jenkins-training.git'
            }
        }
        
        stage('Artifactory Env. Conf.') {
            steps {
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "cloudArtifactory",
                    releaseRepo: "maven",
                    snapshotRepo: "maven"
                )
            }
        }
        
        stage('Build & Sonar') {
            steps {
                withSonarQubeEnv('cloudSonar') {
                    rtMavenRun (
                        tool: "autoMaven",
                        pom: 'pom.xml',
                        goals: 'install sonar:sonar',
                        deployerId: "MAVEN_DEPLOYER"
                    )
                }
            }
        }
        
        stage('Publish Buid Info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "cloudArtifactory"
                )
            }
        }
    }
    
    post {
        always {
            // One or more steps need to be included within each condition's block.
            junit 'target/surefire-reports/*.xml'
            emailext from:'ci-cd@tr.continium.io', attachLog: true, attachmentsPattern: 'target/surefire-reports/*.xml', body: '${DEFAULT_CONTENT}', subject: '${DEFAULT_SUBJECT}', to: 'berk.toprakci@tr.keytorc.com'
        }
        success {
            // One or more steps need to be included within each condition's block.
            archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
            octopusDeployRelease deploymentTimeout: '', environment: 'Production', project: 'jenkins-training', releaseVersion: '0.0.28', serverId: 'cloudOcto', spaceId: 'Spaces-1', tenant: '', tenantTag: '', toolId: 'Default', variables: ''
        }
    }
}