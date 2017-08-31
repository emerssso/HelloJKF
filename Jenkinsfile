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
                withCredentials([file(credentialsId: 'HelloJKF-File', variable: 'authfile')]) {
                    sh "gcloud auth activate-service-account --key-file=$authfile"
                }

                sh "${ tool name: 'gcs'}/bin/gcloud config set project hellojfk-a9b41"

                sh "gcloud beta firebase test android run \\\n" +
                        "--type instrumentation \\\n" +
                        "--app build/outputs/apk/app-debug.apk \\\n" +
                        "--test build/outputs/apk/app-debug-androidTest.apk \\\n" +
                        "--device-ids Nexus6P\\\n" +
                        "--os-version-ids 25 \\\n" +
                        "--locales en \\\n" +
                        "--orientations landscape \\\n" +
                        "--results-dir test-results/jenkins/$BUILD_NUMBER \\\n" +
                        "--environment-variables coverage=true,coverageFile=\"/sdcard/coverage.ec\" \\\n" +
                        "--directories-to-pull=/sdcard"
                /*firebaseTest credentialsId: 'HelloJKF',
                        gcloud: "${tool name: 'gcs'}/bin/gcloud",
                        command: instrumentation(
                                app: 'app/build/outputs/apk/app-debug.apk',
                                test: 'app/build/outputs/apk/app-debug-androidTest.apk',
                                device: [device(
                                        model: 'Nexus7',
                                        version: '22',
                                        orientation: 'landscape',
                                        locale: 'en')],
                                //environmentVariables: 'coverage=true,coverageFile="/sdcard/coverage.ec"',
                                //directoriesToPull: '/sdcard',
                                autoGoogleLogin: true,
                                resultsDir: "test-results/jenkins/$BUILD_NUMBER"
                        )
                        */
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
