#!groovy
node('tse-control-repo') {
  sshagent (credentials: ['jenkins-seteam-ssh']) {
    withEnv(['PATH+EXTRA=/usr/local/bin']) {
      checkout scm

      stage('Setup'){
        ansiColor('xterm') {
          sh(script: '''
            export PATH=$PATH:$HOME/.rbenv/bin

            set +e
            which rbenv &>/dev/null
            if [ $? != 0 ]; then
              set -e
              git clone 'https://github.com/rbenv/rbenv.git' ~/.rbenv
              mkdir -p "$(rbenv root)"/plugins
              git clone 'https://github.com/rbenv/ruby-build.git' "$(rbenv root)"/plugins/ruby-build
            fi
            set -e

            set +e
            rbenv global 2.3.1 || rbenv install 2.3.1
            set -e

            eval "$(rbenv init -)"
            bundle install
          ''')
        }
      }

      stage('Lint Control Repo'){
        ansiColor('xterm') {
          sh(script: '''
            export PATH=$PATH:$HOME/.rbenv/bin
            rbenv global 2.3.1
            eval "$(rbenv init -)"
            bundle exec rake lint
          ''')
        }
      }

      stage('Syntax Check Control Repo'){
        ansiColor('xterm') {
          sh(script: '''
            export PATH=$PATH:$HOME/.rbenv/bin
            rbenv global 2.3.1
            eval "$(rbenv init -)"
            bundle exec rake syntax --verbose
          ''')
        }
      }

      stage('Validate Puppetfile in Control Repo'){
        ansiColor('xterm') {
          sh(script: '''
            export PATH=$PATH:$HOME/.rbenv/bin
            rbenv global 2.3.1
            eval "$(rbenv init -)"
            bundle exec rake r10k:syntax
          ''')
        }
      }

      stage('Validate Tests Exist'){
        ansiColor('xterm') {
          sh(script: '''
            export PATH=$PATH:$HOME/.rbenv/bin
            rbenv global 2.3.1
            eval "$(rbenv init -)"
            bundle exec rake check_for_spec_tests
          ''')
        }
      }
    }
  }
}

stage('Run Spec Tests') {
  parallel(
    'linux::profile::spec': {
      runSpecTests('linux')
    },
    'windows::profile::spec': {
      runSpecTests('windows')
    }
  )
}


// functions
def linux(){
  withEnv(['PATH+EXTRA=/usr/local/bin']) {
    ansiColor('xterm') {
      sh(script: '''
        export PATH=$PATH:$HOME/.rbenv/bin:$HOME/.rbenv/shims
        echo $PATH
        rbenv global 2.3.1
        gem install bundle
        bundle install
        bundle exec rake spec
      ''')
    }
  }
}

def windows(){
  withEnv(['MODULE_WORKING_DIR=C:/tmp']) {
    ansiColor('xterm') {
      sh(script: '''
        bundle install
        bundle exec rake spec
      ''')
    }
  }
}

def runSpecTests(def platform){
  node('tse-slave-' + platform) {
    sshagent (credentials: ['jenkins-seteam-ssh']) {
      checkout scm
      "$platform"()
    }
  }
}
