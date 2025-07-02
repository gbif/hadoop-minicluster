@Library('gbif-common-jenkins-pipelines') _

pipeline {
  agent any
  tools {
    maven 'Maven 3.9.9'
    jdk 'OpenJDK17'
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    skipStagesAfterUnstable()
    timestamps()
  }
  triggers {
    snapshotDependencies()
  }
  parameters {
    separator(name: "release_separator", sectionHeader: "Release Main Project Parameters")
    booleanParam(name: 'RELEASE', defaultValue: false, description: 'Do a Maven release')
    string(name: 'RELEASE_VERSION', defaultValue: '', description: 'Release version (optional)')
    string(name: 'DEVELOPMENT_VERSION', defaultValue: '', description: 'Development version (optional)')
    booleanParam(name: 'DRY_RUN_RELEASE', defaultValue: false, description: 'Dry Run Maven release')
    separator(name: "release_separator", sectionHeader: "Release Trino UDFs Parameters")
  }
  environment {
    JETTY_PORT = utils.getPort()
    POM_VERSION = readMavenPom().getVersion()
  }

  stages {

    stage('Maven build: mini cluster module (Java 17)') {
      tools {
        jdk 'OpenJDK17'
      }
      steps {
        configFileProvider([
            configFile(fileId: 'org.jenkinsci.plugins.configfiles.maven.GlobalMavenSettingsConfig1387378707709', variable: 'MAVEN_SETTINGS')
          ]) {
          sh 'mvn -s ${MAVEN_SETTINGS} clean deploy -pl occurrence-hadoop-minicluster'
        }
      }
    }

    stage('Maven release: mini cluster module') {
      when {
          allOf {
              expression { params.RELEASE };
              branch 'master';
          }
      }
      environment {
          RELEASE_ARGS = utils.createReleaseArgs(params.RELEASE_VERSION, params.DEVELOPMENT_VERSION, params.DRY_RUN_RELEASE)
      }
      steps {
        configFileProvider([
            configFile(fileId: 'org.jenkinsci.plugins.configfiles.maven.GlobalMavenSettingsConfig1387378707709', variable: 'MAVEN_SETTINGS_XML')
          ]) {
          git 'https://github.com/gbif/occurrence.git'
          sh 'mvn -s $MAVEN_SETTINGS_XML -B -Denforcer.skip=true release:prepare release:perform $RELEASE_ARGS -pl occurrence-hadoop-minicluster'
        }
      }
    }
  }
  
  post {
      success {
        echo 'Pipeline executed successfully!'
      }
      failure {
        echo 'Pipeline execution failed!'
      }
  }
}
