import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=bcd61b74-63f1-40b4-8369-a4fd56d15b38',
        'AZURE_TENANT_ID=0adb040b-ca22-4ca6-9447-ab7b049a22ff',
          'AZURE_CLIENT_ID=cc8eab5f-6a60-4343-9e34-828c16252c35',
           'AZURE_CLIENT_SECRET=ux-sk59k4zLS42sg5-5WCZBg_6OyWth5Mm'
          ]) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'MyTraining'
      def webAppName = 'MyWebApp352'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'ux-sk59k4zLS42sg5-5WCZBg_6OyWth5Mm', usernameVariable: 'cc8eab5f-6a60-4343-9e34-828c16252c35')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
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
