#!groovy

// https://github.com/feedhenry/fh-pipeline-library
@Library('fh-pipeline-library') _

stage('Trust') {
    enforceTrustedApproval()
}

fhBuildNode([labels: ['nodejs6-ubuntu']]) {
    stage('Install Dependencies') {
        npmInstall {}
    }

    stage('Lint') {
        print 'Lint task is broken see https://issues.jboss.org/browse/FH-3611'
        //sh 'grunt eslint'
    }

    withOpenshiftServices(['mongodb']) {
        stage('Unit Tests') {
            sh 'grunt unit'
        }

        stage('Integration Tests') {
            sh 'grunt integration'
        }
    }

    stage('Build') {
        gruntBuild {
            name = 'fh-dataman'
        }
    }

    stage('Build Image') {
        dockerBuildNodeComponent("fh-dataman")
    }
}
