pipeline {
    
    agent none


    stages {

        // Worker Stages

        stage('Build Worker') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2 \
                          --user root \
			  -v $HOME/worker.jar:/run/worker.jar'
                }
            }
            // when {
            //     changeset "**/worker/**"
            // }
            steps {
                echo 'Compiling worker app'
                dir('worker'){
                    sh 'mvn compile'
                }
            }
        }
        stage('Test Worker') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2 \
			              --user root \
                          -v $HOME/worker.jar:/run/worker.jar'
                    
                    // -v /usr/local/bin/docker:/usr/local/bin/docker \
                    // -v /var/run/docker.sock:/var/run/docker.sock \
                    // --user root 

                }
            }
            // when {
            //     changeset "**/worker/**"
            // }
            steps {
                echo 'Running Unit Tests on worker app'
                dir('worker'){
                    sh 'mvn clean test'
                }
            }
        }
        stage('Package Worker') {
            agent {
                docker {
                    image 'maven:3.6.1-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2 \
			              --user root \
                          -v $HOME/worker.jar:/run/worker.jar'

                    // -v /usr/local/bin/docker:/usr/local/bin/docker \
                    // -v /var/run/docker.sock:/var/run/docker.sock \
                    // --user root 

                }
            }
            when {
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app'
                dir('worker'){
                    sh 'mvn package -DskipTests'
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        stage('Docker-Package Worker') {
            agent any
            when {
                branch 'master'
                changeset "**/worker/**"
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://registry.hub.docker.com',, 'dockerlogin') {
                        def workerImage = docker.build("tblandford/lfd261-worker:v${env.BUILD_ID}", './worker')
                        workerImage.push()
                    }
                }
            }
        }

        // Vote stages

        stage('Build Vote') {
            agent {
                docker {
                    image 'tblandford/lfd261-vote:v1'
                    args '-u root --privileged'
                }
            }
            // when {
            //     changeset "**/vote/**"
            // }
            steps {
                echo 'Build vote app'
                dir('vote'){
                    sh 'pip list'
                }
            }
        }
        stage('Test Vote') {
            agent {
                docker {
                    image 'tblandford/lfd261-vote:v1'
                    args '-u root --privileged'
                }
            }
            // when {
            //     changeset "**/vote/**"
            // }
            steps {
                echo 'Running Unit Tests on vote app'
                dir('vote'){
                    // sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }
        stage('Integration Test Vote') {
            agent any
            // when {
            //     changeset "**/vote/**"
            //     branch 'master'
            // }
            steps {
                echo 'Running Integration Tests on vote app'
                dir('vote'){
                    sh './integration_test.sh'
                }
            }
        }
        stage('Docker-Package Vote') {
            agent any
            when {
            //     branch 'master'
                changeset "**/vote/**"
            }
            steps {
                echo 'Packaging vote app with docker'
                script {
                    docker.withRegistry('https://registry.hub.docker.com',, 'dockerlogin') {
                        def workerImage = docker.build("tblandford/lfd261-vote:v${env.BUILD_ID}", './vote')
                        workerImage.push()
                    }
                }
            }
        }

        // Result stages

        stage('Build Result') {
            agent {
                docker {
                    image 'tblandford/lfd261-result:v1'
                    args '-p 3000:3000'
                }
            }
            // when {
            //     changeset "**/result/**"
            // }
            steps {
                echo 'Build result app'
                dir('result'){
                    sh 'npm install'
                }
            }
        }
        stage('Test Result') {
            agent {
                docker {
                    image 'tblandford/lfd261-result:v1'
                    args '-p 3000:3000'
                }
            }
            // when {
            //     changeset "**/result/**"
            // }
            steps {
                echo 'Running Unit Tests on result app'
                dir('result'){
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }
        stage('Docker-Package Result') {
            agent any
            when {
                // branch 'master'
                changeset "**/result/**"
            }
            steps {
                echo 'Packaging worker app with docker'
                script {
                    docker.withRegistry('https://registry.hub.docker.com',, 'dockerlogin') {
                        def workerImage = docker.build("tblandford/lfd261-result:v${env.BUILD_ID}", './result')
                        workerImage.push()
                    }
                }
            }
        }

	    // SonarCube Stages

        stage('Sonarqube') {
	      agent any
     	    //   when {
       		//     branch 'master'
      	    //   }
      	      tools {
        	    jdk "JDK11" // the name you have given the JDK installation in Global Tool Configuration
      	      }

              environment {
                sonarpath = tool 'SonarScanner'
      	      }

      	      steps {
                echo 'Running Sonarqube Analysis..'
                withSonarQubeEnv('sonar-instavote') {
                  sh "${sonarpath}/bin/sonar-scanner -Dproject.settings=sonar-project.properties -Dorg.jenkinsci.plugins.durabletask.BourneShellScript.HEARTBEAT_CHECK_INTERVAL=86400"
                  sh "sleep 30"
                }
              }
              
        }


        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                  // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                  // true = set pipeline to UNSTABLE, false = don't
                  waitForQualityGate abortPipeline: true
              }
            }
        } 

    }
    post {
        always {
            echo 'Build pipeline for worker is complete'
        }
        success {
            slackSend (channel: "instavote-cd", message: "Build success - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|open>)")
        }
        failure {
            slackSend (channel: "instavote-cd", message: "Build failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|open>)")
        }
    }
}
