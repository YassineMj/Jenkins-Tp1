pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK24'
    }

    triggers {
        githubPush()
    }

    environment {
        BRANCH_NAME = "${env.GIT_BRANCH ?: 'unknown'}"
        COMMIT_ID = "${env.GIT_COMMIT ?: 'unknown'}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch : ${env.BRANCH_NAME}"
                echo "Commit : ${env.COMMIT_ID}"
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
                sh 'mvn verify -DskipUnitTests=true -B'
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
                    mvn checkstyle:checkstyle -B
                    mvn pmd:pmd -B
                    mvn pmd:cpd -B
                    mvn spotbugs:spotbugs -B
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

        success {
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build réussi ✅

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Branch  : ${env.BRANCH_NAME}

Voir console :
${env.BUILD_URL}console
                """,
                to: 'mjyassine647@gmail.com'
            )
        }

        failure {
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build échoué ❌

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Branch  : ${env.BRANCH_NAME}

Logs :
${env.BUILD_URL}console
                """,
                to: 'mjyassine647@gmail.com',
                attachLog: true
            )
        }

        unstable {
            emailext(
                subject: "⚠️ UNSTABLE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Build instable ⚠️

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}

Voir :
${env.BUILD_URL}
                """,
                to: 'mjyassine647@gmail.com'
            )
        }

        fixed {
            emailext(
                subject: "🟢 FIXED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build redevenu stable ✅ : ${env.BUILD_URL}",
                to: 'mjyassine647@gmail.com'
            )
        }
    }
}