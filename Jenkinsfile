pipeline{
    
    agent any
    tools {nodejs "NODEJS"} 
    environment{
        user_env_input = "Development"
    }
    
    stages{
        
        
        stage('Which environment to build?') {
            steps {
                script {
                    def userInput = input(id: 'userInput', message: 'Deploy to?',
                    parameters: [[$class: 'ChoiceParameterDefinition', defaultValue: 'Development', 
                        description:'Environment choices', name:'denv', choices: "Development\nProduction\nTesting"]
                    ])
                    user_env_input = userInput
                    //Use this value to branch to different logic if needed
                }
            }
        }
        stage('Confirm') {
            steps {
                input("Do you want to proceed building in ${user_env_input} environment?")
            }
        }       
        
        
       
        stage("build")
        {
            steps{
            sh 'node --version'
           sh 'npm install'
     
            sh 'ng build'
        }
        }
        stage("aws cp")
        {
            steps{            


script{

if (user_env_input == "Production") {
    sh 'echo copying to s3 bucket prod env'
}

else if (user_env_input == "Testing") {
   sh 'echo copying to s3 bucket test env'
}
else {
    sh 'echo copying to s3 bucket dev env'
}




            }
        }
    }
    
    }
}
