import jenkins.*
import hudson.*
import hudson.model.*
import groovy.json.JsonOutput

properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '15', artifactNumToKeepStr: '5', daysToKeepStr: '15', numToKeepStr: '5')), disableConcurrentBuilds()])

node ('master') {
    // Clean workspace before doing anything
    deleteDir()

    try {
        notifyBuild('STARTED')
        String jdktool = tool name: "java_home", type: 'hudson.model.JDK'
        def mvnHome = tool name: 'maven3'
        def MAVEN_SETTINGS = "/usr/local/src/apache-maven/conf/settings.xml"
        
        List javaEnv = [
                       "PATH+MVN=${jdktool}/bin:${mvnHome}/bin",
                       "M2_HOME=${mvnHome}",
                       "JAVA_HOME=${jdktool}"]
        withEnv(javaEnv) {

        stage ('Clone') {
              checkout([
                    $class: 'GitSCM',
                    branches: scm.branches,
                    extensions: scm.extensions + [[$class: 'CleanCheckout']] + [[$class: 'CloneOption', shallow: true]],
                    userRemoteConfigs: scm.userRemoteConfigs
                ])
        }
        stage ('Build') {
                sh "mvn -s $MAVEN_SETTINGS clean package"     
            }
       
        stage('Push version Back to Git (tag)') {
        
            git branch: "${BRANCH_NAME}", credentialsId: 'githubkey'
            sh('git tag -a "${BRANCH_NAME}_1.0.${BUILD_NUMBER}.0" -m "${BRANCH_NAME} Build Version #1.0.${BUILD_NUMBER}.0" ')
            sh('git push origin --tags')

	        }

        }
        

    }
    catch (err) {
        currentBuild.result = 'FAILED'
        cleanWs()
        throw err

    }
    finally {
    // Success or failure, always send notifications
    cleanWs()

    }
}
