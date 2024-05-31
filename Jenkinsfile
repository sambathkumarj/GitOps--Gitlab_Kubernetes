pipeline{
	
 	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}

		stage('Build') {

			steps {
				sh 'docker build -t sambathkumarj/jenkinargocd:latest .'
			}
		}

		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}

		stage('Push') {

			steps {
				sh 'docker push sambathkumarj/jenkinargocd:latest'
			}
		}
}
