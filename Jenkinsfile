#!groovy

pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {


        stage('Build') {
            steps {
                sh './gradlew clean build'
            }
        }

        // this stage use's the plugin at
        // https://github.com/SimpleFinance/jenkins-firebase-test-plugin
        // which must be installed manually
        stage('Firebase test') {
            steps {
                firebase instrumentation(
                        app: 'app.apk',
                        test: 'app-test.apk',
                        device: 'model=Nexus7,version=19,locale=en,orientation=landscape',
                        environmentVariables: 'coverage=true,coverageFile="/sdcard/coverage.ec',
                        directoriesToPull: '/sdcard'
                )
            }
            post {
                always {
                    junit allowEmptyResults: true,
                            testResults: '.firebase/*.xml'
                    junit allowEmptyResults: true,
                            testResults: '**/testDebugUnitTest/TEST-*.xml'
                    jacoco execPattern: '**/**.exec, **/**.ec'
                    androidLint canComputeNew: true,
                            defaultEncoding: '',
                            healthy: '',
                            pattern: '**/build/reports/lint-results*.xml',
                            unHealthy: ''
                }
            }
        }
    }

    post {
        always {
            echo "Build completed. currentBuild.result = ${currentBuild.result}"
            sh "./gradlew --stop"

            deleteDir()
        }
    }
}
