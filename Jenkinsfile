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
                    sh "${tool name: 'gcs'}/bin/gcloud auth activate-service-account --key-file=$authfile"
                }

                sh "${tool name: 'gcs'}/bin/gcloud config set project hellojfk-a9b41"
                sh "${tool name: 'gcs'}/bin/gcloud components install beta"

                sh "${tool name: 'gcs'}/bin/gcloud beta firebase test android run " +
                        "--type instrumentation " +
                        "--app app/build/outputs/apk/app-debug.apk " +
                        "--test app/build/outputs/apk/app-debug-androidTest.apk " +
                        "--device-ids Nexus6P " +
                        "--os-version-ids 25 " +
                        "--locales en " +
                        "--orientations landscape " +
                        "--results-dir test-results/jenkins/$BUILD_NUMBER " +
                        "--environment-variables coverage=true,coverageFile=\"/sdcard/coverage.ec\" " +
                        "--directories-to-pull=/sdcard"

                sh "mkdir firebase && " +
                        "${tool name: 'gcs'}/bin/gsutil cp " +
                        "gs://test-lab-5pthyuufc15kk-ka66n638cs6f4/test-results/jenkins/" +
                        "$BUILD_NUMBER/Nexus6P-25-en-landscape/*.xml firebase | true"

                sh "${tool name: 'gcs'}/bin/gsutil cp " +
                        "gs://test-lab-5pthyuufc15kk-ka66n638cs6f4/test-results/jenkins/" +
                        "$BUILD_NUMBER/Nexus6P-25-en-landscape/*.ec firebase | true"
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
                            testResults: 'firebase/**/*.xml, **/testDebugUnitTest/TEST-*.xml'
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
