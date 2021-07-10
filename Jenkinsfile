pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
    name: buildah
    annotations:
        container.apparmor.security.beta.kubernetes.io/builder: unconfined
spec:
    containers:
    - name: builder
      image: quay.io/buildah/stable
      imagePullPolicy: Always
      command: ["/bin/sh"]         #To run command inside container
      args: ["-c", "sleep 3600"]   #Specified sleep command
'''
        }
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
    }

    environment {
        GIT_COMMIT_SHORT = sh(
            script: "printf \$(git rev-parse --short ${GIT_COMMIT})",
            returnStdout: true
        )
    }

    stages {
        stage('Build image') {
            steps {
                container('builder') {
                    sh "buildah --storage-driver vfs -t ${GIT_COMMIT_SHORT} bud ."
                }
            }
        }
        stage('Deploy image') {
            steps {
                echo 'Hello World'
            }
        }
    }
}