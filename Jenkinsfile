
node(){
    try{
    stage('Checkout'){
        echo "Checkout stage"
        git url:"https://github.com/cndacademy/hello-world-war.git"
      }
     stage('Sonar Analysis'){
        withSonarQubeEnv('SonarCloud'){
        
       sh " mvn sonar:sonar"
       sh 'sleep 1m'
        }
    }
    stage("Quality Gate Result"){
      timeout(time: 15, unit: 'MINUTES') {
      def qg = waitForQualityGate()
      if (qg.status != 'OK') {
         currentBuild.result = "FAILURE"
         error "Pipeline aborted due to quality gate failure: ${qg.status}"
          }
       }
      }
    stage('Build'){
        echo "Build stage"
		sh 'mvn clean package -Dmaven.test.skip=true'
		archiveArtifacts 'target/hello-app.war'
    }
    stage('Test'){
        echo "Im in testing"
		sh 'mvn test '
    }
	stage('Publish Artifcts: Nexus'){
	    echo "Publishing Artifacts to Nexus"
	    nexusPublisher nexusInstanceId: 'nexus-server-3', nexusRepositoryId: 'maven-releases', packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: 'target/hello-app.war']], mavenCoordinate: [artifactId: 'hello-world-war', groupId: 'com.efsavage', packaging: 'war', version: '1.2']]]
	   
	 }
	stage('Deploy Dev'){
	  echo "Deploy to Dev Server"
	  sh "sudo cp target/hello-app.war /var/lib/tomcat8/webapps"
	}

	stage('Approval to deploy Prod'){
	    timeout(time: 15, unit: 'MINUTES'){
	    input message: 'Do you approve deployment for production?' , ok: 'Yes'
	    }
	}
   
    stage("Deploy Prod"){
             echo "Finally approved to Production"
          sshPublisher(publishers: [sshPublisherDesc(configName: 'production-server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/hello-app.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
           //  sh "sudo scp -o StrictHostKeyChecking=no  target/*.war root@34.69.158.179:/var/lib/tomcat8/webapps"
         }
    currentBuild.result = 'SUCCESS'
 }
 catch(err){
    currentBuild.result = 'FAILURE'
    }
finally{
    mail to:"ajitgarad333@gmail.com mahadeva.garad1@gmail.com garadms@gmail.com",
             subject: "Status of pipeline: ${currentBuild.fullDisplayName}",
             body: "${env.BUILD_URL} has result ${currentBuild.result} " 
}
}
