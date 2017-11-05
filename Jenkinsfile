pipeline {
  agent none
  stages {
    stage('Quality Checks') {
      agent {
        docker {
          image 'px4io/px4-dev-base:2017-10-23'
          args '--env CCACHE_DISABLE=1 --env CI=true'
        }
        
      }
      steps {
        sh 'make check_format'
      }
    }
    stage('Build') {
      parallel {
        stage('posix_sitl_default') {
          agent {
            docker {
              image 'px4io/px4-dev-base:2017-10-23'
            }
            
          }
          steps {
            sh 'make posix_sitl_default'
          }
        }
        stage('nuttx_px4fmu-v2_default') {
          agent {
            docker {
              image 'px4io/px4-dev-nuttx:2017-10-23'
              args '-e CCACHE_DISABLE=1'
            }
            
          }
          steps {
            sh 'make nuttx_px4fmu-v2_default'
            archive 'build/*/*.px4'
          }
        }
        stage('nuttx_px4fmu-v5_default') {
          agent {
            docker {
              image 'px4io/px4-dev-nuttx:2017-10-23'
              args '-e CCACHE_DISABLE=1'
            }
            
          }
          steps {
            sh 'make nuttx_px4fmu-v5_default'
            archive 'build/*/*.px4'
          }
        }
      }
    }
    stage('Test') {
      steps {
        sh 'make posix_sitl_default test_results_junit'
        junit 'build/posix_sitl_default/JUnitTestResults.xml'
      }
    }
    stage('Generate Metadata') {
      parallel {
        stage('airframe') {
          agent {
            docker {
              image 'px4io/px4-dev-base:2017-10-23'
            }
            
          }
          steps {
            sh 'make airframe_metadata'
            archive 'airframes.md, airframes.xml'
          }
        }
        stage('parameters') {
          agent {
            docker {
              image 'px4io/px4-dev-base:2017-10-23'
            }
            
          }
          steps {
            sh 'make parameters_metadata'
            archive 'parameters.md, parameters.xml'
          }
        }
        stage('modules') {
          agent {
            docker {
              image 'px4io/px4-dev-base:2017-10-23'
            }
            
          }
          steps {
            sh 'make module_documentation'
            archive 'modules/*.md'
          }
        }
      }
    }
    stage('S3 Upload') {
      agent {
        docker {
          image 'px4io/px4-dev-base:2017-10-23'
        }
        
      }
      when {
        branch 'master|beta|stable'
      }
      steps {
        sh 'echo "uploading to S3"'
      }
    }
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '20'))
    timeout(time: 10, unit: 'MINUTES')
    timestamps()
  }
}