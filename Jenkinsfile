pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    environment {
        DOCKERHUB_USER = 'VOTRE_PSEUDO_DOCKERHUB'
        LOCAL_IMAGE    = 'backend-app:latest'
        REMOTE_IMAGE   = "${DOCKERHUB_USER}/backend-app:latest"
        DB_NAME        = 'gestion_projets'
        DB_PASSWORD    = 'root'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                dir('backend') {
                    sh 'mvn compile'
                }
            }
        }

        stage('Package Maven') {
            steps {
                dir('backend') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('backend') {
                    sh 'docker build -t $LOCAL_IMAGE .'
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker tag $LOCAL_IMAGE $REMOTE_IMAGE
                        docker push $REMOTE_IMAGE
                    '''
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                sh '''
                    docker network inspect app-net >/dev/null 2>&1 || docker network create app-net
                    if [ -z "$(docker ps -aq -f name=^mysql$)" ]; then
                        docker run -d --name mysql --network app-net \
                            -e MYSQL_ROOT_PASSWORD=$DB_PASSWORD \
                            -e MYSQL_DATABASE=$DB_NAME \
                            -v mysql-data:/var/lib/mysql \
                            mysql:8.0
                    elif [ -z "$(docker ps -q -f name=^mysql$)" ]; then
                        docker start mysql
                    fi
                    echo "Attente de MySQL..."
                    until docker exec mysql mysqladmin ping -h localhost -uroot -p$DB_PASSWORD --silent; do
                        sleep 3
                    done
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    docker stop backend-app || true
                    docker rm backend-app || true
                    docker pull $REMOTE_IMAGE
                    docker run -d --name backend-app --network app-net -p 8081:8080 \
                        -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/$DB_NAME \
                        -e SPRING_DATASOURCE_USERNAME=root \
                        -e SPRING_DATASOURCE_PASSWORD=$DB_PASSWORD \
                        $REMOTE_IMAGE
                '''
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    sleep 20
                    docker ps
                    docker logs backend-app
                '''
            }
        }
    }
}pipeline {
    agent any

    tools {
        jdk 'JAVA_HOME'
        maven 'M2_HOME'
    }

    environment {
        DOCKERHUB_USER = 'VOTRE_PSEUDO_DOCKERHUB'
        LOCAL_IMAGE    = 'backend-app:latest'
        REMOTE_IMAGE   = "${DOCKERHUB_USER}/backend-app:latest"
        DB_NAME        = 'gestion_projets'
        DB_PASSWORD    = 'root'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                dir('backend') {
                    sh 'mvn compile'
                }
            }
        }

        stage('Package Maven') {
            steps {
                dir('backend') {
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('backend') {
                    sh 'docker build -t $LOCAL_IMAGE .'
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker tag $LOCAL_IMAGE $REMOTE_IMAGE
                        docker push $REMOTE_IMAGE
                    '''
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                sh '''
                    docker network inspect app-net >/dev/null 2>&1 || docker network create app-net
                    if [ -z "$(docker ps -aq -f name=^mysql$)" ]; then
                        docker run -d --name mysql --network app-net \
                            -e MYSQL_ROOT_PASSWORD=$DB_PASSWORD \
                            -e MYSQL_DATABASE=$DB_NAME \
                            -v mysql-data:/var/lib/mysql \
                            mysql:8.0
                    elif [ -z "$(docker ps -q -f name=^mysql$)" ]; then
                        docker start mysql
                    fi
                    echo "Attente de MySQL..."
                    until docker exec mysql mysqladmin ping -h localhost -uroot -p$DB_PASSWORD --silent; do
                        sleep 3
                    done
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    docker stop backend-app || true
                    docker rm backend-app || true
                    docker pull $REMOTE_IMAGE
                    docker run -d --name backend-app --network app-net -p 8081:8080 \
                        -e SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/$DB_NAME \
                        -e SPRING_DATASOURCE_USERNAME=root \
                        -e SPRING_DATASOURCE_PASSWORD=$DB_PASSWORD \
                        $REMOTE_IMAGE
                '''
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    sleep 20
                    docker ps
                    docker logs backend-app
                '''
            }
        }
    }
}
