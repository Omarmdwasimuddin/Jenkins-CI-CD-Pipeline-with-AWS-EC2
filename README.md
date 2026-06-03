## Jenkins-CI-CD-Pipeline-with-AWS-EC2

[EC2 Server e Jenkins Install korar process dekhun](https://github.com/Omarmdwasimuddin/Jenkins-Install-on-Ubuntu)

#### Nextjs diye ekta project create koro--->github e push koro.
#### Browser e http://18.143.149.173:8080/ diye hit koro--->Jenkins open koro.
#### project er root directory te Jenkinsfile create koro--->
```
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
```

####
