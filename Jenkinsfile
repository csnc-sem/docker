@Library('simplerisk') _

pipeline {
	agent none
	stages {
		stage ('Initializing Common Variables') {
			agent { label 'terminator' }
			steps {
				script {
					current_version = miscOps.getOfficialVersion("updates")
					committer_email = gitOps.getCommitterEmail()
				}
			}
			post {
				failure {
					script { emailOps.sendErrorEmail("${env.STAGE_NAME}", "${committer_email}") }
				}
			}
		}
		stage ('simplerisk/simplerisk') {
			agent { label 'buildtestmed' }
			stages {
				stage ('Initialize Variables') {
					agent { label 'buildtestmed' }
					steps {
						script {
							instance_id = awsOps.getEC2Metadata("instance-id")
							main_stage = "simplerisk"
						}
					}
					post {
						failure {
							script { emailOps.sendErrorEmail("${main_stage}/${env.STAGE_NAME}", "${committer_email}") }
						}
					}
				}
				stage ('Build') {
					parallel {
						stage ('Ubuntu 18.04') {
							agent { label 'buildtestmed' }
							steps {
								script {
									image = env.STAGE_NAME
									dockerOps.buildSingleDockerImage(docker_image:"${main_stage}", version:"${current_version}", os_type:"bionic", with_os:true)
								}
							}
							post {
								success {
									script { if (env.CHANGE_ID) { pullRequest.createStatus(status: "success", context: "build/simplerisk/ubuntu-1804", description: "Image built successfully", targetUrl: "$BUILD_URL") } }
								}
								failure {
									script { emailOps.sendErrorEmail("${main_stage}/${env.STAGE_NAME}/${image}", "${committer_email}") }
								}
							}
						}
						stage ('Ubuntu 20.04') {
							agent { label 'buildtestmed' }
							steps {
								script {
									image = env.STAGE_NAME
									dockerOps.buildFullSetOfDockerImages("${main_stage}", "${current_version}", "focal")
								}
							}
							post {
								success {
									script { if (env.CHANGE_ID) { pullRequest.createStatus(status: "success", context: "build/simplerisk/ubuntu-2004", description: "Image built successfully", targetUrl: "$BUILD_URL") } }
								}
								failure {
									script { emailOps.sendErrorEmail("${main_stage}/${env.STAGE_NAME}/${image}", "${committer_email}") }
								}
							}
						}
					}
				}
				stage ('Push to Docker Hub') {
					when { branch 'master' }
					parallel {
						stage ("Current Version - bionic") {
							agent { label 'buildtestmed' }
							steps {
								script {
									image = env.STAGE_NAME
									dockerOps.pushSingleDockerImage(docker_image:"${main_stage}", version:"${current_version}", os_type:"bionic")
								}
							}
						}
						stage ("Latest/Current Version/Current Version - focal") {
							agent { label 'buildtestmed' }
							steps {
								script {
									image = env.STAGE_NAME
									dockerOps.pushFullSetOfDockerImages("${main_stage}", "${current_version}", "focal")
								}
							}
						}
					}
					post {
						failure {
							script { emailOps.sendErrorEmail("${main_stage}/${env.STAGE_NAME}/${image}", "${committer_email}") }
						}
					}
				}
			}
			post {
				cleanup {
					script { dockerOps.deleteAllImages() }
				}
			}
		}
		stage ('simplerisk/simplerisk-minimal') {
			agent { label 'buildtestmed' }
			stages {
				stage ('Initialize Variables') {
					agent { label 'buildtestmed' }
					steps {
						script {
							instance_id = awsOps.getEC2Metadata("instance-id")
							main_stage = "simplerisk-minimal"
						}
					}
					post {
						failure {
							script {
								awsOps.terminateInstance("${instance_id}")
								emailOps.sendErrorEmail("${main_stage}/${env.STAGE_NAME}", "${committer_email}")
							}
						}
						aborted {
							script { awsOps.terminateInstance("${instance_id}") }
						}
					}
				}
				stage ('Build') {
					parallel {
						stage ('PHP 7.2') {
							agent { label 'buildtestmed' }
							steps {
								script {
									image = env.STAGE_NAME
									dockerOps.buildSingleDockerImage(docker_image:"${main_stage}", version:"${current_version}", os_type:"php72", with_os:true)
								}
							}
							post {
								success {
									script {
										if (env.CHANGE_ID) { pullRequest.createStatus(status: "success", context: "build/simplerisk-minimal/php-72", description: "Image built successfully", targetUrl: "$BUILD_URL") }
									}
								}
								failure {
									script {
										awsOps.terminateInstance("${instance_id}")
										emailOps.sendErrorEmail("${main_stage}/${env.STAGE_NAME}/${image}", "${committer_email}")
									}
								}
								aborted { script { awsOps.terminateInstance("${instance_id}") } }
							}
						}
						stage ('PHP 7.4') {
							agent { label 'buildtestmed' }
							steps {
								script {
									image = env.STAGE_NAME
									dockerOps.buildFullSetOfDockerImages("${main_stage}", "${current_version}", "php74")
								}
							}
							post {
								success {
									script {
										if (env.CHANGE_ID) { pullRequest.createStatus(status: "success", context: "build/simplerisk-minimal/php-74", description: "Image built successfully", targetUrl: "$BUILD_URL") }
									}
								}
								failure {
									script {
										awsOps.terminateInstance("${instance_id}")
										emailOps.sendErrorEmail("${main_stage}/${env.STAGE_NAME}/${image}", "${committer_email}")
									}
								}
								aborted {
									script { awsOps.terminateInstance("${instance_id}") }
								}
							}
						}
					}
				}
				stage ('Push to Docker Hub') {
					when { branch 'master' }
					parallel {
						stage ('Current Version - PHP 7.2') {
							agent { label 'buildtestmed' }
							steps {
								script {
									image = env.STAGE_NAME
									dockerOps.pushSingleDockerImage(docker_image:"${main_stage}", version:"${current_version}", os_type:"php72")
								}
							}
						}
						stage ("Latest/Current Version/Current Version - PHP 7.4") {
							agent { label 'buildtestmed' }
							steps {
								script {
									image = env.STAGE_NAME
									dockerOps.pushFullSetOfDockerImages("${main_stage}", "${current_version}", "php74")
								}
							}
						}
					}
					post {
						failure {
							script {
								awsOps.terminateInstance("${instance_id}")
								emailOps.sendErrorEmail("${main_stage}/${env.STAGE_NAME}/${image}")
							}
						}
						aborted {
							script { awsOps.terminateInstance("${instance_id}") }
						}
					}
				}
			}
			post {
				cleanup {
					script { dockerOps.deleteAllImages() }
				}
			}
		}
	}
	post {
		success {
			script { emailOps.sendSuccessEmail("${committer_email}") }
		}
	}
}
