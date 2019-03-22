pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  tools {
    nodejs 'node-latest'
  }
  parameters {
    string(name: 'IMAGE_REPO_NAME', defaultValue: 'jamessmith52963/basic-react', description: '')
    string(name: 'LATEST_BUILD_TAG', defaultValue: 'build-latest', description: '')
    string(name: 'DOCKER_COMPOSE_FILENAME', defaultValue: 'docker-compose.yml', description: '')
    string(name: 'DOCKER_STACK_NAME', defaultValue: 'react_stack', description: '')
    booleanParam(name: 'NPM_RUN_TEST', defaultValue: true, description: '')
    booleanParam(name: 'PUSH_DOCKER_IMAGES', defaultValue: false, description: '')
    booleanParam(name: 'DOCKER_STACK_RM', defaultValue: false, description: 'Remove previous stack.  This is required if you have updated any secrets or configs as these cannot be updated. ')
	booleanParam(name: 'DOCKER_LAST_IMAGE_RM', defaultValue: true, description: 'Removes docker image tag created in previous build')
  }
  stages {
    stage('npm install'){
      steps{
	  	echo "##################################################"
		echo "BUILD_NUMBER" :: $BUILD_NUMBER
		echo "BUILD_ID" :: $BUILD_ID
		echo "BUILD_DISPLAY_NAME" :: $BUILD_DISPLAY_NAME
		echo "JOB_NAME" :: $JOB_NAME
		echo "JOB_BASE_NAME" :: $JOB_BASE_NAME
		echo "BUILD_TAG" :: $BUILD_TAG
		echo "EXECUTOR_NUMBER" :: $EXECUTOR_NUMBER
		echo "NODE_NAME" :: $NODE_NAME
		echo "NODE_LABELS" :: $NODE_LABELS
		echo "WORKSPACE" :: $WORKSPACE
		echo "JENKINS_HOME" :: $JENKINS_HOME
		echo "JENKINS_URL" :: $JENKINS_URL
		echo "BUILD_URL" ::$BUILD_URL
		echo "JOB_URL" :: $JOB_URL
		echo "##################################################"
         sh "npm install"
      }
    }
    stage('npm test'){
	    when{
		    expression{
			    return params.NPM_RUN_TEST
		    }
	    }
	steps{
	  sh "npm test -- --coverage"	
	}
    }
    stage('npm build'){
      steps{
        sh "npm run build"
      }
    }
    stage('docker build'){
      environment {
        COMMIT_TAG = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
        BUILD_IMAGE_REPO_TAG = "${params.IMAGE_REPO_NAME}:${env.BUILD_TAG}"
      }
      steps{
        sh "docker build . -t $BUILD_IMAGE_REPO_TAG"
//      sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_REPO_NAME}:$COMMIT_TAG"
//      sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_REPO_NAME}:${readJSON(file: 'package.json').version}"
//      sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_REPO_NAME}:${params.LATEST_BUILD_TAG}"
//      sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_REPO_NAME}:$BRANCH_NAME-latest"
      }
    }
    stage('docker push'){
      when{
        expression {
          return params.PUSH_DOCKER_IMAGES
        }
      }
      environment {
        COMMIT_TAG = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
        BUILD_IMAGE_REPO_TAG = "${params.IMAGE_REPO_NAME}:${env.BUILD_TAG}"
      }
      steps{
        sh "docker push $BUILD_IMAGE_REPO_TAG"
        sh "docker push ${params.IMAGE_REPO_NAME}:$COMMIT_TAG"
        sh "docker push ${params.IMAGE_REPO_NAME}:${readJSON(file: 'package.json').version}"
        sh "docker push ${params.IMAGE_REPO_NAME}:${params.LATEST_BUILD_TAG}"
        sh "docker push ${params.IMAGE_REPO_NAME}:$BRANCH_NAME-latest"
      }
    }
    stage('Remove Previous Stack'){
      when{
        expression {
	  return params.DOCKER_STACK_RM
        }
      }
      steps{
        sh "docker stack rm ${params.DOCKER_STACK_NAME}"			
      }
    }
    stage('Docker Stack Deploy'){
      steps{
        sh "docker stack deploy -c ${params.DOCKER_COMPOSE_FILENAME} ${params.DOCKER_STACK_NAME}"
      }
    }

	stage('Remove last docker build image tag'){
	  environment {
		LAST_BUILD_TAG = "${params.IMAGE_REPO_NAME}:${env.BUILD_TAG}"
		LAST_BUILD_ID = sh(returnStdout: true, script: 'readlink /var/jenkins_home/jobs/DJ_multibranch_pipeline_1/branches/master/builds/lastStableBuild')
	  }
	  when{
		expression {
		  return params.DOCKER_LAST_IMAGE_RM
		}
	  }
	  steps{
		echo "LAST_BUILD_TAG = $LAST_BUILD_TAG"
		echo "LAST_BUILD_ID = $LAST_BUILD_ID"
		echo "docker rmi ${params.IMAGE_REPO_NAME}:${env.BUILD_TAG}"
		echo "readlink = "
		sh "readlink /var/jenkins_home/jobs/DJ_multibranch_pipeline_1/branches/master/builds/lastStableBuild"
//		sh "docker rmi ${params.IMAGE_REPO_NAME}:${env.BUILD_TAG}"
	  }
	}
  }
  post {
    always {
      sh 'echo "This will always run"'
    }
  }
}
