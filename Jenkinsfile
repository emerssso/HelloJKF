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
                sh 'export CLOUDSDK_CONTAINER_USE_CLIENT_CERTIFICATE=True'
                sh 'gcloud config set project hellojfk-a9b41'
                /*
                // this stage use's the plugin at
                // https://github.com/SimpleFinance/jenkins-firebase-test-plugin
                // which must be installed manually
                firebaseTest credentialsId: 'HelloJKF',
                        command: instrumentation(
                                app: 'app/build/outputs/apk/app-debug.apk',
                                test: 'app/build/outputs/apk/app-debug-androidTest.apk',
                                device: [device(model: 'Nexus7',
                                        version: '22',
                                        orientation: 'landscape',
                                        locale: 'en')],
                                environmentVariables: 'coverage=true,coverageFile="/sdcard/coverage.ec"',
                                directoriesToPull: '/sdcard',
                                autoGoogleLogin: true
                        )
                */

                sh """
                gcloud beta firebase test android run \\
                    --type instrumentation \\
                    --app app/build/outputs/apk/app-debug.apk \\
                    --test app/build/outputs/apk/app-debug-androidTest.apk  \\
                    --device-ids Nexus7 \\
                    --os-version-ids 22  \\
                    --locales en  \\
                    --orientations landscape \\
                    --environment-variables coverage=true,coverageFile="/sdcard/coverage.ec" \\
                    --directories-to-pull=/sdcard \\
                    --results-dir=test-results/jenkins/
                """ + "$BUILD_NUMBER"

                sh "mkdir .firebase"
                sh "gsutil cp -r  gs://test-lab-5pthyuufc15kk-ka66n638cs6f4/test-results/jenkins/$BUILD_NUMBER/Nexus7-22-en-landscape/*.xml .firebase"
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
