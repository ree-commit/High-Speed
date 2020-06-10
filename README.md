# High-Speed
HighSpeed TCP
What is High Speed TCP
1. Just like Standard TCP when cwnd is low
2. More aggressive than Standard TCP when cwnd is high.
3. important question is when is cwnd is low and at what point it
is going to differ with standard TCP.
4. if the size of cwnd is 38 segments then it is considered as low
cwnd and standard TCP is followed.
5. but if size of cwnd is more than 38 segments then HSTCP is
followed.

# Direct Code Execution (DCE) 
It is a module for ns-3 that provides facilities to execute, within ns-3, existing implementations of userspace and kernelspace network protocols or applications without source code changes.  For example instead of using the pseudo application provided by ns-3 V4PingHelper you can use the true ping.

To read more-

https://www.nsnam.org/docs/dce/manual/ns-3-dce-manual.pdf


https://dl.acm.org/doi/abs/10.1145/2535372.2535374


https://www.nsnam.org/docs/dce/manual/ns-3-dce-manual.pdf

# How to reproduce this project?

**Docker** is an application that simplifies the process of managing application processes in containers. Containers let you run your applications in resource-isolated processes. we install docker in Ubuntu 18.04 LTS.

>sudo apt install docker.io

**Install Docker container of using docker image of ns-3 DCE**

>sudo docker run -i -t thehajime/ns-3-dce /bin/bash

After installing docker follow the steps to setup DCE:
##### 1. Pull the docker image to install NS-3 DCE.

>docker pull thehajime/ns-3-dce

##### 2. Run docker to create a container

>sudo docker ps -a				*//for list of all containers*

>sudo docker start your_docker_name				*//start particular container*

>sudo docker exec -it your_docker_name /bin/bash				*//take you to particular container where you can find the ns-3 dce.*


##### 3. Install vim in docker container.
>sudo apt install vim

##### 4. Go to source
>cd source

- ns-3-dce				*// Direct code execution section*

- ns-3-dev				*// Development section*


##### 5. go to ns-3-dce and run following command to check weather install correctly or not(by running iperf)
>cd ns-3-dce

>./waf –run dce-iperf

*// If build successfully : You have correctly installed DCE!*


##### 6. Copy dumbbell topology in ns-3-dce/example

>sudo docker cp dumbbell-topology-ns3-receiver.cc your_docker_name:/home/ns3dce/dce-linux-dev/source/ns-3-dce/example

*// dumbbell-topology-ns3-receiver.cc file must be in local machines' home directory*

*// Run the above command from new terminal outside the container.*


##### 7. add parse_cwnd in /ns-3-dce/utils

>sudo docker cp parse_cwnd.py your_docker_name:/home/ns3dce/dce-linux-dev/source/ns-3-dce/utils

*// parse_cwnd.py file must be in local machines' home directory*

*// Run the above command from new terminal outside the container.*


##### 8. change wscript

*// Go to your wscript and make entry of this dumbell-topology-ns3-receiver file in example.*

>cd /home/ns3dce/dce-linux-dev/source/ns-3-dce

*// wscript will be present at above location.*


##### 9. Run the example topology on linux stack using highspeed TCP.

>./waf --run "dumbbelltopologyns3receiver --stack=linux -queue_disc_type=FifoQueueDisc --WindowScaling=true --Sack=true --stopTime=30 --delAckCount=1 --BQL=true --linux_prot=highspeed"			

*// This will run the dumbell topology on linux stack.*


##### 10. Run parse_cwnd.py inside ns-3-dce/utils.

>python parse cwnd.py 2 2

*// Running this file will collect the traces generated by running the topology example on linux stack. This will
create a folder ’cwnd data’ inside ns-3-dce/result/dumbbell-topology where you will find A-linux.plotme file.*


##### 11. create a folder ”overlapped” inside ns-3-dce/result/dumbell-topology.

>sudo mkdir overlapped


##### 12. copy the A-linux.plotme file from cwnd data to overlapped folder.

>cp cwnd data/A-linux.plotme overlapped/				*// inside the docker container.*


##### 13. Run the example topolgy on ns-3 stack using TcpHighSpeed.

>./waf --run "dumbbelltopologyns3receiver --stack=ns3 -queue_disc_type=FifoQueueDisc --WindowScaling=true --Sack=true --stopTime=300 --delAckCount=1 --BQL=true --transport_prot=TcpHighSpeed "

*// Run this from source/ns-3-dce itself.*
*// This will create a new timestamp folder under ns-3-dce/result/dumbbell-topology/.*


##### 14. Under the latest timestamp folder, you will find a folder ”cwndTraces” where A-ns3.plotme file will be generated. Copy this file in overlapped folder.


##### 15. Overlapped will now have A-ns3.plotme and A-linux.plotme now copy the gnuplotscriptCwnd script insideoverlapped and install gnuplot.

>sudo apt-get install gnuplot
>gnuplot overlapgnuplotscriptCwnd
*// Above steps will create a CwndA.png file inside overlapped.*


##### 16. copy CwndA.png to Home.
>sudo docker cp your_docker_name:/home/ns3dce/dce-linux-dev/source/ns-3-dce/results/dumbbell-topology/overlapped/Cwnd .

*// Now or result(overlapped graph)is in local machines' home directory.*

# Update dev 

#####  optional
*//If you want to take backup of your working dev,copy dev to other location and remove it. 

>rm -r -f ns-3-dev/

**Clone given version of ns-3-dev from the link below.**

>https://gitlab.com/apoorvabhargava/ns-3-dev/

**Go to ns-3-dev/ and switch to the LinuxPrrMergeRequest branch.**

>git checkout LinuxPRRMergeRequest


##### 1. Go to
>cd /source/ns-3-dev/src/internet/model


##### 2. open TcpHighSpeed.cc and make changes and save using vim

> vi TcpHighSpeed.cc


##### 3. Whenever you make any changes in DEV, you need to build the dce, otherwise the changes will not be reflected in your setup.
*// Add dumbbell-topology-ns3-receiver.cc in the example, add parse_cwnd in /ns-3-dce/utils , and change wscript, etc.*

**Then Go to “ns3dce@4c788588614e:~/dce-linux-dev$” and set environment variables :**
>export BAKE_HOME=\`pwd\`/bake

>export PATH=$PATH:$BAKE_HOME

>export PYTHONPATH=$PYTHONPATH:$BAKE_HOME

**Configure DCE :**

>bake.py configure -e dce-linux-dev

**Build DCE :**

>bake.py build -vvv


##### 5. [Follow previous set of commands](#how-to-reproduce-this-project?) 

*// if you wish to use default recovery-
>use –recovery=TcpPrrRecovery while executing topology.

