node{
    
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('prepare enviroment'){
        echo 'initialize all the variables'
        mavenHome = tool name: 'maven' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'docker' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName="3.0"
    }
    
    stage('git code checkout'){
        try{
            echo 'checkout the code from git repository'
            git 'https://github.com/Siva8456/insurance-project.git'
        }
        catch(Exception e){
            echo 'Exception occured in Git Code Checkout Stage'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
            The Jenkins job ${JOB_NAME} has been failed. Request you to please have a look at it immediately by clicking on the below link. 
            ${BUILD_URL}''', subject: 'Job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'mjssivakumar@gmail.com'
        }
    }
    
    stage('Build the Application'){
        echo "Cleaning... Compiling...Testing... Packaging..."
        //sh 'mvn clean package'
        sh "${mavenCMD} clean package"        
    }
    
    stage('publish test reports'){
       publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/insurance/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    }
    
    stage('Containerize the application'){
        echo 'Creating Docker image'
        sh "${dockerCMD} build -t sivakumar8456/insure-me:${tagName} ."
    }
    
    stage('Pushing it ot the DockerHub'){
        echo 'Pushing the docker image to DockerHub'
       withCredentials([usernamePassword(credentialsId: 'f2cd49e0-e470-42b4-a452-d43f81012019', passwordVariable: 'pass', usernameVariable: 'user')]) {
        sh "${dockerCMD} login -u ${user} -p ${pass}"
        sh "${dockerCMD} push sivakumar8456/insure-me:${tagName}"
        }
        
     stage('Configure and Deploy to the test-server'){
         ansiblePlaybook become: true, credentialsId: '251817f4-6833-4ed7-a572-a8ef0a784ca8', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
     }
        
        
    }
}