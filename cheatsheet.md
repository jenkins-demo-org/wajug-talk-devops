
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
	    args '-v /var/run/docker.sock:/var/run/docker.sock'
    }
  }
  stages {
    stage('Validate') {
      steps {
        sh 'mvn compile test'
      }
    }
    stage('Verify') {
      steps {
        sh 'git submodule update --init --recursive'
        sh 'mvn verify'
      }
    }
  }
}
```

## Jenkinsfile with parallel

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
      }
    }
    stage('Verify') {
      steps {
        parallel(
          "Integration Test": {
            sh 'git submodule update --init --recursive'
            sh 'mvn verify'
          },
          "Docker Tests": {
            sh 'bats ./src/test/bats/*.bats'
          }
        )
      }
    }
  }
}
```
