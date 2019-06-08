stage 'CI';
node {

    checkout scm

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    bat 'npm install'

    // stash code &amp; dependencies to expedite subsequent testing
    // and ensure same code &amp; dependencies are used throughout the pipeline
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

// demoing a second agent
node {

    // on windows use: bat 'dir'
    bat 'dir'

    // on windows use: bat 'del /S /Q *'
    bat 'del /S /Q *'

    unstash 'everything'

    // on windows use: bat 'dir'
    bat 'dir'
}

//parallel integration testing
// on windows: swap out Safari below for IE or just use Chrome and Firefox
stage 'Browser Testing'
parallel chrome: {
    runTests("Chrome")
}, firefox: {
    runTests("Firefox")
}

def runTests(browser) {
    node {
        // on windows use: bat 'del /S /Q *'
        bat 'del /S /Q *'

        unstash 'everything'

        // on windows use: bat "npm run test-single-run -- --browsers ${browser}"
        bat "npm run test-single-run -- --browsers ${browser}"
        step([$class: 'JUnitResultArchiver', 
              testResults: 'test-results/**/test-results.xml'])
    }
}

node {
    notify("Deploy to staging?")
}

input 'Deploy to staging?'

// limit concurrency so we don't perform simultaneous deploys
// and if multiple pipelines are executing, 
// newest is only that will be allowed through, rest will be canceled
stage name: 'Deploy', concurrency: 1
node {
    // write build number to index page so we can see this update
    
    // deploy to a docker container mapped to port 3000
    // on windows use: bat 'docker-compose up -d --build'
    bat 'docker-compose up -d --build'
    
    notify 'Solitaire Deployed!'
}











def notify(status){
    emailext (
      to: "wesmdemos@gmail.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """&lt;p&gt;${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':&lt;/p&gt;
        &lt;p&gt;Check console output at &lt;a href='${env.BUILD_URL}'&gt;${env.JOB_NAME} [${env.BUILD_NUMBER}]&lt;/a&gt;&lt;/p&gt;""",
    )
}