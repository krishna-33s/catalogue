pipeline {
    agent {
        node {
            label "roboshop"
        }
    }
    environment {
        version = ""
        app_name = "catalogue"
        region = "us-east-1"
        id = "220719767845"
    }
    options {
        // disableConcurrentBuilds()
        timeout(time: 6, unit: 'MINUTES')
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
        
        stage('Dependabot Alerts Check') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GH_TOKEN')]) {
                    script {
                        def owner = 'krishna-33s'
                        def repo = 'catalogue'

                        
                        def response = sh(
                            script: """
                                curl -s -w "HTTPSTATUS:%{http_code}" \
                                -H "Authorization: Bearer ${GH_TOKEN}" \
                                -H "Accept: application/vnd.github+json" \
                                "https://api.github.com/repos/${owner}/${repo}/dependabot/alerts?severity=high,critical&state=open&per_page=100"
                            """,
                            returnStdout: true
                        ).trim()

                        def parts = response.tokenize('\n')
                        def httpStatus = parts[-1].trim()
                        def body = parts[0..-2].join('\n')

                        if (httpStatus != '200') {
                            error("GitHub API call failed. HTTP status: ${httpStatus}. Response: ${body}")
                        }

                        def alerts = readJSON text: body

                        if (alerts.size() == 0) {
                            echo "✅ No open high or critical severity Dependabot alerts found.pipeline continues"
                        }
                        else {
                            echo "\n⚠️  Found ${alerts.size()} high and critical Dependabot alert(s):"
                                alerts.each { alert ->
                                def pkg = alert.security_vulnerability?.package?.name ?: 'unknown'
                                def ghsa = alert.security_advisory?.ghsa_id ?: 'unknown'
                                def summaryText = alert.security_advisory?.summary ?: 'No summary'
                                def fixedin = alert.security_advisory?.fixed_in ?: 'No fix available'
                                echo "Package: ${pkg}, GHSA: ${ghsa}, Summary: ${summaryText}, Fixed in: ${fixedin}"
                                }
                                error "pipeline failed: $(alerts.size()) high or critical Dependabot alert(s) found. Please address them before proceeding."
                        }
                    }
                }
            }       
        }       
        stage("build docker image") {
            steps {
                script{
                    withAWS(credentials: 'aws-creds', region: "${region}") {

                    sh """
                        aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${id}.dkr.ecr.${region}.amazonaws.com
                        docker build -t ${id}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${version} .
                        docker push ${id}.dkr.ecr.${region}.amazonaws.com/roboshop/catalogue:${version}

                    """
                    }
                }
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