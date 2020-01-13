node{
	stage('Poll') {
		checkout scm
	}
	stage('Build & Unit test'){
		sh 'mvn clean verify -DskipITs=true';
      		junit '**/target/surefire-reports/TEST-*.xml'
      		archive 'target/*.war'
   	}
	stage('Static Code Analysis'){
    		sh 'mvn clean verify sonar:sonar -Dsonar.projectName=Esafe-Project -Dsonar.projectKey=Esafe-Project -Dsonar.projectVersion=$BUILD_NUMBER';
	}
	stage ('Integration Test'){
    		sh 'mvn clean verify -Dsurefire.skip=true';
		junit '**/target/failsafe-reports/TEST-*.xml'
      		archive 'target/*.war'
	}
	
 stage ('Start Tomcat'){
    		sh label: '', script: '''cd /home/ubuntu/tomcat/bin
    		./startup.sh''';
  	}
  	stage ('Deploy '){
    		unstash 'binary'
    		sh 'cp target/Esafe-0.0.1.war /home/ubuntu/tomcat/webapps/';
  	}
  	stage ('Performance Testing'){
    		sh '''cd /opt/jmeter/bin/
    		./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l $WORKSPACE/test_report.jtl''';
  		step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
  	}
        stage ('Promote build in Artifactory'){
    		def server = Artifactory.server 'Default Artifactory Server'
    		def uploadSpec = """{
    		"files": [
    		{
     		"pattern": "target/Esafe-0.0.1.war",
     		"target": "Esafe-Project/${BUILD_NUMBER}/",
	 	"props": "Integration-Tested=Yes;Performance-Tested=Yes"
   		}
           	]
		}"""
		server.upload(uploadSpec)
	}
	stage('Deploy to ansiblesaerver'){
             def server = Artifactory.server 'Default Artifactory Server'
             def downloadSpec = """{
             "files": [
              {
              "pattern": "Esafe-Project/$BUILD_NUMBER/*.war",
              "target": "/opt/ansible/",
              "props": "Performance-Tested=Yes;Integration-Tested=Yes",
              "flat": "true"
               }
               ]
               }"""
               server.download(downloadSpec)
               }
}

