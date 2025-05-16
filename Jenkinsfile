pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def branch = env.BRANCH_NAME
                    echo "Deploying from ${branch} branch"
                    
                    // Create deployment directory based on branch
                    def deployPath = "/var/www/html"
                    if (branch != "main") {
                        deployPath = "/var/www/html/${branch}"
                    }
                    
                    // Deploy to web server
                    sshagent(['web-server-ssh']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no ec2-user@172.31.94.234 "sudo mkdir -p ${deployPath}"
                            scp -r *.html ec2-user@172.31.94.234:/tmp/
                            ssh -o StrictHostKeyChecking=no ec2-user@172.31.94.234 "sudo cp -r /tmp/*.html ${deployPath}/"
                            ssh -o StrictHostKeyChecking=no ec2-user@172.31.94.234 "sudo chown -R apache:apache ${deployPath}"
                            
                            # Create Apache VirtualHost configuration for branches
                            if [ "${branch}" != "main" ]; then
                                ssh -o StrictHostKeyChecking=no ec2-user@172.31.94.234 "sudo bash -c 'cat > /etc/httpd/conf.d/${branch}.conf << EOL
<VirtualHost *:80>
    ServerName ${branch}.example.com
    DocumentRoot ${deployPath}
    <Directory ${deployPath}>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOL'"
                                ssh -o StrictHostKeyChecking=no ec2-user@172.31.94.234 "sudo systemctl reload httpd"
                            fi
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
