
/* An uber-JAR—also known as a fat JAR or JAR with dependencies—is a JAR file that contains not only a Java program, but embeds its dependencies as well. This means that the JAR functions as an “all-in-one” distribution of the software, without needing any other Java code. (You still need a Java runtime, and an underlying operating system, of course.)*/

pipeline {
    agent {
        node {
            label 'maven'
        } 
    } //agent

    stages {
        stage('Tests') {
            steps {
                sh './mvnw clean test'
        } 
    }
        stage('Package') {
            steps {
                sh '''
                    ./mvnw package -DskipTests \
                    -Dquarkus.package.type=uber-jar
                '''
                archiveArtifacts 'target/*.jar'
            } // steps
        } // stage
    } // stages
} // pipeline