pipeline {
    agent any
    stages {
        //Build the Dockerfile image
        stage( 'Docker Image Build') {
            steps {
                //Build the Docker image with the new commits
                sh 'docker build -t pym .'
            }
        }
        //Run the python file 'tests' to perform the Unit Testing. If it fails, consider the stage a success anyway and move on to next stage
        stage( 'Unit Testing' ) {
            steps {
                script {
                    try {
                        //Move to the correct directory
                        sh 'cd /$WORKSPACE/'
                        //Make sure all of the necessary libraries and plug-ins are installed
                        sh 'sudo apt install python3-pip'
                        sh 'sudo python3 -m pip install pytest'
                        //Execute test in verbose format
                        sh 'pytest tests.py -v'
                    }
                    catch (exc) {
                            echo 'Unit tests failed'
                    }
                }
            }
        }
        //Using the sonar scan plug-in, execute SonarCloud analysis on the project, including the coverage which is createad in this step.
        //Return 'Ok' if the code passes the Quality Gate and move on to Docker Build. Otherwise, abort because of timeout and don't run.
        stage('SonarScan w/ Coverage') {
            environment {
             //Set variable for the SonarQube Sanner server being run on Jenkins through a plug-in
             SCANNER_HOME = tool 'SonarQubeScanner'
            }
            steps {
                script {
                    //Call the credentials that are saved in the Jenkins plug-in in encrypted formatto avoid including sensitive 
                    //information in the code
                    withCredentials([
                        string(
                            credentialsId: 'project-key',
                            variable: 'PROJECT_NAME'),
                        string(
                            credentialsId: 'organization-key',
                            variable: 'ORGANIZATION')
                        ]) {
                        //Make sure all of the necessary libraries and programs are installed
                        sh 'sudo python3 -m pip install coverage'
                        //Run a coverage command for the pytest and pipe it into a coverage report. Pipe that report into an xml which
                        //will be interpreted by SonarCloud
                        sh 'coverage run -m pytest /var/lib/jenkins/workspace/FinalProject/tests.py -v | coverage report | coverage xml'
                        //Call SonarCloud with the credentials in order to connect to the right project, and with the correct path for the
                        //coverage report to be imported and interpreted
                        withSonarQubeEnv('SonarCloud') {
                                sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.organization=$ORGANIZATION \
                                -Dsonar.java.binaries=build/classes/java/ \
                                -Dsonar.projectKey=$PROJECT_NAME \
                                -Dsonar.python.coverage.reportPaths=$WORKSPACE/coverage.xml'''
                        }
                        //Set a timeout for the Quality gate and run the function inside a variable to use it in the next stage
                        timeout(time: 5, unit: 'MINUTES'){
                            env.QG=waitForQualityGate().status
                        }
                   }
                }
            }
        }
        //Run the docker image in port 8000 if the Quality Gate passes, if the container already has an active instance, 
        //remove it, using try/catch method
        stage('Docker Run') {
            when {
                //Only run if Quality Gate is Ok
                environment name: 'QG', value: 'OK'
            }
            environment {
                //Assign a variable for the ID of the active container with the same command line as the one that was built
                CONTAINER_ID = sh(script: "docker ps | grep ash | awk '{ print \$1 }'", returnStdout: true).trim()
            }
            steps {
                script {
                    try {
                        //Remove the active container and run the new image. If there is no active container this step fail and go to catch
                        sh 'echo "CONTAINER_ID = $CONTAINER_ID"'
                        sh 'docker rm -f $CONTAINER_ID'
                        sh 'docker run -d -p 8000:8000 pym'
                    }
                    catch (exc) {
                        //Run the new image
                        sh 'echo "Docker image Pym is not running; will run now"'
                        sh 'sudo docker run -d -p 8000:8000 pym'
                    }
                }   
            }
        }
    }
}
