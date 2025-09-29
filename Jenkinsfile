pipeline {
  agent any
  options {
        timeout(time: 4, unit: 'HOURS') 
    }   

  
  parameters {
    string(name: 'publishedAppVersion', defaultValue: '', description: 'Version of the ServiceNow application to install')
  }
  
  environment {
    APPSYSID = 'a6a75e9997183e941ea4f1e0f053afa1'
    CREDENTIALS = 'servicenow'
    DEVENV = 'https://hclnowintelligence.service-now.com/'
    PRODENV = 'https://ven03869.service-now.com'
    TESTSUITEID = 'bf8c266d732333005ce769972bf6a777'
  }
    

stages {
    stage('Build') {
      steps {
        script {
          snApplyChanges(appSysId: "${APPSYSID}", url: "${DEVENV}", credentialsId: "${CREDENTIALS}")
          def version = snPublishApp(
            credentialsId: "${CREDENTIALS}",
            url: "${DEVENV}",
            appSysId: "${APPSYSID}",
            isAppCustomization: true,
            obtainVersionAutomatically: true,
            incrementBy: 2
          )
          // Save the version to environment or file if needed
          env.publishedAppVersion = version
        }
      }
    }

    stage('Test') {
      steps {
        snRunTestSuite(credentialsId: "${CREDENTIALS}", url: "${DEVENV}", testSuiteSysId: "${TESTSUITEID}", withResults: true)
      }
    }

    stage('Install') {
      steps {
        snDevOpsChange()
        script {
          if (!params.publishedAppVersion && !env.publishedAppVersion) {
            error "publishedAppVersion is not set. Please ensure the application was published successfully."
          }
          snInstallApp(
            credentialsId: "${CREDENTIALS}",
            url: "${PRODENV}",
            appSysId: "${APPSYSID}",
            baseAppAutoUpgrade: false,
            publishedAppVersion: "${params.publishedAppVersion ?: env.publishedAppVersion}"
          )
        }
      }
    }
  }
}

