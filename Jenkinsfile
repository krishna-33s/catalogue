pipeline {
    agent {
        node {
            label "roboshop"
        }
    }
    environment {
        version = ""
        app_name = "catalogue"
    }
    options {
        // disableConcurrentBuilds()
        timeout(time: 5, unit: 'MINUTES')
    }
    // parameters {
    //     string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
    //     text(name: 'BIOGRAPHY', defaultValue: '', description: 'Enter some information about the person')
    //     booleanParam(name: 'DEPLOY', defaultValue: false, description: 'Deploy the application')
    //     choice(name: 'CHOICE', choices: ['One', 'Two', 'Three'], description: 'Pick something')
    //     password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'Enter a password')
    // }
    stages {
        stage("read the version in package.json") {
            steps {
                script {
                    def packagejson = readJSON file: 'package.json'

                    version = packagejson.version
                    echo "Version is ${version}"
                }
            }
        }

        stage("install dependencies") {
            steps {
                script{
                    sh """
                        npm install
                    """
                }
            }
        }

        stage("build docker image") {
            steps {
                script{
                    sh """
                        docker build -t ${app_name}:${version} .
                    """
                }
            }
        }
    post {
        always {
            echo "docker image is built"
        }
        success {
            echo "docker image built image successfully completed"
        }
        failure {
            echo "building image failed"
        }
    }
    }
}