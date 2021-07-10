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
        IMAGE_NAME = "flask-test"
    }

    stages {
        stage('Build & Push image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'quay-registry',\
                 usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    container('builder') {
                        sh "buildah --storage-driver vfs -t ${IMAGE_NAME}:${GIT_COMMIT_SHORT} bud ."
                        sh "buildah push --storage-driver vfs --creds ${USERNAME}:${PASSWORD} localhost/${IMAGE_NAME}:${GIT_COMMIT_SHORT} quay.io/rageofgods/${IMAGE_NAME}:${GIT_COMMIT_SHORT}"
                    }
                 }
            }
        }
        stage('Deploy image') {
            steps {
                podTemplate(yaml: '''
apiVersion: v1
kind: Pod
metadata:
  name: helm
spec:
  containers:
  - name: helm
    image: docker.io/alpine/helm:3.6.2
    imagePullPolicy: Always
    command: ["/bin/sh"]         #To run command inside container
    args: ["-c", "sleep 3600"]   #Specified sleep command
''') {
    node(POD_LABEL) {
        container('helm') {
            sh "git https://github.com/rageofgods/test-flask.git"
            withCredentials([file(credentialsId: "186561ed-39ab-406d-ac17-341e60226ece", variable: 'my_private_key')]) {
                    writeFile file: "${WORKSPACE}/kubeconfig", text: readFile(my_private_key)
                    sh "helm upgrade --kubeconfig=${WORKSPACE}/kubeconfig --install ${IMAGE_NAME} ./helm"
                }
        }
    }
}
            }
        }
    }
}