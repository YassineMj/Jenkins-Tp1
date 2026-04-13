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
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Le build a échoué ❌

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Branche : ${env.GIT_BRANCH}

Logs :
${env.BUILD_URL}console
                """,
                to: 'equipe-dev@monentreprise.fr',
                attachLog: true
            )
        }

        success {
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Le build est réussi ✅

Projet  : ${env.JOB_NAME}
Build   : #${env.BUILD_NUMBER}
Branche : ${env.GIT_BRANCH}

Voir :
${env.BUILD_URL}
                """,
                to: 'equipe-dev@monentreprise.fr'
            )
        }

        unstable {
            emailext(
                subject: "⚠️ UNSTABLE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Le build est instable ⚠️

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
                body: "Le build est redevenu stable ✅ : ${env.BUILD_URL}",
                to: 'mjyassine647@gmail.com'
            )
        }
    }
}