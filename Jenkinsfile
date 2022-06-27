pipeline {
    agent any

    parameters {
        string(name: 'environment', defaultValue: 'terraform', description: 'Workspace/environment file to use for deployment')
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
        booleanParam(name: 'destroy', defaultValue: false, description: 'Destroy Terraform build?')

    }

    stages {
        stage('checkout') {
            steps {
                 script{
                        dir("terraform")
                        {   
                            git branch: 'main', url: '<replace with repo url>'
                        }
                    }
                }
            }

        stage('Plan') {
            when {
                not {
                    equals expected: true, actual: params.destroy
                }
            }
            
            steps {
                withAWS(region:'us-west-2',credentials:'aws') {
                    sh 'terraform -chdir=examples/node-groups/managed-node-groups init -backend-config backend.conf -reconfigure -input=false'
                    sh 'terraform workspace select ${environment} || terraform workspace new ${environment}'

                    sh "terraform -chdir=examples/node-groups/managed-node-groups plan -var-file base.tfvars -input=false -out tfplan "
                    sh 'terraform -chdir=examples/node-groups/managed-node-groups show -no-color tfplan > tfplan.txt'
                }
            }
        }
        stage('Approval') {
           when {
               not {
                   equals expected: true, actual: params.autoApprove
               }
               not {
                    equals expected: true, actual: params.destroy
                }
           }
           
                
            

           steps {
               script {
                    def plan = readFile 'tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Apply') {
            when {
                not {
                    equals expected: true, actual: params.destroy
                }
            }
            
            steps {
                 withAWS(region:'us-west-2',credentials:'aws') {
                    sh "terraform -chdir=examples/node-groups/managed-node-groups apply -input=false tfplan"
                 }
            }
        }
        
        stage('Destroy') {
            when {
                equals expected: true, actual: params.destroy
            }
        
        steps {
             withAWS(region:'us-west-2',credentials:'aws') {
                sh "terraform -chdir=examples/node-groups/managed-node-groups destroy -var-file=base.tfvars -auto-approve"
             }
        }
    }

  }
}
