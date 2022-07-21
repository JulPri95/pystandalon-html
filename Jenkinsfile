pipeline {
    agent any
    stages {
        //Start Linting Process
        stage( 'Linting') {
            steps {
                script {
                //testing pipeline
                sh 'echo "pipeline connecting"'
                withCredentials([file(credentialsId: 'GitHubActions_Token2', variable: 'GITHUB_ACTIONS_CREDENTIALS')]) {
                        final String url = "https://api.github.com/repos/JulPri95/pystandalon-html/dispatches"
                        final String response = sh(script: "curl --request POST --url '$url' --header 'authorization: Bearer ${env.GITHUB_ACTIONS_CREDENTIALS}' --data '{'event_type': 'hello'}'", returnStdout: true).trim() 
                        echo response
                }
                //Install requirements
                //sh 'pip3 install -r requirements_dev.txt'
                //Move to the correct directory
                //sh 'cd $WORKSPACE/.github/workflows'
                //sh 'ls'
                //Lint
                //sh 'python3 pylint.yml'
                }
            }
        }
        //Run the python file 'tests' to perform the Unit Testing. If it fails, consider the stage a success anyway and move on to next stage
        stage( 'Unit Testing' ) {
            steps {
                script {
                    try {
                        //Move to the correct directory
                        sh 'cd $WORKSPACE/tests'
                        //Make sure all of the necessary libraries and plug-ins are installed
                        sh 'sudo apt install python3-pip'
                        sh 'pip3 install -r requirements_dev.txt'
                        //sh 'sudo python3 -m pip install pytest'
                        //Execute test in verbose format
                        sh 'pytest test_cli.py -v'
                        sh 'pytest test_converter.py -v'
                        //Get report
                        sh 'pytest --junit-xml=results.xml'
                    }
                    catch (exc) {
                            echo 'Unit tests failed'
                    }
                }
            }
        }
        //Expose a release process and deploy in https://test.pypi.org
        stage('Release and Deploy') {
            steps {
                script {
                    sh 'pip3 install -r requirements_dev.txt'
                    try {
                        //Release and Deploy
                        sh 'echo "Release and Deploy"'
                        sh 'python3 -m twine upload --repository testpypi dist/*'
                    }
                    catch (exc) {
                        sh 'echo "Release and Deploy failed"'
                    }
                }   
            }
        }
    }
}
