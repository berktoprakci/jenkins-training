pipeline {
    agent any
    triggers { pollSCM('* * * * *') }
    stages {
        stage ('Clone') {
            steps {
                git branch: 'main', url: "https://github.com/berktoprakci/jenkins-training.git"
            }
        }

        stage ('Artifactory configuration') {
            steps {
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "cloudArtifactory",
                    releaseRepo: "maven",
                    snapshotRepo: "maven"
                )
            }
        }

        stage ('Exec Maven') {
            steps {
                withSonarQubeEnv('cloudSonar') {
                    rtMavenRun (
                        tool: "autoMaven", // Tool name from Jenkins configuration
                        pom: 'pom.xml',
                        goals: 'install sonar:sonar',
                        deployerId: "MAVEN_DEPLOYER",
                    )
                }
            }   
        }    

        stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "cloudArtifactory"
                )
            }
        }
        
        stage ('Archive Artifacts') {
            steps {
                archiveArtifacts artifacts: 'target/*.war',
                followSymlinks: false
            }
        }
        
        stage ('Publish Test Results') {
            steps {
                junit 'target/surefire-reports/*.xml'
            }
        }
        
        stage ('Deployment') {
            steps {
                octopusDeployRelease deploymentTimeout: '',
                environment: 'Production',
                project: 'jenkins-training',
                releaseVersion: '0.0.16',
                serverId: 'cloudOcto',
                spaceId: 'Spaces-1',
                tenant: '',
                tenantTag: '',
                toolId: 'Default',
                variables: ''
            }
        }
    }
    
    post { 
        always { 
            emailext attachLog: true,
            attachmentsPattern: 'target/surefire-reports/*.xml',
            body: '${DEFAULT_CONTENT}',
            subject: '${DEFAULT_SUBJECT}',
            to: 'berk.toprakci@tr.keytorc.com',
            from: 'ci-cd@tr.continium.io'
        }
    }
}