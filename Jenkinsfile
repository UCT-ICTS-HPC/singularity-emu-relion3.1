// Environment Variables
env.SW_LOCATION = '/opt/exp_soft/singularity-containers/relion'
env.CONTAINER_NAME = 'relion-gpu'
env.CONTAINER_FMT = 'sif'
env.CONTAINER_DEF = 'relion-gpu-build.def'
env.CONTAINER_DIR = 'container'
env.APP_VER = 'v3.1'
env.SINGULARITY_BIN = '/usr/bin/singularity'

// Define the Software Pipeline
node('gpu') {

    stage ('Checkout code') {checkout scm}
         
    stage('Build') {
      if (fileExists (CONTAINER_DIR)) {
             echo 'Directory exists'
             } else {
             sh "mkdir $CONTAINER_DIR" 
           }
       // Running with --notest as 'singularity build ' does not feature the --nv for GPU and executes %test scriptlet during the build.
      sh "sudo $SINGULARITY_BIN build --notest $CONTAINER_DIR/$APP_VER-$CONTAINER_NAME-$BUILD_NUMBER.$CONTAINER_FMT $CONTAINER_DEF"
    }

    stage('Container Cleanup') {
      // Cleaning up unwanted files from the container.
      sh "$SINGULARITY_BIN cache clean -f --name $CONTAINER_DIR/$APP_VER-$CONTAINER_NAME-$BUILD_NUMBER.$CONTAINER_FMT"
    }

    stage('Running Tests') {
      // Execute the %test scriptlet.
      sh "$SINGULARITY_BIN test --nv $CONTAINER_DIR/$APP_VER-$CONTAINER_NAME-$BUILD_NUMBER.$CONTAINER_FMT "
    }

    stage('Deliver HPC software to repository') {
      // Make application available to HPC users 
      sh "cp $CONTAINER_DIR/$APP_VER-$CONTAINER_NAME-$BUILD_NUMBER.$CONTAINER_FMT $SW_LOCATION"
      dir(SW_LOCATION) {
         sh "ln -sf $SW_LOCATION/$APP_VER-$CONTAINER_NAME-$BUILD_NUMBER.$CONTAINER_FMT $CONTAINER_NAME-$APP_VER.$CONTAINER_FMT"
      }
      echo "Generating software environment module file"
sh label: '', script: '''
(
cat <<MODULE_FILE
#%Module######################################################################
##
##     Relion 
##
proc ModulesHelp { } {
    puts stderr "Sets up the paths you need to run Relion Container"
}

set base       /opt/exp_soft/singularity-containers
set basepath   \$base

set-alias relion "singularity exec -B /scratch:/scratch \$basepath/relion/relion-gpu-v3.1.sif \$@"
set modname     [module-info name]
set modmode     [module-info mode]

prepend-path PATH \$basepath
MODULE_FILE
) > relionv3.1.module
'''
    }   
}

