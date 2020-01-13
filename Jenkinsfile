pipeline {
  agent any

  tools {
    maven "apache-maven-3.6.3"
  }

  stages {
    stage('Build') {
      steps {
        git 'https://github.com/ajlanghorn/dvja.git'
        sh "mvn clean package"
      }
    }
    stage('Analysis') {
      steps {
        sh "mvn --batch-mode -V -U -e checkstyle:checkstyle pmd:pmd pmd:cpd com.github.spotbugs:spotbugs-maven-plugin:3.1.7:spotbugs"
      }
    }
    stage('Check dependencies') {
      steps {
        dependencyCheck additionalArguments: '', odcInstallation: 'Dependency-Check'
        dependencyCheckPublisher pattern: ''
      }
    }
    stage('Scan for vulnerabilities') {
      steps {
        sh 'nohup mvn jetty:run 2>&1 &'
        sh 'zap-cli quick-scan --self-contained --spider -r http://127.0.0.1:8080 && zap-cli report -o zap-report.html -f html'
      }
    }
    stage('Publish to S3') {
      steps {
        sh "aws s3 cp /var/lib/jenkins/workspace/dvja/target/dvja-1.0-SNAPSHOT.war s3://ako20-buildartifacts-16n3i2hlt458h/dvja-1.0-SNAPSHOT.war"
      }
    }
    stage('Tidy up') {
      steps {
        cleanWs()
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'zap-report.html', fingerprint: true
      recordIssues enabledForFailure: true, tools: [mavenConsole(), java(), javaDoc()]
      recordIssues enabledForFailure: true, tool: checkStyle()
      recordIssues enabledForFailure: true, tool: spotBugs()
      recordIssues enabledForFailure: true, tool: cpd(pattern: '**/target/cpd.xml')
      recordIssues enabledForFailure: true, tool: pmdParser(pattern: '**/target/pmd.xml')
    }
  }

}
