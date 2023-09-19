pipeline {
    agent any
    stages {
        stage('Clone repo') {
            steps {
                git url: 'https://github.com/malekalghraba/banking', branch: 'main'
            }
        }

   stage('Build frontend') {
            steps {
                dir('FRONTEND') {
                    sh '/usr/bin/npm install'
                    sh '/usr/bin/npm run build'
                }} }
        
         stage('Run SonarQube Analysis') {
    steps {
        dir('FRONTEND') {
            script {
                def scannerHome = tool 'SonarQube'

                withSonarQubeEnv('SonarQube') {
                   sh "${scannerHome}/bin/sonar-scanner " }}}}}

    }
    stage('Test frontend') {
            steps {
                dir('FRONTEND') {
                    sh 'npm run test'
                }}} 

         stage('build backend'){
            parallel{ 
                stage('Compile ') {
            steps {
                dir('BACKEND') {
                    sh 'mvn clean compile'
                }
            }
        }
                
                
                stage('checkstyle '){
            steps{ dir('BACKEND'){ script{
        def mvnHome = tool 'maven'
          sh "${mvnHome}/bin/mvn -batch-mode -V -U -e checkstyle:checkstyle"
          def checkstyle = scanForIssues tool: [$class: 'CheckStyle'], pattern: '/target/checkstyle-result.xml'
        publishIssues issues:[checkstyle], unstableTotalAll:3000
        }}}}
        
        }}
        stage('Unit Test backend') { 
            steps {dir('BACKEND') {
               
                    sh 'mvn test'
                }}
      
        }
        stage('Code Quality Analysis'){
              parallel{
                stage('pmd '){
            steps{ dir('BACKEND'){ script{
                 def mvnHome = tool 'maven'
            
        sh "${mvnHome}/bin/mvn -batch-mode -V -U -e checkstyle:checkstyle pmd:pmd "
                
        def pmd = scanForIssues tool: [$class: 'Pmd'], pattern: '/target/pmd.xml'
        publishIssues issues:[pmd], unstableTotalAll:100 }}}}
                  stage("findbug"){
                       steps{ dir('BACKEND'){ script{
                 def mvnHome = tool 'maven'
                  sh "${mvnHome}/bin/mvn -batch-mode -V -U -e checkstyle:checkstyle findbugs:findbugs "
                            def findbugs = scanForIssues tool: [$class: 'FindBugs'], pattern: '/target/findbugsXml.xml'
        publishIssues issues:[findbugs], unstableTotalAll:200}}}}
              
              stage('cpd '){
            steps{ dir('BACKEND'){ script{
                 def mvnHome = tool 'maven'
            
        sh "${mvnHome}/bin/mvn -batch-mode -V -U -e checkstyle:checkstyle pmd:cpd "
                
        def cpd = scanForIssues tool: [$class: 'Cpd'], pattern: '/target/cpd.xml'
        publishIssues issues:[cpd] }}}}
                
              
        stage('SonarQube Analysis') {
                steps {
        dir('BACKEND') { script{
    def mvn = tool 'maven';
    withSonarQubeEnv() {
      sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=back -Dsonar.projectName='back'"
    }
  }}
              }} }}

 stage("Deploy Artifact To Nexus") {
            steps { dir('BACKEND'){
                script {
                pom = readMavenPom file: 'pom.xml'
                   echo "${pom.artifactId}-${pom.version}.${pom.packaging}"
                   sh "mvn deploy:deploy-file  -DskipTests=true -DgroupId=${pom.groupId} -DartifactId=${pom.artifactId} -Dversion=${pom.version}  -DgeneratePom=true -Dpackaging=${pom.packaging}  -DrepositoryId=deploymentRepo -Durl=http://localhost:8081/repository/maven-snapshots/ -Dfile=target/${pom.artifactId}-${pom.version}.${pom.packaging}"
                }
            }
        }}
/*        
 stage ('Analysis') { steps{ dir('BACKEND'){ script{
        def mvnHome = tool 'maven'
        def maven = scanForIssues tool: [$class: 'MavenConsole']
        publishIssues issues:[maven]
      
    }}}} */
           
        stage('Build and tag Images') {
            steps {
                  sh "docker login -u malekghraba -p fra10malek"
                sh 'docker-compose build'
                sh'docker tag ebank-fullstack_frontend:latest malekghraba/frontend:1'
                 sh'docker tag ebank-fullstack_backend:latest malekghraba/backend:1'
                sh'docker tag ebank-fullstack_mysql:latest malekghraba/mysql:1'

            }
        }
          stage('push Images to dockerhub') {
            steps {
               
            sh'docker push malekghraba/frontend:1'
            sh'docker push malekghraba/backend:1'
            sh'docker push malekghraba/mysql:1'
            }}

    stage('Start Minikube') {
            steps {
                script {
                    // Start Minikube with the Docker driver
                    sh 'minikube start --driver=docker'
                }
            }
        }

        stage('Deploy to Minikube') {
            steps { 
             
             script{
                // Apply Kubernetes Deployment and Service manifests.
            
                sh 'kubectl create -f mysql-deployment.yaml'
                sh 'kubectl create -f backend-deployment.yaml'
                sh 'kubectl create -f frontend-deployment.yaml'
                sh 'kubectl create -f mysql-service.yaml'
                sh 'kubectl create -f frontend-service.yaml'
        }}}
         stage('Stop Minikube') {
            steps {
                script {
                    // Stop Minikube after the pipeline is complete
                    sh 'minikube stop'
                    sh 'minikube delete'
                }
            }
        }
   
}
    }
