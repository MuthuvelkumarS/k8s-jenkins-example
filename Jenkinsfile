pipeline {
    agent any
    environment {
        DEPLOY = "${env.BRANCH_NAME == "master" || env.BRANCH_NAME == "develop" ? "true" : "false"}"
        NAME = "${env.BRANCH_NAME == "master" ? "example" : "example-staging"}"
        VERSION = readMavenPom().getVersion()
        DOMAIN = 'localhost'
        REGISTRY = 'smvkumar/k8s-jenkins-example'
        REGISTRY_CREDENTIAL = 'muthu-dockerhub'
        // -- These credentials should be set under Jenkins credentials
        // -- Aws cli will take the credentials from the below environment
        // -- variables to access aws resources
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = credentials('AWS_DEFAULT_REGION')
        // -- Set configfile to Kubectl configuration environment variable
        // -- so that kubectl will take the cluster configuration from here
        KUBECONFIG = credentials('AWS_EKS_KUBECONFIG_FILE')
        // -- This is an user defined variables
        AWS_ACCOUNT_ID = credentials('AWS_ACCOUNT_ID')
    }
    //agent {
    //    kubernetes {
    //        defaultContainer 'jnlp'
    //        yamlFile 'build.yaml'
    //    }
    //}
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn package'
                }
            }
        }
        stage('Docker Build') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('docker') {
                    sh "docker build -t ${REGISTRY}:${VERSION} ."
                }
            }
        }
        stage('Docker Publish') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('docker') {
                    withDockerRegistry([credentialsId: "${REGISTRY_CREDENTIAL}", url: ""]) {
                        sh "docker push ${REGISTRY}:${VERSION}"
                    }
                }
            }
        }
        stage('Kubernetes Deploy') {
            when {
                environment name: 'DEPLOY', value: 'true'
            }
            steps {
                container('helm') {
                    sh "helm upgrade --install --force --set name=${NAME} --set image.tag=${VERSION} --set domain=${DOMAIN} ${NAME} ./helm"
                }
            }
        }
    }
}
