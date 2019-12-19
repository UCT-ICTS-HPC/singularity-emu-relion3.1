// Environment Variables
env.SW_LOCATION = '/opt/exp_soft/singularity-containers/relion'
env.CONTAINER_NAME = 'relion-gpu'
env.CONTAINER_FMT = '.sif'
env.CONTAINER_DEF = 'relion-gpu-build.def'
env.CONTAINER_DIR = 'container'
env.TF_VER = 'v3'
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
      sh "sudo $SINGULARITY_BIN build --notest $CONTAINER_DIR/$TF_VER-$CONTAINER_NAME-$BUILD_NUMBER $CONTAINER_DEF"
    }

    stage('Container Cleanup') {
      // Cleaning up unwanted files from the container.
      sh "$SINGULARITY_BIN cache clean --name $CONTAINER_DIR/$TF_VER-$CONTAINER_NAME-$BUILD_NUMBER"
    }

    stage('Running Tests') {
      // Execute the %test scriptlet.
      sh "$SINGULARITY_BIN test --nv $CONTAINER_DIR/$TF_VER-$CONTAINER_NAME-$BUILD_NUMBER "
    }

    stage('Deliver HPC software to repository') {
      // Make application available to HPC users 
      sh "cp $CONTAINER_DIR/$TF_VER-$CONTAINER_NAME-$BUILD_NUMBER $SW_LOCATION"
      dir(SW_LOCATION) {
         sh "ln -sf $SW_LOCATION/$TF_VER-$CONTAINER_NAME-$BUILD_NUMBER tensorflow-$TF_VER-gpu.sif"
      }
      echo "Generating software environment module file"
    }   
}