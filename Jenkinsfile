node {
    stage('Checkout') {
        echo "Checking out code from Git..."
        checkout([$class: 'GitSCM',
            branches: [[name: '*/main']],
            userRemoteConfigs: [[url: 'https://github.com/manishamondal/dummyApp.git']]
        ])
    }

    stage('Build & Package') {
        echo "Running Ant build & package..."
        def antHome = tool name: 'Ant', type: 'hudson.tasks.Ant$AntInstallation'
        withEnv([
            "ANT_HOME=${antHome}",
            "PATH+ANT=${antHome}\\bin"
        ]) {
            bat 'ant -version'
            bat 'ant -f build.xml package'
        }
    }

    stage('Authenticate to Salesforce (JWT)') {
        echo "Authenticating to Salesforce via JWT..."
        withCredentials([
            file(credentialsId: 'sfdx_jwt_key', variable: 'JWT_KEY_FILE'),
            string(credentialsId: 'sfdx_client_id', variable: 'SFDC_CLIENT_ID'),
            string(credentialsId: 'sfdx_username', variable: 'SFDC_USERNAME'),
            string(credentialsId: 'sfdx_instance_url', variable: 'SFDC_INSTANCE_URL')
        ]) {
            bat """
            type "%JWT_KEY_FILE%" > server.key
            if "%SFDC_INSTANCE_URL%"=="" ( set SFDC_INSTANCE_URL=https://login.salesforce.com )

            "C:/Users/manisha.mondal/AppData/Roaming/npm/sfdx" force:auth:jwt:grant ^
              --clientid %SFDC_CLIENT_ID% ^
              --jwtkeyfile server.key ^
              --username %SFDC_USERNAME% ^
              --instanceurl %SFDC_INSTANCE_URL% ^
              --setdefaultusername

            "C:/Users/manisha.mondal/AppData/Roaming/npm/sfdx" force:org:display --targetusername %SFDC_USERNAME%
            """
        }
    }

    stage('Deploy to Salesforce (MDAPI)') {
        echo "Deploying via MDAPI..."
        withCredentials([
            string(credentialsId: 'sfdx_username', variable: 'SFDC_USERNAME')
        ]) {
                withEnv(['NODE_OPTIONS=--max-old-space-size=4096 --trace-warnings']) {
   
    bat '"C:/Users/manisha.mondal/AppData/Roaming/npm/sfdx" deploy metadata --source-dir force-app/main/default --target-org CICD_DevHub --verbose'
}

            }
        
    }
}
