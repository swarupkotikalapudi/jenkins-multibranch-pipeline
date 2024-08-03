def getTimestamp() {
    def now = new Date()
    return now.format("yyyyMMdd_HHmm", TimeZone.getTimeZone('UTC'))
}
def getGitBranchName() {
    return "FAB-319"
}

pipeline {
 options {
     buildDiscarder(logRotator(numToKeepStr: '15', daysToKeepStr: '15',  artifactNumToKeepStr: '5', artifactDaysToKeepStr: '15'))
 }
 agent {
    node {
      label 'linuxagent'
    }
 }
 tools {
    jdk 'JDK_1.8'
 }
 environment {
    GIT_BRANCH_NAME = "${getGitBranchName()}"
 }
 
 // trigger this jenkins job every Firday 7 PM UTC time
// triggers {
    // tmp to be removed, poll every one hour
    //pollSCM 'H * * * *'
 //}
 
 stages {
    stage('Clean Workspace') {
        steps {
            deleteDir()
        }
    }
    stage('checkout fabricmci') {
      steps {
        withCredentials([gitUsernamePassword(credentialsId: 'source:FabricmciGSA', gitToolName: 'Default')]) {
          sh "git clone -b $GIT_BRANCH_NAME https://source.developers.google.com/p/fabricmanager/r/podm-common"
          echo "Branch name is: ${BRANCH_NAME}"
        }
      }
    }
    stage('Clean') {
        steps {
            dir("podm-common") {
                gradlew('clean')
            }
        }
    }
    stage('Compile') {
        steps {
            dir("podm-common") {
                gradlew('build')
                echo "branch name is : ${BRANCH_NAME}"
            }
        }
    }

    stage('Publish') {
            environment {
                //nexus_url = 'https://10.52.0.26/repository/maven-releases/'

                nexus_cred = credentials('FM_NEXUS_REPO_JAVA_JAR_PUBLISH_CRED')
                nexus_user = "${nexus_cred_USR}"
                nexus_pass = "${nexus_cred_PSW}"
            }
            when {
                expression {
                    def branchMarkedForPublish=env.BRANCH_NAME
                    return branchMarkedForPublish == "master" ||
                           branchMarkedForPublish == "release"
                }
            }
            steps {
               dir("podm-common") {
                gradlew('publish')
                }
            }
    }
 }
 post {
    always {
      echo "Build finished.. with status : ${currentBuild.currentResult}"
    }
    //success {
    //  echo "it is a success"
    //    script {
    //      slackSend (
    //        color: 'good',
    //        channel: '#jenkins-ci-builds',
    //        message: "Job ${env.JOB_NAME}\nbuild ${env.BUILD_NUMBER}\nMore info at: ${env.BUILD_URL}"
    //        )
    //    }
    //}
    //failure {
      //echo "it is a failure"
        //script {
          //slackSend (
            //color: 'danger',
            //channel: '#jenkins-ci-builds',
            //message: "${currentBuild.currentResult} Job ${env.JOB_NAME}\nbuild ${env.BUILD_NUMBER}\nMore info at: ${env.BUILD_URL}"
            //)
        //}
    //}
 }
}

def gradlew(String... args) {
    sh "./gradlew ${args.join(' ')} -s"
}
