pipeline {
  agent any
  environment {
    AUTHOR = """${sh(
                returnStdout: true,
                script: "git --no-pager show -s --format='%an' ${GIT_COMMIT}"
             ).trim()}"""

    DOCKER_IMAGE_TAG = "panda:build-${env.BUILD_ID}"
  }
  stages {
    stage('Build Docker Image') {
      steps {
        timeout(time: 60, unit: 'MINUTES') {
          script {
            sh 'git clone --no-checkout --depth 1 git@github.com:commaai/xx.git || true'
            sh 'cd xx && git fetch origin && git checkout origin/master -- pandaextra && cd ..' // Needed for certs for panda flashing
            sh 'git archive -v -o panda.tar.gz --format=tar.gz HEAD'
            dockerImage = docker.build("${env.DOCKER_IMAGE_TAG}")
          }
        }
      }
    }
    stage('Test') {
      steps {
        lock(resource: "Pandas", inversePrecedence: true, quantity:1){
          timeout(time: 60, unit: 'MINUTES') {
            sh "docker run --name panda --rm --privileged --volume /dev/bus/usb:/dev/bus/usb --volume /var/run/dbus:/var/run/dbus --net host ${env.DOCKER_IMAGE_TAG} bash -c 'cd /tmp/panda; ./run_automated_tests.sh'"
          }
        }
      }
    }
  }
  post {
    always {
      script {
        dockerImage.stop()
      }
    }
  }
}