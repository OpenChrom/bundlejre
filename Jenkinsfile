pipeline {
    agent any
    tools { 
        jdk 'JDK8' 
        maven 'MAVEN3'
    }
    triggers {
        pollSCM('@daily')
    }
    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '1'))
    }
    stages {
    	stage('checkout') {
    	steps {
    			dir('bundlejre') {
					checkout scm
					//windows 32
					sh 'wget -nv -N https://github.com/OpenChrom/openchromcomp/raw/develop/openchrom/packaging/net.openchrom.rcp.compilation.community.packaging/build/jre/jre-8u102-windows-i586.tar.gz'
					sh 'tar -xvzf jre-8u102-windows-i586.tar.gz -C openchrom/features/net.openchrom.jre.win32.win32.x86.feature/jre'
					//windows 64
					sh 'wget -nv -N https://cdn.azul.com/zulu/bin/zulu8.33.0.1-ca-fx-jdk8.0.192-win_x64.zip'
					sh 'unzip -o zulu8.33.0.1-ca-fx-jdk8.0.192-win_x64.zip -d openchrom/features/net.openchrom.jre.win32.win32.x86_64.feature/jre'
					//linux x86_64
					sh 'wget -nv -N https://cdn.azul.com/zulu/bin/zulu8.33.0.1-ca-fx-jdk8.0.192-linux_x64.tar.gz'
					sh 'tar -xvzf zulu8.33.0.1-ca-fx-jdk8.0.192-linux_x64.tar.gz -C openchrom/features/net.openchrom.jre.linux.gtk.x86_64.feature/jre'
					//mac osx x86_64
					sh 'wget -nv -N https://cdn.azul.com/zulu/bin/zulu8.33.0.1-ca-fx-jdk8.0.192-macosx_x64.tar.gz'
					sh 'tar -xvzf zulu8.33.0.1-ca-fx-jdk8.0.192-macosx_x64.tar.gz -C openchrom/features/net.openchrom.jre.macosx.cocoa.x86_64.feature/jre'
				}
			}
    	}
		stage('package') {
			steps {
				sh 'mvn -B -Dmaven.repo.local=.repository -f bundlejre/openchrom/features/pom.xml install'
				sh 'mvn -B -Dmaven.repo.local=.repository -f bundlejre/openchrom/sites/pom.xml install'
			}
		}
		stage('deploy') {
			when { branch 'develop' }
			steps {
				withCredentials([string(credentialsId: 'DEPLOY_HOST', variable: 'DEPLOY_HOST')]) {
					sh 'scp -r bundlejre/openchrom/sites/net.openchrom.jre.updatesite/target/site/* '+"${DEPLOY_HOST}community/latest/jre"
				}
			}
		}
    }
    post {
        failure {
            emailext(body: '${DEFAULT_CONTENT}', mimeType: 'text/html',
		         replyTo: '$DEFAULT_REPLYTO', subject: '${DEFAULT_SUBJECT}',
		         to: emailextrecipients([[$class: 'CulpritsRecipientProvider'],
		                                 [$class: 'RequesterRecipientProvider']]))
        }
        success {
            cleanWs notFailBuild: true
        }

    }
}