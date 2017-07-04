# Prerequisites
To build and use the traffic generator we need GCC, python interpreter and gnuplot on the linux end-host

# Generator Components
The server program is set to listen for incoming requests on a given port (i.e., 5001) and when the server is not loaded, the connection is accepted and a dummy flow of packets are generated that closely match the requested size for each request.

The client program establishes either a persistent or non-persistent TCP connections to a given list of servers. Then, it will randomly generate flow requests to the server where request sizes and/or inter-arrival times follow a given distribution. 

Note that, if the client finds all TCP connections from the pool of open (persistent) connections are busy, then the client will establish a new one. In addition, each request only consists of one flow.  

Configuration Files:
1- Destination servers: the user can set the list of destination servers for One-to-All scenarios in the conf/client_config_one* file and for All-to-All scenarios in the conf/client_config_all* file
2- Request Sizes: the user can set the CDF of the request size distribution in the conf/worload_SIZE_CDF file.
3- Inter Arrival Times: the user can set the CDF of the request size distribution in the conf/worload_INTER_CDF file.


# Installation steps

change your current directory to to where the source and Makefile is located then issue:

```
git clone https://github.com/ahmedcs/Traffic_Generator.git
cd Traffic_Generator
make
```
The result is that the following program files in the **bin** directory: **client**, **server** and all the necessary python and shell scripts for running the experiments.    

In the following, we give the necessary details to run the experiments both the abstract and detailed way

## Running the experiments (the abstract way)

### Example 1: One-to-All communication pattern of websearch
```
make; python bin/run_all_to_one.py -c conf/client_config_oneWEB.txt -i 1631 -b 7000 -n 1000 -a 172.16.0.1:8001 -CC dctcp;
```

This will run an experiment of websearch workload where:
1- (-b 7000) the aggregate arrival rate of the clients that is 7000 Mbps, if inter arrival distribution is present this option is ignored
2- (-i 1631) this is the job id given to the folder which collect the results (the folder is named jobo\_*jobid* for One-to-All and joba\_*jobid* for All-to-All)
3- (-n 1000) each of the client will run a 1000 request 
4- (-a 172.16.0.1:8001) the master server which collects the results are listening at -a 172.16.0.1:8001
5- (-CC dctcp) the congestion control protocol to be used by all end-hosts during the experiment, in this case dctcp is used and for using dctcp you need an ECN-enabled switches in the network with a DCTCP AQM.
 
### Example 1: All-to-All communication pattern of websearch
```
make; python bin/run_all_to_all.py -c conf/client_config_allDATA.txt -i 2631 -b 7000 -n 1000 -a 172.16.0.1:8001 -CC cubic;
```

This will run an experiment of websearch workload where:
1- (-b 7000) the aggregate arrival rate of the clients that is 7000 Mbps, if inter arrival distribution is present this option is ignored
2- (-i 2631) this is the job id given to the folder which collect the results (the folder is named jobo\_*jobid* for One-to-All and joba\_*jobid* for All-to-All)
3- (-n 1000) each of the client will run a 1000 request 
4- (-a 172.16.0.1:8001) the master server which collects the results are listening at -a 172.16.0.1:8001
5- (-CC cubic) the congestion control protocol to be used by all end-hosts during the experiment, in this case dctcp is used and for using dctcp you need an ECN-enabled switches in the network with a DCTCP AQM.
 
### Full Options of the python script

