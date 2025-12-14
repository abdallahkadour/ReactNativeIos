pipeline {
    agent { label 'macos' }

    environment {
        NODE_ENV = 'production'
        LANG = 'en_US.UTF-8'
        LC_ALL = 'en_US.UTF-8'
    }

    options {
        timeout(time: 60, unit: 'MINUTES')
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Node Dependencies') {
            steps {
                sh '''
                    node -v
                    npm -v
                    npm ci
                '''
            }
        }

        stage('Install iOS Pods') {
            steps {
                dir('ios') {
                    sh '''
                        pod install --repo-update
                    '''
                }
            }
        }

        stage('Bundle React Native JS') {
            steps {
                sh '''
                    npx react-native bundle \
                      --platform ios \
                      --dev false \
                      --entry-file index.js \
                      --bundle-output ios/main.jsbundle \
                      --assets-dest ios
                '''
            }
        }

        stage('Build iOS App (Simulator)') {
            steps {
                dir('ios') {
                    sh '''
                        xcodebuild clean build \
                          -workspace ReactNativeIos.xcworkspace \
                          -scheme ReactNativeIos \
                          -configuration Release \
                          -sdk iphonesimulator \
                          -derivedDataPath build \
                          CODE_SIGNING_ALLOWED=NO
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ iOS build completed successfully'
        }
        failure {
            echo '❌ iOS build failed'
        }
        always {
            cleanWs()
        }
    }
}
