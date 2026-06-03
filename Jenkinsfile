node {
    def appDir = 'var/www/nextjs-app'

    stage('Clean Workspace'){
        echo 'Cleaning workspace....'
        deleteDir()
    }
    stage('Clone Repo'){
        echo 'Cloning repository...'
        git(
            branch: 'main',
            url: 'https://github.com/Omarmdwasimuddin/Jenkins-CI-CD-Pipeline-with-AWS-EC2'
        )
    }
    stage('Deploy to EC2'){
        echo 'Deploying to EC2...'
        sh """
        
         sudo mkdir -p ${appDir}
         sudo chown -R jenkins:jenkins ${appDir}
         rsync -av --delete --exclude='.git' --exclude='node_modules' ./ ${appDir} 
         cd ${appDir}
         sudo npm install
         sudo npm run build
         sudo fuser -k 3000/tcp || true
         sudo npm start

        """
    }
}