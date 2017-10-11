#!groovy

// https://github.com/feedhenry/fh-pipeline-library
@Library('fh-pipeline-library') _

stage('Trust') {
    enforceTrustedApproval()
}

fhBuildNode([labels: ['nodejs6-ubuntu']]) {

    final String COMPONENT = 'fh-dataman'
    final String VERSION = getBaseVersionFromPackageJson()
    final String BUILD = env.BUILD_NUMBER
    final String DOCKER_HUB_ORG = "feedhenry"
    final String CHANGE_URL = env.CHANGE_URL

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
        s3PublishArtifacts([
                bucket: "fh-wendy-builds/$COMPONENT/$BUILD",
                directory: "./dist"
        ])
    }

    stage('Platform Update') {
        final Map updateParams = [
                componentName: COMPONENT,
                componentVersion: VERSION,
                componentBuild: BUILD,
                changeUrl: CHANGE_URL
        ]
        fhcapComponentUpdate(updateParams)
        //ToDo Currently dataman isn't added to any openshift templates
        //fhOpenshiftTemplatesComponentUpdate(updateParams)
    }

    stage('Build Image') {
        dockerBuildNodeComponent(COMPONENT, DOCKER_HUB_ORG)
    }
}
