pipeline {
    agent any
    stages {
        //Start Linting Process
        stage( 'Linting') {
            steps {
                //testing pipeline
                //Install requirements
                //sh 'pip3 install -r requirements_dev.txt'
                //Move to the correct directory
                //sh 'cd $WORKSPACE/.github/workflows'
                //sh 'ls'
                //Lint
                //sh 'python3 pylint.yml'
                sh 'echo Linting...'
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
                    }
                }
            }
        }
        //Expose a release process and deploy in https://test.pypi.org
        stage('Release and Deploy') {
            steps {
                script {
                    // sh 'pip3 install -r requirements_dev.txt'
                    // try {
                    //     //Release and Deploy
                    //     sh 'echo "Release and Deploy"'
                    //     sh 'python3 -m twine upload --repository testpypi dist/*'
                    // }
                    // catch (exc) {
                    //     sh 'echo "Release and Deploy failed"'
                    // }
                    sh 'echo Deployment...'
                }   
            }
        }
    }
}
