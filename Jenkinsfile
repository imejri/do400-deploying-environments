
/* An uber-JAR—also known as a fat JAR or JAR with dependencies—is a JAR file that contains not only a Java program, but embeds its dependencies as well. This means that the JAR functions as an “all-in-one” distribution of the software, without needing any other Java code. (You still need a Java runtime, and an underlying operating system, of course.)*/

pipeline {
    agent {
        node {
            label 'maven'
        } 
    } //agent

    environment {
                RHT_OCP4_DEV_USER = 'uchwjb' 
                DEPLOYMENT_STAGE = 'shopping-cart-stage' 
                DEPLOYMENT_PRODUCTION = 'shopping-cart-production'
            }

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
        
        stage ('Build Image'){
            environment {
                QUAY = credentials ('QUAY_USER')
            }
            steps {
                sh """
                    ./mvnw quarkus:add-extension \
                -Dextensions="kubernetes,container-image-jib"
                """

                sh """
                    ./mvnw package -DskipTests \
                    -Dquarkus.jib.base-jvm-image=quay.io/redhattraining/do400-java-alpine-openjdk11-jre:latest \
                    -Dquarkus.container-image.build=true \
                    -Dquarkus.container-image.registry=quay.io \
                    -Dquarkus.container-image.group="$QUAY_USR" \
                    -Dquarkus.container-image.name=do400-deploying-environments \
                    -Dquarkus.container-image.username="$QUAY_USR" \
                    -Dquarkus.container-image.password="$QUAY_PSW" \
                    -Dquarkus.container-image.tag=build-${BUILD_NUMBER} \
                    -Dquarkus.container-image.push=true
                """
            }
        }
        stage('Deployment to STAGE environment'){
            environment {
                APP_NAMESPACE = "${env.DEPLOYMENT_STAGE}"
                QUAY = credentials('QUAY_USER')
            }
            steps {
                sh """
                oc set image \
                deployment ${DEPLOYMENT_STAGE} \
                shopping-cart-stage=quay.io/${QUAY_USR}/do400-deploying-environments:build-${BUILD_NUMBER} \
                -n ${APP_NAMESPACE} --record
                
                """

            } // steps
        } // stage
    } // stages
} // pipeline