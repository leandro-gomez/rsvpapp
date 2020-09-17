stage('test') {
    node("docker-jenkins") { 
        checkout scm
        sh 'chmod a+x ./run_tests.sh'
        sh './run_tests.sh'
    }
}

node() {
    checkout scm
    stage("build image") {
        // This step should not normally be used in your script. Consult the inline help for details.
        withDockerServer([credentialsId: 'dockerhost0', uri: 'tcp://192.168.99.100:2376']) {
            docker.build 'leandrogomez/rsvpapp:mooc'
        }
    }    
    stage("push image") {
        withDockerServer([credentialsId: 'dockerhost0', uri: 'tcp://192.168.99.100:2376']) {
            // This step should not normally be used in your script. Consult the inline help for details.
            withDockerRegistry(credentialsId: 'dockerhub_auth') {
                docker.image('leandrogomez/rsvpapp:mooc').push()
            }
        }

    }
    stage("deploy to stg server") {
        // This step should not normally be used in your script. Consult the inline help for details.
        withDockerServer([credentialsId: 'dockerstg0', uri: 'tcp://192.168.99.101:2376']) {
            sh 'docker-compose -p rsvp_staging up -d'
        }
        input 'is http://192.168.99.101:5000 up?'
        withDockerServer([credentialsId: 'dockerstg0', uri: 'tcp://192.168.99.101:2376']) {
            sh 'docker-compose -p rsvp_staging down -v'
        }
    
    }
    stage("deploy to prod server") {

        // This step should not normally be used in your script. Consult the inline help for details.
        withDockerServer([credentialsId: 'docker-swarm-certs', uri: 'tcp://192.168.99.105:2376']) {
            sh 'docker stack deploy -c docker-stack.yml myrsvpapp'
        }
    
    }
}
