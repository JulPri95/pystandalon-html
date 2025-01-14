pipeline {
    agent any
    stages {
        //Start Linting Process
        stage( 'Linting') {
            steps {
                script {
                //testing pipeline
                sh 'echo "pipeline connecting"'
                withCredentials([string(credentialsId: 'GitHubActions_Token', variable: 'GITHUB_ACTIONS_CREDENTIALS')]) {
                        final String url = "https://api.github.com/repos/JulPri95/pystandalon-html/dispatches"
                        final String response = sh(script: "curl --request POST --url '$url' --header 'authorization: Bearer ${env.GITHUB_ACTIONS_CREDENTIALS}' --data '{\"event_type\": \"hello\"}'", returnStdout: true).trim() 
                        echo response
                }
                }
            }
        }
        //Run the python file 'tests' to perform the Unit Testing. If it fails, consider the stage a success anyway and move on to next stage
        stage( 'Unit Testing' ) {
            steps {
                script {
                    try {
                        //Make sure all of the necessary libraries and plug-ins are installed
                        sh 'pip3 install pytest'
                        //Get report
                        sh 'python3 -m pytest --junit-xml=results.xml'
                        junit 'results.xml'
                    }
                    catch (exc) {
                            echo 'Unit tests failed'
                            currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
        //Expose a release process and deploy in https://test.pypi.org
        stage('Release and Deploy') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'PyPiToken', variable: 'PYPI_TOKEN')]) { 
                        sh 'find $HOME/ -type f -name ".pypirc" -exec sed -i "s~<TestPyPI Token>~$PYPI_TOKEN~g" {} \\;'
                    sh 'pip3 install -r requirements_dev.txt'
                    try {
                        //Release and Deploy
                        sh 'echo "Release and Deploy"'
                        sh 'python3 -m pip install --upgrade build'
                        sh 'python3 -m build'
                        sh 'python3 -m twine upload --skip-existing --repository testpypi dist/* -u __token__ -p $PYPI_TOKEN'
                    }
                    catch (exc) {
                        sh 'echo "Release and Deploy failed"'
                        currentBuild.result = 'FAILURE'
                    }
                    }
                }   
            }
        }
    }
}
