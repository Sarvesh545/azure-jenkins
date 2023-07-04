import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=5aebd2c3-b54c-4d15-add4-fd170e97c890',
        'AZURE_TENANT_ID=44b6398f-0fe5-4261-8bb1-5f16c24c86a8']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'sarvjenkins'
      def webAppName = 'sarveshazure'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '01b756a5-7755-4a20-9b17-2ccd4182fcaa', passwordVariable: 'b2491d70-0039-41b8-b5fe-88ba8fdb8dd4', usernameVariable: '01b756a5-7755-4a20-9b17-2ccd4182fcaa')]) {
       sh '''
          az login --01b756a5-7755-4a20-9b17-2ccd4182fcaa -u 01b756a5-7755-4a20-9b17-2ccd4182fcaa -p b2491d70-0039-41b8-b5fe-88ba8fdb8dd4 -t 44b6398f-0fe5-4261-8bb1-5f16c24c86a8
          az account set -s 5aebd2c3-b54c-4d15-add4-fd170e97c890
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      sh 'az logout'
    }
  }
}
