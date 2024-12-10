pipeline {
    agent any
    environment {
        DB_HOST = 'database-1.c1aoqqaugpm7.eu-north-1.rds.amazonaws.com'
        DB_NAME = 'test'
        AWS_BUCKET = 'test-jnk'
        GZ_FILE_NAME = "backup_${new Date().format('yyyyMMdd_HHmmss')}.sql.gz"
    }
    stages {
        stage('MySQL Backup and Gzip') {
            steps {
                script {
                    // Create a backup of the MySQL database and gzip it
                    withCredentials([usernamePassword(credentialsId: 'rds', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')]) {
                        sh '''
                                set -e
                                 mysqldump --single-transaction -h ${DB_HOST} -u ${DB_USER} -p${DB_PASSWORD} ${DB_NAME} | gzip > ${GZ_FILE_NAME}
                        '''

                    }
                }
            }
        }
        stage('Upload to S3') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                    credentialsId: 'aws'
                ]]) {
                    script {
                        sh '''
                            set -e
                            aws s3 cp ${GZ_FILE_NAME} s3://${AWS_BUCKET}/ --region eu-north-1
                        '''
                    }
                }
            }
        }
        stage('Delete Local Backup') {
            steps {
                script {
                    sh '''
                        set -e
                        rm -f ${GZ_FILE_NAME}
                    '''
                }
            }
        }
    }
 
    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo 'Backup and upload successful.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
