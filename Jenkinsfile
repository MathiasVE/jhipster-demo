node {
  stage 'fetch code'
  git 'https://github.com/MathiasVE/jhipster-demo'

  def maven = docker.image('maven:3.3.3-jdk-8')
  maven.pull()
  def jhipster = docker.image('jhipster/jhipster')
  jhipster.pull() 
  
  maven.inside {
    stage 'Install'
    sh 'mvn -B clean install'
    if(env.BRANCH_NAME == "testing") {
      stage 'Test'
      sh 'mvn -B test'
    }
  }

  jhipster.inside {
    stage 'Prepare js/css'
    // Due to a bug in npm we need to keep installing until it succeeds
    sh 'NPM=-1; while [ ${NPM} -ne 0 ]; do npm install --no-bin-links; NPM=$?; done'
    // Bad dependency declared so we take the latest version ignoring the unsafe warnings
    sh 'npm install --unsafe-perm node-sass --no-bin-links'
	stage 'Test js/css'
	sh 'gulp test'
	stage 'Build js/css'
    sh 'gulp build'
  }
 
  maven.inside { 
    stage 'Build docker image'
    if(env.BRANCH_NAME == "development") {
      sh './mvnw package -DskipTests -Pdev docker:build'
    } else {
      sh './mvnw package -DskipTests -Pprod docker:build'
    }
  }

  stage 'Deploy'
  if(env.BRANCH_NAME == "development") {
    sh 'sed -i "s/8080:8080/5000:8080/g" src/main/docker/app.yml'
    sh 'sed -i "s/5432:5432/5001:5432/g" src/main/docker/postgresql.yml'
  } else if (env.BRANCH_NAME == "testing") {
    sh 'sed -i "s/8080:8080/5100:8080/g" src/main/docker/app.yml'
    sh 'sed -i "s/5432:5432/5101:5432/g" src/main/docker/postgresql.yml'
  } else {
    sh 'sed -i "s/8080:8080/80:8080/g" src/main/docker/app.yml'
  }
  if(env.BRANCH_NAME != "master") {
    sh "docker-compose -p jhipster_${env.BRANCH_NAME} -f src/main/docker/app.yml up -d"
  }

}
