node {
    try {
        stage 'Build'
        
        def workspace = pwd() 
        def version = env.BRANCH_NAME

        checkout scm

        if(env.BRANCH_NAME=="master") {
            version = sh(returnStdout: true, script: 'git describe --abbrev=0 --tags').trim()
            checkout([$class: 'GitSCM', branches: [[name: "tags/${version}"]], extensions: [[$class: 'CleanCheckout'], [$class: 'LocalBranch', localBranch: "master"]]])
        }
        
        sh """
        cd api
        mvn versions:set -DnewVersion=${version}
        mvn clean package
        """

        stage 'Test'
        sh '''
        cd api/src/main/resources
        pylint_wrapper.py 10
        nosetests test_*.py
        '''

        stage 'Deploy' 
        

        emailext attachLog: true, body: "Build succeeded (see ${env.BUILD_URL})", subject: "[JENKINS] ${env.JOB_NAME} succeeded", to: "trsmith2@cisco.com"

    }
    catch(error) {
        emailext attachLog: true, body: "Build failed (see ${env.BUILD_URL})", subject: "[JENKINS] ${env.JOB_NAME} failed", to: "trsmith2@cisco.com"
        currentBuild.result = "FAILED"
        throw error
    }
}
