stage('Test Application')
{
    node ('docker-jenkins-slave')
    {
        git credentialsId: 'gitHub', url: "${GIT_REPO_URL}"
        sh 'chmod a+x ./runTest.sh'
        sh './runTest.sh'
    }
}

node('slave1')
{
    git credentialsId: 'gitHub', url: "${GIT_REPO_URL}"
    stage('Build Application Image')
    {
        withDockerServer([credentialsId: 'dd8fd66f-bcf2-4aff-962b-73a4f65756e9', uri: "tcp://${DOCKER_HOST}:2376"])
        {
            docker.build "${DOCKER_REGISTRY_USER}/rsvpapp:mooc_v1"
        }
    }
    stage('Push Image to Registry')
    {
        withDockerServer([credentialsId: 'dd8fd66f-bcf2-4aff-962b-73a4f65756e9', uri: "tcp://${DOCKER_HOST}:2376"])
        {
            withDockerRegistry(credentialsId: 'DockerHub')
            {
                docker.image("${DOCKER_REGISTRY_USER}/rsvpapp:mooc_v1").push()
            }
        }
    }
    stage('Deploy Image to Staging')
    {
        withDockerServer([credentialsId: 'staging-server', uri: "tcp://${STAGING_HOST}:2376"])
        {
            sh 'docker-compose pull'
            sh 'docker-compose -p rsvp_staging up -d'
        }
        input "Check Application running at http://${STAGING_HOST}:5000. Looks good?"
        withDockerServer([credentialsId: 'staging-server', uri: "tcp://${STAGING_HOST}:2376"])
        {
            sh 'docker-compose -p rsvp_staging down -v'
        }
    }
    stage('Deploy App to Production')
    {
        withDockerServer([credentialsId:'production', uri:"tcp://${PRODUCTION_HOST}:2376"])
        {
            sh 'docker stack deploy -c docker-stack.yaml myrsvpapp'
        }
        input "Check application running at http://${LOAD_BALANCER_HOST} or http://${PRODUCTION_HOST}:5000"
        withDockerServer([credentialsId:'production', uri:"tcp://${PRODUCTION_HOST}:2376"])
        {
            sh 'docker stack down myrsvpapp'
        }
    }
}