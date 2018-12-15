#!/usr/bin/env groovy

node {

  stage('Checkout') {
    checkout scm
  }

  stage('Build') {
    docker.image('zenoss/build-tools:0.0.10').inside('-u 0') { 
      sh '''
        mkdir -p $GOPATH/src/github.com/zenoss/
        ln -s $PWD $GOPATH/src/github.com/zenoss/metricshipper
        make -C $GOPATH/src/github.com/zenoss/metricshipper MIN_GO_VERSION=go1.6 clean
        make -C $GOPATH/src/github.com/zenoss/metricshipper MIN_GO_VERSION=go1.6 tgz' 
      '''
    }
  }

  stage('Publish') {
    def remote = [:]
    withFolderProperties {
      withCredentials( [sshUserPrivateKey(credentialsId: 'PUBLISH_SSH_KEY', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')] ) {
        remote.name = env.PUBLISH_SSH_HOST
        remote.host = env.PUBLISH_SSH_HOST
        remote.user = userName
        remote.identityFile = identity
        remote.allowAnyHosts = true

        def tar_ver = sh( returnStdout: true, script: "cat VERSION" ).trim()
        sshPut remote: remote, from: 'metricshipper-' + tar_ver + '.tgz', into: env.PUBLISH_SSH_DIR
      }
    }
  }

}
