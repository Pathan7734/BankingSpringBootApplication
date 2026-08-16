node {
    def tag, dockerHubUser, containerName, httpPort = ""

    stage('Prepare Environment'){
        tag="3.0"
        containerName="bankingapp"
        httpPort="8989"
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
            dockerHubUser="$dockerUser"
        }
    }

    stage('Code Checkout'){
        checkout scm
    }

    stage('Maven Build'){
        sh "mvn clean package -DskipTests"
    }

    stage('Docker Image Build'){
        sh "docker build -t $dockerHubUser/$containerName:$tag --pull --no-cache ."
    }

    stage('Publishing Image to DockerHub'){
        withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'dockerUser', passwordVariable: 'dockerPassword')]) {
            sh "echo $dockerPassword | docker login -u $dockerUser --password-stdin"
            sh "docker push $dockerUser/$containerName:$tag"
        }
    }

    stage('Ansible Playbook Execution'){
        withCredentials([usernamePassword(credentialsId: 'azureVMAccount', usernameVariable: 'vmUser', passwordVariable: 'vmPassword')]) {
            sh """
                export ANSIBLE_HOST_KEY_CHECKING=False
                ansible-playbook -i inventory.yaml containerDeploy.yaml \
                -e httpPort=$httpPort \
                -e containerName=$containerName \
                -e dockerImageTag=$dockerHubUser/$containerName:$tag \
                -e key_pair_path=/var/lib/jenkins/server.pem \
                -e ansible_user=$vmUser \
                -e ansible_password=$vmPassword \
                --become
            """
        }
    }
}
