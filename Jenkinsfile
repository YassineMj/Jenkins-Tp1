pipeline {

    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK24'
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch  : ${env.GIT_BRANCH}"
                echo "Commit  : ${env.GIT_COMMIT}"
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean compile -B'
            }
        }

        stage('Tests unitaires') {
            steps {
                sh 'mvn test -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Tests intégration') {
            steps {
                sh 'mvn verify -Dsurefire.skip=true -B'
            }
            post {
                always {
                    junit '**/target/failsafe-reports/*.xml'
                }
            }
        }

        stage('Couverture JaCoCo') {
            steps {
                sh 'mvn jacoco:report -B'
            }
            post {
                always {
                    jacoco(
                        execPattern: '**/target/jacoco.exec',
                        classPattern: '**/target/classes',
                        sourcePattern: '**/src/main/java'
                    )
                }
            }
        }

        stage('Qualité') {
            steps {
                sh '''
                    mvn checkstyle:checkstyle \
                        pmd:pmd \
                        pmd:cpd \
                        spotbugs:spotbugs -B
                '''
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
    }

    post {

        always {
            echo "Pipeline terminée — ${currentBuild.currentResult}"
        }

        failure {
            mail(
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Build FAILED

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Branche : ${env.GIT_BRANCH}
Status  : ${currentBuild.currentResult}

Logs: ${env.BUILD_URL}console
                """,
                to: 'mjyassine647@gmail.com'
            )
        }

        success {
            mail(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Build SUCCESS

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Branche : ${env.GIT_BRANCH}
Status  : ${currentBuild.currentResult}

URL: ${env.BUILD_URL}
                """,
                to: 'mjyassine647@gmail.com'
            )
        }

        unstable {
            mail(
                subject: "UNSTABLE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Build UNSTABLE

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Status  : ${currentBuild.currentResult}

URL: ${env.BUILD_URL}
                """,
                to: 'mjyassine647@gmail.com'
            )
        }
        fixed {
            mail(
                subject: "FIXED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build is now STABLE!\n\nURL: ${env.BUILD_URL}",
                to: 'mjyassine647@gmail.com'
            )

        }
    }
}