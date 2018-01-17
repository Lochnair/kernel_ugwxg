pipeline {
    agent {
        docker { image 'lochnair/octeon-buildenv:latest' }
    }

    stages {
        stage('Clean') {
            steps {
               sh 'git reset --hard'
               sh 'git clean -fX'
            }
        }

        stage('Prepare for out-of-tree builds') {
            steps {
                sh 'make -j5 ARCH=mips CROSS_COMPILE=mips64-octeon-linux- prepare modules_prepare'
                sh 'rm -rf tmp && mkdir tmp'
                sh 'tar --exclude-vcs --exclude=tmp -cf tmp/ugwxg-ksrc.tar .'
                sh 'mv tmp/ugwxg-ksrc.tar ugwxg-ksrc.tar'
            }
        }

        stage('Build') {
            steps {
                sh 'make -j5 ARCH=mips CROSS_COMPILE=mips64-octeon-linux- vmlinux modules'
            }
        }
        
        stage('Archive kernel image') {
            steps {
                sh 'mv -v vmlinux vmlinux.64'
                sh 'tar cvjf ugwxg-kernel.tar.bz2 vmlinux.64'
                archiveArtifacts artifacts: 'ugwxg-kernel.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }
        
        stage('Archive kernel modules') {
            steps {
                sh 'make ARCH=mips CROSS_COMPILE=mips64-octeon-linux- INSTALL_MOD_PATH=destdir modules_install'
                sh 'tar cvjf ugwxg-modules.tar.bz2 -C destdir .'
                archiveArtifacts artifacts: 'ugwxg-modules.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }

        stage('Archive build tree') {
            steps {
                sh 'tar -uvf ugwxg-ksrc.tar Module.symvers'
                sh 'bzip2 ugwxg-ksrc.tar'
                archiveArtifacts artifacts: 'ugwxg-ksrc.tar.bz2', fingerprint: true, onlyIfSuccessful: true
            }
        }
    }
}
