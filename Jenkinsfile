pipeline {
	agent none
	environment {
		ON_JENKINS = 'TRUE'
	}
	stages {
	  stage ('Lisk Provisioning') {
			steps {
					node('master-nano-01'){
					lock(resource: "master-nano-01", inversePrecedence: true) {
						sh '''#!/bin/bash
									env

									# Clean up old processes
									pkill -f selenium -9 || true
									pkill -f Xvfb -9 || true
									rm -rf /tmp/.X0-lock || true
									pkill -f app.js || true

									# Start lisk and make sure its current
									cd /var/lib/jenkins/workspace/
									git clone https://github.com/LiskHQ/lisk.git
									cd /var/lib/jenkins/workspace/lisk
									git checkout development
									git pull
									dropdb lisk_test || true
									createdb lisk_test
									psql -d lisk_test -c "alter user "$USER" with password 'password';"
									cp /var/lib/jenkins/workspace/lisk-node-Linux-x86_64.tar.gz .
									tar -zxvf lisk-node-Linux-x86_64.tar.gz
									npm install
									git submodule init
									git submodule update
									cd public
									npm install
									bower install
									grunt release
									cd ..
									cd test/lisk-js/; npm install; cd ../..
									cp test/config.json test/genesisBlock.json .
									export NODE_ENV=test
									BUILD_ID=dontKillMe ~/start_lisk.sh

									# Build nano
									cd $WORKSPACE/src
									npm install

									if [ -z ${ghprbPullId+x} ]; then echo "Not a PR build"; else export CI_PULL_REQUEST=$ghprbPullId; fi
									cd $WORKSPACE/src
									cp ~/.coveralls.yml-nano .coveralls.yml
									npm run build

									# Start nano in development environment
									npm run dev &> .lisk-nano.log &

									# Prepare lisk core for testing
									bash ~/tx.sh

									# Run nano tests
									npm run test

									# Commented until e2e is ready
									# export CHROME_BIN=chromium-browser
									# export DISPLAY=:0.0
									# Xvfb :0 -ac -screen 0 1280x1024x24 &
									# ./node_modules/protractor/bin/webdriver-manager update
									# npm run e2e-test

									# Commented until e2e is ready
									# cat .protractor.log
									cat .lisk-nano.log

									pkill -f app.js -9 || true
									# Commented until e2e is ready
									# pkill -f webpack-dev-server -9
							 '''
							 milestone 1
						}
					}
				}
		}
	}
}
