stage 'CI'
node {

    checkout scm

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    bat 'npm install'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'test-results/**', 
          includes: '**'

    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    bat 'npm run test-single-run -- --browsers PhantomJS'

    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: 'test-results/**/test-results.xml'])
}

stage 'Browser Testing'

parallel 'Chrome': {
    RunTests("Chrome")
}

def RunTests(browser) {
    node {
        bat 'del /F /Q *.*'
        unstash 'everything'
        bat "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver', testResults: 'test-results/**/test-results.xml'])
    }
}

input 'Deploye to Staging?'

def notify(status){
    emailext (
      to: "wesmdemos@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}