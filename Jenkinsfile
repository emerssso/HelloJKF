#!groovy

pipeline {
    agent any

    options {
        timeout(time: 30, unit: 'MINUTES')
    }

    stages {


        stage('Build') {
            steps {
                script { currentBuild.result = 'SUCCESS' }
                sh './gradlew clean build assembleAndroidTest'
            }
        }

        stage('Firebase test') {
            steps {

                // this stage use's the plugin at
                // https://github.com/SimpleFinance/jenkins-firebase-test-plugin
                // which must be installed manually
                firebaseTest credentialsId: 'HelloJKF3',
                        gcloud: "${tool name: 'gcs'}/bin/gcloud",
                        command: instrumentation(
                                app: 'app/build/outputs/apk/app-debug.apk',
                                test: 'app/build/outputs/apk/app-debug-androidTest.apk',
                                device: [device(
                                        model: 'Nexus7',
                                        version: '22',
                                        orientation: 'landscape',
                                        locale: 'en')],
                                environmentVariables: 'coverage=true,coverageFile="/sdcard/coverage.ec"',
                                directoriesToPull: '/sdcard',
                                autoGoogleLogin: true,
                                resultsDir: "test-results/jenkins/$BUILD_NUMBER"
                        )
            }
            post {
                always {
                    junit allowEmptyResults: true,
                            testResults: '.firebase/*.xml, **/testDebugUnitTest/TEST-*.xml'
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
