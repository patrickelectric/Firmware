pipeline {
  agent none
  options {
    buildDiscarder(logRotator(numToKeepStr: '20'))
    timeout(time: 10, unit: 'MINUTES')
    timestamps()
  }
  stages {
        stage("Build/Test") {
            steps {
                script {
                  def builds = [:]

                  for (def option in ["one", "two"]) {
                          def node_name = ""
                          if ("one" == "${option}") {
                              node_name = "node001"
                          } else {
                              node_name = "node002"
                          }
                          
                          def option_inside = "${option}"
                          
                          builds["${node_name} ${option_inside}"] = {
                              node {
                                  stage("Build Test ${node_name} ${option_inside}") {
                                    agent {
                                      docker {
                                        image 'px4io/px4-dev-base:2017-10-23'
                                        args '--env CCACHE_DISABLE=1 --env CI=true'
                                      }
                                    }
                                    steps {
                                      sh 'ping -c 10 localhost'
                                    }
                                  }
                              }
                          }
                  }
                  parallel builds
                }
            }
        }
    stage('Build') {
      parallel {
        stage('posix_sitl_default') {
          agent {
            docker {
              image 'px4io/px4-dev-base:2017-10-23'
              args '--env CCACHE_DISABLE=1 --env CI=true'
            }
          }
          steps {
            sh 'make clean'
            sh 'make posix_sitl_default'
          }
        }
        stage('nuttx_px4fmu-v2_default') {
          agent {
            docker {
              image 'px4io/px4-dev-nuttx:2017-10-23'
              args '--env CCACHE_DISABLE=1 --env CI=true'
            }
          }
          steps {
            sh 'make clean'
            sh 'make nuttx_px4fmu-v2_default'
            archive 'build/*/*.px4'
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('check_format') {
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
        stage('clang-tidy') {
          agent {
            docker {
              image 'px4io/px4-dev-clang:2017-10-23'
              args '--env CCACHE_DISABLE=1 --env CI=true'
            }
          }
          steps {
            sh 'make clang-tidy-quiet'
          }
        }
        stage('tests') {
          agent {
            docker {
              image 'px4io/px4-dev-base:2017-10-23'
              args '--env CCACHE_DISABLE=1 --env CI=true'
            }
          }
          steps {
            sh 'make posix_sitl_default test_results_junit'
            junit 'build/posix_sitl_default/JUnitTestResults.xml'
          }
        }
      }
    }
    stage('Generate Metadata') {
      agent {
        docker {
          image 'px4io/px4-dev-base:2017-10-23'
        }
      }
      steps {
        sh 'make airframe_metadata'
        archive 'airframes.md, airframes.xml'
        sh 'make parameters_metadata'
        archive 'parameters.md, parameters.xml'
        sh 'make module_documentation'
        archive 'modules/*.md'
      }
    }
    stage('S3 Upload') {
      agent {
        docker {
          image 'px4io/px4-dev-base:2017-10-23'
        }
      }
      when {
        branch '*/master|*/beta|*/stable'
      }
      steps {
        sh 'echo "uploading to S3"'
      }
    }
  }
}
