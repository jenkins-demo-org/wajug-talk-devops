
# Jenkins Demo

## Start the VM

* Needs Vagrant + VirtualBox
* Port 10000 must be free and open on your machine
* With a command line:

```bash
mkdir jenkins-demo-wajug
cd jenkins-demo-wajug
vagrant box add jenkins-demo-wajug <URL of Box>
vagrant init -m -f jenkins-demo-wajug
vagrant up
```

* Then, open http://localhost:10000

## Access BlueOcean and add the Pipeline

* From [Jenkins](http://localhost:10000/jenkins), click on the top button **Open Blue Ocean**
  -  Direct URL: http://localhost:10000/jenkins/blue

* Follow the Wizard to create a new pipeline:
  - Type: Git SCM
  - Repository URL: http://localhost:10000/gitserver/butler/worker.git
  - Credential: butler/butler (user / password kind)

* Once the pipeline created and complaining
about no Jenkinsfile, open the BlueOcean editor in a new tab:
  - Direct URL: http://localhost:10000/jenkins/blue/organizations/jenkins/pipeline-editor/
  - Save/Load a pipeline: use CMD + S or CTRL + S

## Access Source Code and configure it

* Start by logging in in the Gitserver:
  - Direct URL: http://localhost:10000/gitserver/butler/worker
  - User/Password for **Sign In**: butler/butler

* We want the pipeline to build at each commit/branch/PR:
  - Access the Repository **Settings** -> **WebHooks** (Direct URL: http://localhost:10000/gitserver/butler/worker/settings/hooks)
  - Add a **Gitea** WebHooks
  - Set **Payload URL** to  http://localhost:10000/jenkins/job/worker/build?delay=0
  - Set triggers to **everything**

## Simple Jenkinsfile

* Create a new file at the root of the repository, named `Jenkinsfile` with the content below:
  - Do not hesitate to use the BO editor

```groovy
pipeline {
  agent {
    dockerfile {
      filename 'Dockerfile.build'
      args '-e DOCKER_HOST=tcp://172.17.0.1:2375'
    }
  }
  environment {
    DOCKER_HOSTNAME = '172.17.0.1'
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
}
```

## Jenkinsfile with parallel support

* Add this snippet in the existing pipeline,
replacing the existing "Verify" stage:

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
        sh 'bats ./src/test/bats/*.bats'
      }
    )
  }
}
```

## Jenkinsfile with deploy and shared

* Add this snippet at the beginning, before the `pipeline` keyword

```groovy
@Library('deploy@master') _
```

* Add this snippet as last step:
```groovy
stage('Deploy') {
  steps {
    deploy 'worker:latest'
  }
}
```
