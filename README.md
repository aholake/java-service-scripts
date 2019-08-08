# java-service-scripts
Professional script for making jar app as a service on linux
```#!/bin/bash

###
# chkconfig: 345 20 80
# description: Java application service script
# processname: java
#
# Installation (CentOS):
# copy file to /etc/init.d
# chmod +x /etc/init.d/javaApp
# chkconfig --add /etc/init.d/javaApp
# chkconfig javaApp on
# 
# Installation (Ubuntu):
# copy file to /etc/init.d
# chmod +x /etc/init.d/javaApp
# update-rc.d javaApp defaults
#
#
# Usage: (as root)
# service javaApp start
# service javaApp stop
# service javaApp status
#
###

# The directory in which your application is installed
APPLICATION_DIR="/home/ec2-user/services/evoucher"
# The fat jar containing your application
APPLICATION_JAR="EVoucher.jar"
# The application argument such as -cluster -cluster-host ...
APPLICATION_ARGS=""
# vert.x options and system properties (-Dfoo=bar). 
VERTX_OPTS="-Xmx1g -Dspring.profiles.active=prd"
# The Java command to use to launch the application (must be java 8+)
JAVA=/usr/bin/java

# ***********************************************
OUT_FILE="${APPLICATION_DIR}"/out.log
RUNNING_PID="${APPLICATION_DIR}"/RUNNING_PID
# ***********************************************

# colors
red='\e[0;31m'
green='\e[0;32m'
yellow='\e[0;33m'
reset='\e[0m'

echoRed() { echo -e "${red}$1${reset}"; }
echoGreen() { echo -e "${green}$1${reset}"; }
echoYellow() { echo -e "${yellow}$1${reset}"; }

# Check whether the application is running.
# The check is pretty simple: open a running pid file and check that the process
# is alive.
isrunning() {
  # Check for running app
  if [ -f "$RUNNING_PID" ]; then
    proc=$(cat $RUNNING_PID);
    if /bin/ps --pid $proc 1>&2 >/dev/null;
    then
      return 0
    fi
  fi
  return 1
}

start() {
  if isrunning; then
    echoYellow "The Service application is already running"
    return 0
  fi

  pushd $APPLICATION_DIR > /dev/null
  nohup $JAVA $VERTX_OPTS -jar $APPLICATION_JAR $APPLICATION_ARGS > $OUT_FILE 2>&1 &
  echo $! > ${RUNNING_PID}
  popd > /dev/null

  if isrunning; then
    echoGreen "Service Application started"
    exit 0
  else
    echoRed "The Service Application has not started - check log"
    exit 3
  fi
}

restart() {
  echo "Restarting Service Application"
  stop
  start
}

stop() {
  echoYellow "Stopping Service Application"
  if isrunning; then
    kill `cat $RUNNING_PID`
    rm $RUNNING_PID
  fi
}

status() {
  if isrunning; then
    echoGreen "Service Application is running"
  else
    echoRed "Service Application is either stopped or inaccessible"
  fi
}

case "$1" in
start)
    start
;;

status)
   status
   exit 0
;;

stop)
    if isrunning; then
	stop
	exit 0
    else
	echoRed "Application not running"
    fi
;;

restart)
    stop
    start
;;

*)
    echo "Usage: $0 {status|start|stop|restart}"
    exit 1
esac
```
