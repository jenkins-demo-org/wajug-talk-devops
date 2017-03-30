
# Cheat Sheet

## TODO

* [ ] Shared Library for maven cache
* [ ] Docker registry deploy
* [ ] Docker Swarm 2nd agent
* [ ] Jenkinsfile for example demo app with input
* [ ] Triggering beetween jobs
* [ ] Preload example demo app inside Gitea (org ?)

## BO Editor

http://localhost:10000/jenkins/blue/organizations/jenkins/pipeline-editor/

## WebHooks for MultiBranch

http://localhost:10000/jenkins/job/worker/build?delay=0

## Simple Dockerfile

```groovy
pipeline {
  agent {
    dockerfile {
      filename 'Dockerfile.build'
	    args '-e DOCKER_HOST=tcp://192.168.0.1:2375'
    }
  }
  stages {
    stage('Validate') {
      steps {
        sh 'mvn compile test'
        stash(name: 'target', includes: 'target/**')
      }
    }
    stage('Verify') {
      steps {
        sh 'git submodule update --init --recursive'
        sh 'mvn verify'
      }
    }
  }
  environment {
    DOCKER_HOSTNAME = '192.168.0.1'
    DOCKER_HOST = 'tcp://192.168.0.1:2375'
  }
}
```

## Jenkinsfile with parallel

```groovy
stage('Verify') {
  steps {
    parallel(
      "Integration Test": {
        sh 'git submodule update --init --recursive'
        sh 'mvn verify'

      },
      "Docker Tests": {
	      unstash 'target'
	      sh 'ls -l ./target/'
        sh 'bats ./src/test/bats/*.bats'
      }
    )
  }
}
```

## Jenkinsfile with deploy

```groovy
stage('Deploy') {
  steps {
    input(message: 'Is it OK to deploy Boss ?', id: 'boss')
    sh 'docker tag worker:latest 172.17.0.1:5000/worker:latest'
    sh 'docker push 172.17.0.1:5000/worker:latest'
  }
}
```

## Shared Library

```groovy
#!groovy
@Library('deploy@master') _
```
