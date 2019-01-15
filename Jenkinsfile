#!/usr/bin/env groovy
@Library('peon-pipeline') _

node {
    def appToken
    def commitHash
    try {
        cleanWs()

        stage("checkout") {
            appToken = github.generateAppToken()

            sh "git init"
            sh "git pull https://x-access-token:$appToken@github.com/navikt/samordning-schema.git"

            sh "make bump-version"

            commitHash = sh(script: 'git rev-parse HEAD', returnStdout: true).trim()
            github.commitStatus("pending", "navikt/samordning-schema", appToken, commitHash)
        }

        stage("build") {
            sh "make"
        }

        stage("release") {
            sh "make release"
            sh "git push --tags https://x-access-token:$appToken@github.com/navikt/samordning-schema HEAD:master"
        }

        github.commitStatus("success", "navikt/samordning-schema", appToken, commitHash)
    } catch (err) {
        github.commitStatus("failure", "navikt/samordning-schema", appToken, commitHash)

        throw err
    }
}
