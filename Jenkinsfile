def projectName = 'app'
def version = "0.0.${currentBuild.number}"
def dockerImageTag = "${projectName}:${version}"
def mysqlName = 'mysql'
def dockerImageTag2 = "${mysqlName}:${version}"


pipeline {
    agent none

    stages {
       
        stage('Build') {
            agent any
             // this stage also builds and tests the Java project using Maven
             steps {
//                sh "docker build -f Dockerfile-mysql -t emps/mysql ."
//                sh "docker build -f Dockerfile-app -t emps/app ."
//                sh "docker run --name mysql -d -p 3306:3306 emps/mysql"
//                sh "sleep 10"
//                sh "docker run --name app -d --link mysql:mysql emps/app"
                 
                sh "docker build -f Dockerfile-mysql -t ${dockerImageTag2} ."
                sh "docker build -f Dockerfile-app -t ${dockerImageTag} ."
                sh "docker run --name mysql -d -p 3306:3306 emps/mysql"
                sh "sleep 10"
                sh "docker run --name app -d --link mysql:mysql ${dockerImageTag}"
             }
         }
        
        
        stage('Test') { 
            agent {
                docker {
                  image 'maven:3.8.1-adoptopenjdk-11'
                }
            }
            steps {
                sh 'mvn -DskipTests -Pprod clean package'
                sh 'mvn -Pprod test'
                stash(includes: 'target/*.jar', name: 'jar')
                archiveArtifacts 'target/*.jar'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml' 
                }
            }
        }
//         stage('Deploy Container To Openshift') {
//               agent any
//               steps {
//                 sh "oc login https://localhost:8443 --username admin --password admin --insecure-skip-tls-verify=true"
//                 sh "oc project ${projectName} || oc new-project ${projectName}"
//                 sh "oc delete all --selector app=${projectName} || echo 'Unable to delete all previous openshift resources'"
//                 sh "oc new-app ${dockerImageTag} -l version=${version}"
//                 sh "oc expose svc/${projectName}"
//               }
//             }
        
       stage('Deploy') {
          agent any
          steps {
            sh "oc login https://devopsapac33.conygre.com:8443 --username admin --password admin --insecure-skip-tls-verify=true"
            sh "oc project ${projectName} || oc new-project ${projectName}"
            sh "oc delete all --selector app=${projectName} || echo 'Unable to delete all previous openshift resources'"
            sh "oc delete all --selector app=${mysqlName} || echo 'Unable to delete all previous openshift resources'"
            sh "oc new-app ${dockerImageTag2}"
            sh "oc new-app ${dockerImageTag}"
            sh "oc expose svc/${projectName}"
          }
        }
    }
}
