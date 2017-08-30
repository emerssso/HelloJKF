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
                sh "${ tool name: 'gcs'}/bin/gcloud config set project hellojfk-a9b41"
                firebaseTest credentialsId: 'HelloJKF',
                        gcloud: "${tool name: 'gcs'}/bin/gcloud",
                        command: instrumentation(
                                app: 'app/build/outputs/apk/app-debug.apk',
                                test: 'app/build/outputs/apk/app-debug-androidTest.apk',
                                device: [device(
                                        model: 'Nexus7',
                                        version: '22',
                                        orientation: 'landscape',
                                        locale: 'en')],
                                autoGoogleLogin: true
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
}
