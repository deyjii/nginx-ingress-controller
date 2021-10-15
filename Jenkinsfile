@Library('addons') _

pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
        disableConcurrentBuilds()
        ansiColor('xterm')
        timestamps()
        parallelsAlwaysFailFast()
    }
    environment {
        Name = "ingress-nginx"
    }

    stages {
        stage('QA') {
            when {
                branch 'qa'
                beforeAgent true
            }
            stages {
                stage('Pre work') {
                    steps {
                        script {
                            setEKS()
                            setHelm()
                        }
                        sh "helm version"
                    }
                }
                stage('Validate Helm') {
                    steps {
                        sh "helm lint ./${NAME} --values ./values.yaml"
                    }
                }
                stage('Deploy') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'AD_JOIN_DETAILS', passwordVariable: 'PASS', usernameVariable: 'USER')
                        ]) {
                            sh "aws eks --region eu-west-1 update-kubeconfig --name <<cluster-name>>"
                            sh "kubectl apply -f ./namespace.yaml"
                            sh "helm upgrade --debug \
                            --namespace nginx \
                            --values ./values.yaml \
                            --set env.LDAP_BIND_DN=${USER} \
                            --set env.LDAP_BIND_PWD=${PASS} \
                            --atomic \
                            --install \
                            --wait ${NAME} ./${NAME}"
                        }
                    }
                }
            }
            post {
                always {
                    cleanWs()
                }
            }
        }
        stage('prod') {
            when {
                branch 'master'
                beforeAgent true
            }
            stages {
                stage('Pre work') {
                    steps {
                        script {
                            setEKS()
                            setHelm()
                        }
                        sh "helm version"
                    }
                }
                stage('Validate Helm') {
                    steps {
                        sh "helm lint ./${NAME} --values ./values.yaml"
                    }
                }
                stage('Deploy') {
                    steps {
                        withCredentials([usernamePassword(credentialsId: 'AD_JOIN_DETAILS', passwordVariable: 'PASS', usernameVariable: 'USER')
                        ]) {
                            withAWS(region: 'eu-west-1', role: 'arn:aws:iam::164135465533:role/Jenkins') {
                            sh "aws eks --region eu-west-1 update-kubeconfig --name <<cluster-name>>"
                            sh "kubectl apply -f ./namespace.yaml"
                            sh "helm upgrade \
                            --namespace nginx \
                            --values ./values.yaml \
                            --set env.LDAP_BIND_DN=${USER} \
                            --set env.LDAP_BIND_PWD=${PASS} \
                            --atomic \
                            --install \
                            --wait ${NAME} ./${NAME}"
                        }
                     }
                 }
            }
        }
        post {
            always {
                cleanWs()
            }
        }
    }
    }
}