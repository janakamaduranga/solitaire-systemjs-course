stage &apos;CI&apos;
node {

	checkout scm
    //git branch: &apos;jenkins2-course&apos;, 
        //url: &apos;https://github.com/g0t4/solitaire-systemjs-course&apos;

    // pull dependencies from npm
    // on windows use: bat &apos;npm install&apos;
    bat &apos;npm install&apos;

    // stash code &amp; dependencies to expedite subsequent testing
    // and ensure same code &amp; dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: &apos;everything&apos;, 
          excludes: &apos;test-results/**&apos;, 
          includes: &apos;**&apos;
    
    // test with PhantomJS for &quot;fast&quot; &quot;generic&quot; results
    // on windows use: bat &apos;npm run test-single-run -- --browsers PhantomJS&apos;
    bat &apos;npm run test-single-run -- --browsers PhantomJS&apos;
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: &apos;JUnitResultArchiver&apos;, 
          testResults: &apos;test-results/**/test-results.xml&apos;])
          
}

// demoing a second agent
node {

    // on windows use: bat &apos;dir&apos;
    bat &apos;dir&apos;

    // on windows use: bat &apos;del /S /Q *&apos;
    bat &apos;del /S /Q *&apos;

    unstash &apos;everything&apos;

    // on windows use: bat &apos;dir&apos;
    bat &apos;dir&apos;
}

//parallel integration testing
// on windows: swap out Safari below for IE or just use Chrome and Firefox
stage &apos;Browser Testing&apos;
parallel chrome: {
    runTests(&quot;Chrome&quot;)
}, firefox: {
    runTests(&quot;Firefox&quot;)
}

def runTests(browser) {
    node {
        // on windows use: bat &apos;del /S /Q *&apos;
        bat &apos;del /S /Q *&apos;

        unstash &apos;everything&apos;

        // on windows use: bat &quot;npm run test-single-run -- --browsers ${browser}&quot;
        bat &quot;npm run test-single-run -- --browsers ${browser}&quot;
        step([$class: &apos;JUnitResultArchiver&apos;, 
              testResults: &apos;test-results/**/test-results.xml&apos;])
    }
}

node {
    notify(&quot;Deploy to staging?&quot;)
}

input &apos;Deploy to staging?&apos;

// limit concurrency so we don&apos;t perform simultaneous deploys
// and if multiple pipelines are executing, 
// newest is only that will be allowed through, rest will be canceled
stage name: &apos;Deploy&apos;, concurrency: 1
node {
    // write build number to index page so we can see this update
    
    // deploy to a docker container mapped to port 3000
    // on windows use: bat &apos;docker-compose up -d --build&apos;
    bat &apos;docker-compose up -d --build&apos;
    
    notify &apos;Solitaire Deployed!&apos;
}











def notify(status){
    emailext (
      to: &quot;wesmdemos@gmail.com&quot;,
      subject: &quot;${status}: Job &apos;${env.JOB_NAME} [${env.BUILD_NUMBER}]&apos;&quot;,
      body: &quot;&quot;&quot;&lt;p&gt;${status}: Job &apos;${env.JOB_NAME} [${env.BUILD_NUMBER}]&apos;:&lt;/p&gt;
        &lt;p&gt;Check console output at &lt;a href=&apos;${env.BUILD_URL}&apos;&gt;${env.JOB_NAME} [${env.BUILD_NUMBER}]&lt;/a&gt;&lt;/p&gt;&quot;&quot;&quot;,
    )
}