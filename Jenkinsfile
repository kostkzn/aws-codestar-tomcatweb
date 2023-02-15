pipeline {
    
    
    agent { label 'docker-agent-maven' }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUSIP = "172.31.10.13"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "swapp-maven-group"
        RELEASE_REPO = "swapp-release"
        SNAP_REPO = "swapp-snapshot"
        CENTRAL_REPO = "swapp-maven-central"
        NEXUS_CREDENTIAL_ID = "nexusServerLogin"
        NEXUS_GROUP_ID = "QA"
        NEXUS_ART_ID = "swapp"
        NEXUS_USER = credentials('nexus-login') 
        NEXUS_PASS = credentials('nexus-pass')
    }

    stages{
    
        stage('Fetch code') {
            steps {
                git credentialsId: 'GitHub-token', url: 'https://github.com/kostkzn/aws-codestar-tomcatweb.git', branch: 'main'
                stash includes:'**/ansible/**', name: 'source'
            }
        }
        
        stage('Build'){
            steps {
                sh 'mvn clean install -DskipTests -s settings.xml'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('Unit Test'){
            steps {
                echo 'STAGE: Starting unit tests...'
                sh 'mvn test -s settings.xml'
            }
        }

        stage('Intergration Test'){
            steps {
                echo 'STAGE: Starting intergration test...'
                sh 'mvn verify -DskipUnitTests -s settings.xml'
            }
        }
		
        stage("Publish to Nexus Repository Manager") {
            steps {
                echo 'STAGE: Starting uploading artifact to Nexus...'
                nexusArtifactUploader artifacts: [[artifactId: 'swapp', classifier: '', file: 'target/ROOT.war', type: 'war']], credentialsId: "${NEXUS_CREDENTIAL_ID}", groupId: 'QA', nexusUrl: "${NEXUSIP}:${NEXUSPORT}", nexusVersion: "$NEXUS_VERSION", protocol: "$NEXUS_PROTOCOL", repository: "${RELEASE_REPO}", version: "v${BUILD_ID}-${BUILD_TIMESTAMP}--ROOT"
            }
        }
      
        stage("Deploy artifact to Tomcat9") {
            agent { label 'ansible-agent' }
                
            steps {
                echo 'STAGE: Starting deploying artifact to Tomcat9...'
                //ansiblePlaybook disableHostKeyChecking: true, installation: 'Ansible', inventory: 'ansible/hosts', playbook: 'ansible/swapp-setup.yml'
            sh "hostname -i"
            echo "Current folder is ${WORKSPACE}"
            unstash 'source'
            sh "ls -l"
            sh 'ansible-playbook ansible/swapp-setup.yml -i ansible/hosts -e NEXUS_USER=$NEXUS_USER -e NEXUS_PASS=$NEXUS_PASS -e NEXUSIP=$NEXUSIP -e RELEASE_REPO=$RELEASE_REPO -e NEXUS_GROUP_ID=$NEXUS_GROUP_ID -e NEXUS_ART_ID=$NEXUS_ART_ID -e BUILD_ID=$BUILD_ID -e BUILD_TIMESTAMP=$BUILD_TIMESTAMP'
            }
        }
    }        
}