```
  -h, --help            show this help message and exit
  -i ID, --id ID        ID of joba (required)
  -b BANDWIDTH, --bandwidth BANDWIDTH
                        expected per-host RX bandwidth (Mbps) (required)
  -c CONF, --conf CONF  configuration file name (required)
  -n NUMBER, --number NUMBER
                        number of per-host requests (instead of -t, required
                        if -t not supplied)
  -t TIME, --time TIME  time in seconds to generate requests (instead of -n,
                        required if -n not supplied)
  -s SEED, --seed SEED  seed to generate random numbers (not required,
                        default=123456)
  -a ADDRESS, --address ADDRESS
                        address of master node (IP:port) (required)
  -g GETLOG, --getlog GETLOG
                        get the logs of printf 0 or 1 (not required,
                        default=0)
  -m METHOD, --method METHOD
                        get the method used for running tasks(not required,
                        either ssh or xmlrpc, default=xmlrpc)
  -w TOTWORKERS, --totworkers TOTWORKERS
                        total number of workers to use in the experiment (not
                        required, set to # of servers, default=28)
  -k PERSIST, --persist PERSIST
                        use persistent connection or not (not required,
                        default=non-persistent))
  -d PCAPDUMP, --pcapdump PCAPDUMP
                        use pcap tcpdump or not (not required, default is
                        disabled)
  -l LOSSPROBE, --lossprobe LOSSPROBE
                        Buffer size of loss probe module or not (not required,
                        default=0 to disable)
  -r RACK, --rack RACK  use RACK module or not (not required, default is
                        disabled)
  -D DROP, --DROP DROP  use NETEM Qdisc at all NICs to drop (not required,
                        default is disabled)
  -T TIME, --TIME TIME  Waiting time for ACKs before RACK taking action (not
                        required, default is 4ms)
  -M MAXACK, --MAXACK MAXACK
                        Maximum ACK no before considering the flow as elephant
                        (not required, default is 100000 which means no
                        elephant detection is enabled)
  -B BACKGROUND, --Background BACKGROUND
                        all to all background traffic (Mbps) using iperf
                        (optional)
  -I INCASTGUARD, --Incastguard INCASTGUARD
                        Use IncastGuard on the NetFPGA switch (optional)
  -SI SCALEINT, --ScaleInt SCALEINT
                        Scale factor of request interarrivals (optional,
                        default 1.0)
  -SS SCALESIZE, --ScaleSize SCALESIZE
                        Scale factor of request size (optional, default 1.0)
  -CC CONGESTIONCONTROL, --CongestionControl CONGESTIONCONTROL
                        Congestion Control Protocol to be used (optional,
                        default Reno)
  -dt DATE, --date DATE
                        Date of experiment to be used for naming main folder
                        (optional, default today date)
  -HS HSCC, --HSCC HSCC
                        Enable HSCC switch (optional, default disabled)
```

## Running the experiments (the detailed way)
For example to run a websearch workload experiment with rate of 900 Mbps, 5000 flows and seed of 123, go to the main directory and run the following command:
- Start server 
```
./bin/server -p 5001 -d
```

- Start client
```
./bin/client -b 900 -c conf/client_config.txt -n 5000 -l flows.txt -s 123 -r bin/result.py
```

- Start incast-client
```
./bin/incast-client -b 900 -c conf/client_config_allWEB.txt -n 5000 -l log -s 123 -r bin/result.py
```

### Explaining the Command Line Arguments
```
./bin/server -p 5001 -d  
```
* **-p** : the TCP **port** that the server listens on (default 5001)

* **-v** : give more detailed output (**verbose**)

* **-d** : run the server in the daemon mode which means it does not keep the shell busy

* **-h** : display the help information about the server

```
./bin/client -b 900 -c conf/client_config.txt -n 5000 -l flows.txt -s 123 -r bin/result.py
```
* **-b** : the target average arrival rate in Mbps
 
* **-c** : the path to the configuration file which specifies workload characteristics

* **-n** : the total number of requests to be generated by a single client

* **-t** : the total time in seconds to keep generating requests (either use this option or use -n)
 
* **-l** : the log file to which the flow information (e.g., FCT, size, timeouts, etc) is dumbed (default flows.txt)

* **-s** : the seed value to be used by the random number generators (by default the current system time is used, but if same pattern of requests is required for comparison, please use this option for instance -s 123456)

* **-r** : the python script used to parse the output result files to generate the aggregate statistics

* **-v** : this option will give more detailed output

* **-h** : this option will display the above help information

## Client Configuration File
The client configuration file specifies the list of servers, the request size distribution, the Differentiated Services Code Point (DSCP) value distribution, the sending rate distribution and the request fanout distribution (only for **incast-client**). We provide several client configuration files as examples in ./conf directory.  

The format is a sequence of key and value(s), one key per line. The permitted keys are:

 **server:** IP address and TCP port of a server.
```
server 172.21.0.1 5001
```

* **req_size_dist:** request size distribution file path and name.
```
req_size_dist conf/Websearch_SIZE_CDF.txt
```
There must be one request size distribution file, present at the given path, 
which specifies the CDF of the request size distribution. See "Websearch\_SIZE\_CDF.txt" in ./conf directory 
for an example with proper formatting.

* **req_inter_dist:** inter arrival time distribution file path and name.
```
req_inter_dist conf/Websearch_INTER_CDF.txt
```
The inter arrival time distribution file is optional but if given it must be present at the given path, 
which specifies the CDF of the arrival time distribution. See "Websearch\_INTER\_CDF" in ./conf directory 
for an example with proper formatting.

Note that, if the inter arrival distribution is not give, a Poisson arrival process is generated to create the target arrival rate (load) given by option (-b)

##Output
After the client program finish all the requests and receive the responses, the program creates a file containing the response times of each request. 

In the result file, each line gives flow size (in bytes), flow completion time (in microseconds), DSCP value, desired sending rate (in Mbps) and actual per-flow goodput (in Mbps). 
