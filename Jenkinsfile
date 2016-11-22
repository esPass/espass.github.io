
node {

 stage 'checkout'
  checkout scm

 stage 'bootstrap'
  sh "./script/bootstrap"

 stage 'cibuild'
  sh "./script/cibuild"
     
}