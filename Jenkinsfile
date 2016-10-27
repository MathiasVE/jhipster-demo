node {
  
  // note - the postgresql will be linked to the application sharing an independent network unexposed allowing to reuse port 5432
  def PORT="9000"
  if (env.BRANCH_NAME == "testing") {
    PORT="9100"
  } else {
    PORT="80"
  }

  stage 'undeploy current'
  if(env.BRANCH_NAME != "master") {
      sh "if [ -f /src/main/docker/docker/app.yml ]; then export ABC_PORT=$PORT; export ABC_ENV=${env.BRANCH_NAME}; docker-compose -p jhipster_${env.BRANCH_NAME} -f src/main/docker/app.yml down; fi"
  }
  
  stage 'fetch code'
  // Fetch the from my repository
  git url: 'https://github.com/MathiasVE/jhipster-demo', branch: env.BRANCH_NAME
  

  
  stage 'pull container images'
  def maven = docker.image('maven:3.3.3-jdk-8')
  maven.pull()
  def jhipster = docker.image('jhipster/jhipster')
  jhipster.pull() 
  
  maven.inside {
    stage 'Install'
    sh 'mvn -B clean install'
    stage 'Test'
    sh 'mvn -B test'
  }
  
  jhipster.inside {
    stage 'Prepare js/css'
    // Due to a bug in npm we need to keep installing until it succeeds.
    // Many dependencies cause a stack overflow, running the install again 
    // will pick up the installation where it stopped before due to the error.
    int tryCount = 3;
    boolean installed = false;
    
    while(!installed && tryCount > 0) {
      try {
        sh 'npm install --no-bin-links'
        installed = true;
      } catch(Exception e) {
      }
      tryCount--;
    }
    // Bad dependency declared so we take the latest version ignoring the unsafe warnings
    sh 'npm install --unsafe-perm node-sass --no-bin-links'
    stage 'Test js/css'
    // sh 'export PHANTOMJS_BIN="$(pwd)/node_modules/phantomjs-prebuild/bin/phantomjs"; gulp test'
    stage 'Build js/css'
    sh 'export PHANTOMJS_BIN="$(pwd)/node_modules/phantomjs-prebuild/bin/phantomjs"; gulp build'
    stage 'Build image'
    sh 'export PHANTOMJS_BIN="$(pwd)/node_modules/phantomjs-prebuild/bin/phantomjs"; ./mvnw package -DskipTests -Pprod docker:build'
  }
  
  stage 'Deploy'
  
  if(env.BRANCH_NAME != "master") {
    sh "export ABC_PORT=$PORT; export ABC_ENV=${env.BRANCH_NAME}; docker-compose -p jhipster_${env.BRANCH_NAME} -f src/main/docker/app.yml up -d"
  }
}

