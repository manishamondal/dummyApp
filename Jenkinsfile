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
            bat 'ant zipCode'
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

            "C:/Users/manisha.mondal/AppData/Roaming/npm/sfdx" auth:jwt:grant ^
              --clientid %SFDC_CLIENT_ID% ^
              --jwtkeyfile server.key ^
              --username %SFDC_USERNAME% ^
              --instanceurl %SFDC_INSTANCE_URL% ^
              --setdefaultusername

            "C:/Program Files (x86)/sf/bin/sf" org display --target-org %SFDC_USERNAME%
            """
        }
    }

    stage('Deploy to Salesforce via Ant') {
        echo "Deploying via Ant to avoid Node.js memory issues..."
        def antHome = tool name: 'Ant', type: 'hudson.tasks.Ant$AntInstallation'
        withEnv([
            "ANT_HOME=${antHome}",
            "PATH+ANT=${antHome}\\bin"
        ]) {
            bat 'ant -version'
        }
    }

    stage('Optional: Deploy via sf CLI') {
        echo "Deploying via sf CLI (in case you want CLI deployment)..."
        withCredentials([
            string(credentialsId: 'sfdx_username', variable: 'SFDC_USERNAME')
        ]) {
            // Set Node.js memory limit to prevent crashes
            withEnv(['NODE_OPTIONS=--max_old_space_size=4096 --trace-warnings']) {
                bat '"C:/Program Files (x86)/sf/bin/sf" deploy metadata --source-dir force-app/main/default --target-org CICD_DevHub --verbose'
            }
        }
    }
}
