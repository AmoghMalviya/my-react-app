node {
    def app = "my-react-app"
    def imageName = "my-react-app:latest"
    def dockerRegistry = "localhost:8082"  // Nexus container URL
    def nexusCredentialsId = "nexus"  // Use Jenkins credential for Nexus login

    stage('Clone Repository') {
        checkout scm
    }

    stage('Build React App') {
        bat 'npm install'
        bat 'npm run build'
    }

    stage('Build Docker Image') {
        bat "docker build -t ${imageName} ."
    }

    stage('Tag and Push Docker Image to Nexus') {
        withCredentials([usernamePassword(credentialsId: nexusCredentialsId, usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            bat "docker login -u ${USERNAME} -p ${PASSWORD} ${dockerRegistry}"
            bat "docker tag ${imageName} ${dockerRegistry}/${imageName}"
            bat "docker push ${dockerRegistry}/${imageName}"
        }
    }

    stage('Deploy on Minikube') {
        bat '''
        kubectl delete deployment my-react-app-deployment || true
        kubectl delete service my-react-app-service || true

        kubectl create deployment my-react-app-deployment --image=${dockerRegistry}/${imageName}

        kubectl expose deployment my-react-app-deployment --type=NodePort --port=80 --target-port=80 --name=my-react-app-service
        '''
    }

    stage('Expose the App URL') {
        def nodePort = sh(script: "kubectl get service my-react-app-service -o=jsonpath='{.spec.ports[0].nodePort}'", returnStdout: true).trim()
        def minikubeIp = sh(script: "minikube ip", returnStdout: true).trim()
        echo "App is running on http://${minikubeIp}:${nodePort}"
    }
}
