/**
 * Jenkinsfile – Pipeline CI complète
 * Projet : Boutique en ligne – ICDE848
 */

pipeline {

    agent any

    // 🔥 Déclenchement automatique à chaque push GitHub
    triggers {
        githubPush()
    }

    tools {
        maven 'Maven3'
        jdk   'JDK24'
    }

    parameters {
        string(
            name: 'BRANCH',
            defaultValue: 'main',
            description: 'Branche Git à builder'
        )
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'staging', 'prod'],
            description: 'Environnement cible'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Ignorer les tests'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm

                script {
                    def commit = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
                    echo "Branch  : ${env.BRANCH_NAME}"
                    echo "Commit  : ${commit}"
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile -B'
            }
        }

        stage('Tests unitaires') {
            when {
                not { expression { return params.SKIP_TESTS } }
            }
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
            when {
                not { expression { return params.SKIP_TESTS } }
            }
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
                        sourcePattern: '**/src/main/java',
                        minimumLineCoverage: '70'
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
                        spotbugs:spotbugs \
                        -B
                '''
            }

            post {
                always {
                    recordIssues(
                        enabledForFailure: true,
                        tools: [
                            checkStyle(pattern: '**/checkstyle-result.xml'),
                            pmdParser(pattern: '**/pmd.xml'),
                            cpd(pattern: '**/cpd.xml'),
                            spotBugs(pattern: '**/spotbugsXml.xml')
                        ],
                        qualityGates: [[
                            threshold: 10,
                            type: 'TOTAL',
                            unstable: true
                        ]]
                    )
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts(
                    artifacts: '**/target/*.jar',
                    fingerprint: true
                )
                echo "Artefact archivé avec succès"
            }
        }
    }

    post {

        always {
            echo "Pipeline terminée — statut : ${currentBuild.currentResult}"
        }

        failure {
            emailext(
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                        Build échoué

                        Projet  : ${env.JOB_NAME}
                        Build   : #${env.BUILD_NUMBER}
                        URL     : ${env.BUILD_URL}

                        Logs : ${env.BUILD_URL}console
                                        """,
                to: 'equipe-dev@monentreprise.fr',
                attachLog: true
            )
        }

        fixed {
            emailext(
                subject: "✅ FIXED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build redevenu stable : ${env.BUILD_URL}",
                to: 'equipe-dev@monentreprise.fr'
            )
        }
    }
}