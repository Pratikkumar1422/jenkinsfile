pipeline{
    agent any
    environment {
        PATH = "/Users/pratikkumar/downloads/apache-maven-3.8.1/bin:$PATH"
    }
    stages{
       /* stage("Pre-steps"){
            steps{
                build "Pre-Steps"
            }
        }*/
        stage("SCM Checkout"){
            steps{
                git 'http://localhost:3000/pratik/Train-ticket.git'
            }
        }
        stage("Build & Test"){
            steps{
                sh "mvn clean package" 
            }    
        }
        stage('Ready for code analysis') {
            options {
                timeout(time: 5, unit: 'MINUTES') 
            }
            steps {
                input(message: "start code quality check from sonarqube?")
            
                sh "docker restart sonarqube"
                sleep 30 //seconds
            }
        }
        stage("code quality check from sonarqube")
        {
            steps{
                script{
                scannerHome = tool 'soanrqube'}
                withSonarQubeEnv('sonarqube'){
                sh "${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=Train-ticket \
                -Dsonar.projectname=Train-ticket \
                -Dsonar.host.url=http://localhost:9000 \
                -Dsonar.login=admin \
                -Dsonar.password=admin123 \
                -Dsonar.sources=src/main/webapp"
                    }
                    junit 'target/surefire-reports/**/*.xml'
                jacoco()
                }
        }
        stage('Ready to Deploy') {
            options {
                timeout(time: 5, unit: 'MINUTES') 
            }
            steps {
                input(message: "Deploy artifacts to nexus?")
            }
        }
        stage("uploading artifacts to nexus")
        {
            steps{
                sh "docker stop sonarqube"
                sh "docker restart nexus2"
                sleep 40 //seconds
                nexusArtifactUploader artifacts: [[artifactId: 'Rail1', classifier: '', file: '/Users/pratikkumar/.jenkins/workspace/Train-ticket/target/Rail-tickets.war', type: 'war']], credentialsId: '39f0c31f-63d7-49c1-97ec-4f76d410a351', groupId: 'capstone', nexusUrl: 'localhost:8087', nexusVersion: 'nexus3', protocol: 'http', repository: 'capstone-artifacts', version: '6.0.0'
            }
            
        }       
        
        stage("Pulling artifacts from nexus")
        {
            steps{
                sh "wget --user=admin --password=password http://localhost:8087/repository/capstone-artifacts/capstone/Rail1/6.0.0/Rail1-6.0.0.war"
                sh "mv Rail1-6.0.0.war Rail-tickets.war"
                sh "cp Rail-tickets.war  /users/pratikkumar/desktop/ansible"
            }
        }
        stage("building a docker image with the war file")
        { 
            steps{
                git 'file:///Users/pratikkumar/desktop/ansible'
                sh "docker build -t simple-devloper ."
            }
        }
        stage("Playbook to build image & start a contaier"){
            steps{
                sh "sudo chmod 744 /Users/pratikkumar/desktop/ansible/invent"
                ansiblePlaybook installation: 'ansible', inventory: 'invent', playbook: '/Users/pratikkumar/desktop/ansible/ansi.yaml'
            }
        }
        
        stage("calling a DB job")
        {
            steps{
                build "Capstone-db-create"
            }
        }
        stage('Ready to Deploy to k8s cluster') {
            options {
                timeout(time: 5, unit: 'MINUTES') 
            }
            steps {
                input(message: "Deploy container to kubernestes cluster?")
            }
        }
        stage("Deploy container to K8s cluster")
        {
            steps{
                sh "docker stop nexus2"
                sh "minikube start"
                sleep 10 //seconds
                sh "minikube image load webapp-capstone"
                git 'file:///Users/pratikkumar/desktop/ansible'
                sh "kubectl create -f deployment.yaml"
            }
        }
        
}
}