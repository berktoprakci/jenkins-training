{\rtf1\ansi\ansicpg1252\cocoartf2513
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\paperw11900\paperh16840\margl1440\margr1440\vieww19600\viewh13600\viewkind0
\pard\tx566\tx1133\tx1700\tx2267\tx2834\tx3401\tx3968\tx4535\tx5102\tx5669\tx6236\tx6803\pardirnatural\partightenfactor0

\f0\fs28 \cf0 pipeline \{\
    agent any\
    triggers \{ pollSCM('* * * * *') \}\
    stages \{\
        stage ('Clone') \{\
            steps \{\
                git branch: 'main', url: "https://github.com/berktoprakci/jenkins-training.git"\
            \}\
        \}\
\
        stage ('Artifactory configuration') \{\
            steps \{\
                rtMavenDeployer (\
                    id: "MAVEN_DEPLOYER",\
                    serverId: "cloudArtifactory",\
                    releaseRepo: "maven",\
                    snapshotRepo: "maven"\
                )\
            \}\
        \}\
\
        stage ('Exec Maven') \{\
            steps \{\
                withSonarQubeEnv('cloudSonar') \{\
                    rtMavenRun (\
                        tool: "autoMaven", // Tool name from Jenkins configuration\
                        pom: 'pom.xml',\
                        goals: 'install sonar:sonar',\
                        deployerId: "MAVEN_DEPLOYER",\
                    )\
                \}\
            \}   \
        \}    \
\
        stage ('Publish build info') \{\
            steps \{\
                rtPublishBuildInfo (\
                    serverId: "cloudArtifactory"\
                )\
            \}\
        \}\
        \
        stage ('Archive Artifacts') \{\
            steps \{\
                archiveArtifacts artifacts: 'target/*.war',\
                followSymlinks: false\
            \}\
        \}\
        \
        stage ('Publish Test Results') \{\
            steps \{\
                junit 'target/surefire-reports/*.xml'\
            \}\
        \}\
        \
        stage ('Deployment') \{\
            steps \{\
                octopusDeployRelease deploymentTimeout: '',\
                environment: 'Production',\
                project: 'jenkins-training',\
                releaseVersion: '0.0.16',\
                serverId: 'cloudOcto',\
                spaceId: 'Spaces-1',\
                tenant: '',\
                tenantTag: '',\
                toolId: 'Default',\
                variables: ''\
            \}\
        \}\
    \}\
    \
    post \{ \
        always \{ \
            emailext attachLog: true,\
            attachmentsPattern: 'target/surefire-reports/*.xml',\
            body: '$\{DEFAULT_CONTENT\}',\
            subject: '$\{DEFAULT_SUBJECT\}',\
            to: 'berk.toprakci@tr.keytorc.com',\
            from: 'ci-cd@tr.continium.io'\
        \}\
    \}\
\}}