1..
set ns [new Simulator]

set tf [open 1.tr w]
$ns trace-all $tf

set nf [open 1.nam w]
$ns namtrace-all $nf

#$ns color 1 Blue
#$ns color 2 Red

set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]

$ns duplex-link $n0 $n2 2Mb 2ms DropTail
$ns duplex-link $n1 $n2 2Mb 20ms DropTail
$ns duplex-link $n2 $n3 2Mb 2ms DropTail
$ns queue-limit $n2 $n3 10

set udp1 [new Agent/UDP]
$ns attach-agent $n0 $udp1

set null1 [new Agent/Null]
$ns attach-agent $n3 $null1

$ns connect $udp1 $null1

set cbr1 [new Application/Traffic/CBR]
$cbr1 attach-agent $udp1

$ns at 1.1 "$cbr1 start"

set tcp1 [new Agent/TCP]
$ns attach-agent $n1 $tcp1

set sink1 [new Agent/TCPSink]
$ns attach-agent $n3 $sink1

$ns connect $tcp1 $sink1

set ftp1 [new Application/FTP]
$ftp1 attach-agent $tcp1

$ns at 0.1 "$ftp1 start"
$ns at 0.8 "$ftp1 stop"
$ns at 10.0 "finish"


proc finish {} {
global ns tf nf
$ns flush-trace
close $tf
close $nf
puts "Running nam file..."

exec nam 1.nam &
exit 0
}

$ns run

1.. awk ---------------------

BEGIN {
tcp_count=0;
udo_count=0;
}{
if($1 == "-" && $5 == "tcp")
tcp_count++;
if($1 == "+" && $5 == "cbr")
udp_count++;
} END {
printf("TCP %d\n",tcp_count);
printf("UDP %d\n",udp_count);
}

----------------------------------------------------------------------------------

2....

set ns [new Simulator]
set tf [open tf2.tr w]
set nf [open nf2.nam w]
$ns trace-all $tf
$ns namtrace-all $nf
set cwind [open win2.tr w]
set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]
$ns duplex-link $n0 $n2 2Mb 2ms DropTail
$ns duplex-link $n1 $n2 2Mb 2ms DropTail
$ns duplex-link $n2 $n3 0.4Mb 5ms DropTail
$ns duplex-link $n3 $n4 2Mb 2ms DropTail
$ns duplex-link $n3 $n5 2Mb 2ms DropTail
$ns queue-limit $n2 $n3 10
set tcp1 [new Agent/TCP]
set sink1 [new Agent/TCPSink]
set ftp1 [new Application/FTP]
$ns attach-agent $n0 $tcp1
$ns attach-agent $n5 $sink1
$ns connect $tcp1 $sink1
$ftp1 attach-agent $tcp1
$ns at 0.1 "$ftp1 start"

set tcp2 [new Agent/TCP]
set sink2 [new Agent/TCPSink]
set telnet1 [new Application/Telnet]
$ns attach-agent $n1 $tcp2
$ns attach-agent $n4 $sink2
$ns connect $tcp2 $sink2
$telnet1 attach-agent $tcp2
$ns at 1.1 "$telnet1 start"
$ns at 1.0 "$ftp1 stop"
#$ns at 4.0 "$telnet1 stop"
$ns at 2.0 "finish"
proc plotWindow {tcpsource file} {
global ns
set time 0.01
set now [$ns now]
set cwind [$tcpsource set cwnd_]
puts $file "$now $cwind"
$ns at [expr $now + $time] "plotWindow $tcpsource $file" 
}
$ns at 0.2 "plotWindow $tcp1 $cwind"
$ns at 0.5 "plotWindow $tcp2 $cwind"
proc finish {} {
global ns tf nf
$ns flush-trace
close $tf
close $nf
puts "Running nam..."
exec nam nf2.nam &
exec xgraph win2.tr &
exit 0 
}
$ns run

-----------------------------------------------------------------------------

3..... 

set ns [new Simulator]
set tf [open ex3.tr w]
$ns trace-all $tf
set nf [open ex3.nam w]
$ns namtrace-all $nf
set cwind [open win3.tr w]

$ns color 1 Blue
$ns color 2 Red

$ns rtproto DV

set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]

$ns duplex-link $n0 $n1 0.3Mb 10ms DropTail
$ns duplex-link $n0 $n2 0.3Mb 10ms DropTail
$ns duplex-link $n2 $n3 0.3Mb 10ms DropTail
$ns duplex-link $n1 $n4 0.3Mb 10ms DropTail
$ns duplex-link $n3 $n5 0.5Mb 10ms DropTail
$ns duplex-link $n4 $n5 0.5Mb 10ms DropTail

$ns duplex-link-op $n0 $n1 orient right-up
$ns duplex-link-op $n0 $n2 orient right-down
$ns duplex-link-op $n2 $n3 orient right
$ns duplex-link-op $n1 $n4 orient right
$ns duplex-link-op $n3 $n5 orient right-up
$ns duplex-link-op $n4 $n5 orient right-down

set tcp [new Agent/TCP]
$ns attach-agent $n0 $tcp
set sink [new Agent/TCPSink]
$ns attach-agent $n4 $sink
$ns connect $tcp $sink
$tcp set fid_ 1
set ftp [new Application/FTP]
$ftp attach-agent $tcp

$ns rtmodel-at 1.0 down $n1 $n4
$ns rtmodel-at 3.0 up $n1 $n4
$ns at 0.1 "$ftp start"
$ns at 12.0 "finish"

proc plotWindow {tcpSource file} {
global ns
set time 0.01
set now [$ns now]
set cwnd [$tcpSource set cwnd_]
puts $file "$now $cwnd"
$ns at [expr $now + $time] "plotWindow $tcpSource $file"
}

$ns at 1.0 "plotWindow $tcp $cwind"

proc finish {} {
global ns tf nf cwind
$ns flush-trace
close $tf
close $nf
exec nam ex3.nam &
exec xgraph win3.tr &
exit 0
}
$ns run


----------------------------------------------
4........

set ns [new Simulator]
set tf [open 4.tr w]
$ns trace-all $tf
set nf [open 4.nam w]
$ns namtrace-all $nf

set n0 [$ns node]
set n1 [$ns node]

$ns color 1 Blue

$n0 label "Server"
$n1 label "Client"

$ns duplex-link $n0 $n1 10Mb 22ms DropTail
$ns duplex-link-op $n0 $n1 orient right

set tcp [new Agent/TCP]
$ns attach-agent $n0 $tcp
$tcp set packetSize_ 1500
set sink [new Agent/TCPSink]
$ns attach-agent $n1 $sink
$ns connect $tcp $sink
set ftp [new Application/FTP]
$ftp attach-agent $tcp
$tcp set fid_ 1

proc finish {} {
global ns tf nf
$ns flush-trace
close $tf
close $nf
exec nam 4.nam &
exec awk -f p4transfer.awk 4.tr &
exec awk -f p4convert.awk 4.tr > convert.tr
exec xgraph convert.tr -geometry 800*400 -t
"Bytes_received_at_Client" -x "Time _in_secs" -y "Bytes_in_bps" &
}

$ns at 0.01 "$ftp start"
$ns at 15.0 "$ftp stop"
$ns at 15.1 "finish"
$ns run

4.. convert.awk------------
BEGIN{
count=0;
time=0;
}
{
if($1=="r" && $4==1 && $5=="tcp")
{
count+=$6;
time=$2;
printf("\n%f\t%f",time,(count)/1000000);
}
} 
END{
}

4... transfer.awk ------------
BEGIN{
count=0;
time=0;
total_bytes_received=0;
total_bytes_sent=0;
}
{
if($1=="r" && $4==1 && $5=="tcp")
total_bytes_received+=$6;
if($1=="+" && $3==0 && $5=="tcp")
total_bytes_sent+=$6;
} 
END{
system("clear");
printf("\nTransmission time required to transfer the file is %f",$2);
printf("\nActual data sent from the server is %f Mbps",
(total_bytes_sent)/1000000);
printf("\nData received by the client is %f Mbps\n",
(total_bytes_received)/1000000);
}


----------------------------------------------------------------------------
5.....
#Create an event scheduler wit multicast turned on
set ns [new Simulator -multicast on]

$ns multicast


set tf [open mcast.tr w]
$ns trace-all $tf


set fd [open mcast.nam w]
$ns namtrace-all $fd


set n0 [$ns node]
set n1 [$ns node]
set n2 [$ns node]
set n3 [$ns node]
set n4 [$ns node]
set n5 [$ns node]
set n6 [$ns node]
set n7 [$ns node]


$ns duplex-link $n0 $n2 1.5Mb 10ms DropTail
$ns duplex-link $n1 $n2 1.5Mb 10ms DropTail
$ns duplex-link $n2 $n3 1.5Mb 10ms DropTail
$ns duplex-link $n3 $n4 1.5Mb 10ms DropTail
$ns duplex-link $n3 $n7 1.5Mb 10ms DropTail
$ns duplex-link $n4 $n5 1.5Mb 10ms DropTail
$ns duplex-link $n4 $n6 1.5Mb 10ms DropTail

# Routing protocol: say distance vector
#Protocols: CtrMcast, DM, ST, BST
set mproto DM
set mrthandle [$ns mrtproto $mproto {}]

set group1 [Node allocaddr]
set group2 [Node allocaddr]


set udp0 [new Agent/UDP]
$ns attach-agent $n0 $udp0
$udp0 set dst_addr_ $group1
$udp0 set dst_port_ 0
set cbr1 [new Application/Traffic/CBR]
$cbr1 attach-agent $udp0


set udp1 [new Agent/UDP]
$ns attach-agent $n1 $udp1
$udp1 set dst_addr_ $group2
$udp1 set dst_port_ 0
set cbr2 [new Application/Traffic/CBR]
$cbr2 attach-agent $udp1


set rcvr1 [new Agent/Null]
$ns attach-agent $n5 $rcvr1
$ns at 1.0 "$n5 join-group $rcvr1 $group1"

set rcvr2 [new Agent/Null]
$ns attach-agent $n6 $rcvr2
$ns at 1.5 "$n6 join-group $rcvr2 $group1"

set rcvr3 [new Agent/Null]
$ns attach-agent $n7 $rcvr3
$ns at 2.0 "$n7 join-group $rcvr3 $group1"

set rcvr4 [new Agent/Null]
$ns attach-agent $n5 $rcvr1
$ns at 2.5 "$n5 join-group $rcvr4 $group2"

set rcvr5 [new Agent/Null]
$ns attach-agent $n6 $rcvr2
$ns at 3.0 "$n6 join-group $rcvr5 $group2"

set rcvr6 [new Agent/Null]
$ns attach-agent $n7 $rcvr3
$ns at 3.5 "$n7 join-group $rcvr6 $group2"

$ns at 4.0 "$n5 leave-group $rcvr1 $group1"
$ns at 4.5 "$n6 leave-group $rcvr2 $group1"
$ns at 5.0 "$n7 leave-group $rcvr3 $group1"

$ns at 5.5 "$n5 leave-group $rcvr4 $group2"
$ns at 6.0 "$n6 leave-group $rcvr5 $group2"
$ns at 6.5 "$n7 leave-group $rcvr6 $group2"

$ns at 0.5 "$cbr1 start"
$ns at 9.5 "$cbr1 stop"
$ns at 0.5 "$cbr2 start"
$ns at 9.5 "$cbr2 stop"
$ns at 10.0 "finish"

proc finish {} {
global ns tf fd
$ns flush-trace
close $tf
close $fd
exec nam mcast.nam &
exit 0
}


#$udp0 set fid_ 1
#$n0 color red
$n0 label "Source 1"


#$udp1 set fid_ 2
#$n1 color green
$n1 label "Source 2"


$ns color 1 red
$ns color 2 green

$n5 label "Receiver 1"
$n5 color blue
$n6 label "Receiver 2"
$n6 color blue
$n7 label "Receiver 3"
$n7 color blue
$ns run

------------------------------------------------------------------------

6......

set val(chan) Channel/WirelessChannel
set val(prop) Propagation/TwoRayGround
set val(netif) Phy/WirelessPhy
set val(mac) Mac/802_11
set val(ifq) Queue/DropTail/PriQueue
set val(ll) LL
set val(ant) Antenna/OmniAntenna
set val(x) 500
set val(y) 500
set val(ifqlen) 50
set val(nn) 2
set val(stop) 20.0
set val(rp) DSDV

set ns_ [new Simulator]
set tf [open 6.tr w]
$ns_ trace-all $tf

set nf [open 6.nam w]
$ns_ namtrace-all-wireless $nf $val(x) $val(y)
set prop [new $val(prop)]
set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)
create-god $val(nn)


$ns_ node-config -adhocRouting $val(rp) \
-llType $val(ll) \
-macType $val(mac) \
-ifqType $val(ifq) \
-ifqLen $val(ifqlen) \
-antType $val(ant) \
-propType $val(prop) \
-phyType $val(netif) \
-channelType $val(chan) \
-topoInstance $topo \
-agentTrace ON \
-routeTrace ON \
-macTrace ON \

for { set i 0 } { $i < $val(nn) } { incr i } {
set node_($i) [$ns_ node]
$node_($i) random-motion 0

}

for { set i 0 } { $i < $val(nn) } { incr i } {
$ns_ initial_node_pos $node_($i) 60
}
$ns_ at 1.1 "$node_(0) setdest 310.0 10.0 20.0"
$ns_ at 1.1 "$node_(1) setdest 10.0 310.0 20.0"

set tcp0 [new Agent/TCP]

set sink0 [new Agent/TCPSink]
$ns_ attach-agent $node_(0) $tcp0
$ns_ attach-agent $node_(1) $sink0

$ns_ connect $tcp0 $sink0
set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0
$ns_ at 1.0 "$ftp0 start"
$ns_ at 18.0 "$ftp0 stop"
for { set i 0 } { $i < $val(nn) } { incr i } {
$ns_ at $val(stop) "$node_($i) reset";
}


puts "Starting simulation"


$ns_ at $val(stop) "finish"

proc finish {} {
    global ns_ tf nf
    $ns_ flush-trace
    close $tf
    close $nf
    exec nam 6.nam &
    exit 0
}

$ns_ run

------------------------------------------------------------------------

7........
set val(chan) Channel/WirelessChannel
set val(prop) Propagation/TwoRayGround
set val(netif) Phy/WirelessPhy
set val(mac) Mac/802_11
#set val(ifq) CMUPriQueue
set val(ifq) Queue/DropTail/PriQueue
set val(ll) LL
set val(ant) Antenna/OmniAntenna
set val(x) 500
set val(y) 400
set val(ifqlen) 50
set val(nn) 3
set val(stop) 60.0
set val(rp) AODV
set ns_ [new Simulator]
set tracefd [open 002.tr w]
$ns_ trace-all $tracefd
set namtrace [open 002.nam w]
$ns_ namtrace-all-wireless $namtrace $val(x) $val(y)
set prop [new $val(prop)]
set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)
create-god $val(nn)
#Node Configuration
$ns_ node-config -adhocRouting $val(rp) \
-llType $val(ll) \
-macType $val(mac) \
-ifqType $val(ifq) \
-ifqLen $val(ifqlen) \
-antType $val(ant) \
-propType $val(prop) \
-phyType $val(netif) \
 -channelType $val(chan) \
-topoInstance $topo \
-agentTrace ON \
-routerTrace ON \
-macTrace ON
#Creating Nodes
for {set i 0} {$i < $val(nn) } {incr i} { 
set node_($i) [$ns_ node]
$node_($i) random-motion 0
}
#Initial Positions of Nodes
$node_(0) set x_ 5.0
$node_(0) set y_ 5.0
$node_(0) set z_ 0.0
$node_(1) set x_ 490.0
$node_(1) set y_ 285.0
$node_(1) set z_ 0.0
$node_(2) set x_ 150.0
$node_(2) set y_ 240.0
$node_(2) set z_ 0.0
for {set i 0} {$i < $val(nn)} {incr i} {
$ns_ initial_node_pos $node_($i) 40
}
#Topology Design
$ns_ at 0.0 "$node_(0) setdest 450.0 285.0 30.0"
$ns_ at 0.0 "$node_(1) setdest 200.0 285.0 30.0"
$ns_ at 0.0 "$node_(2) setdest 1.0 285.0 30.0"
$ns_ at 25.0 "$node_(0) setdest 300.0 285.0 10.0"
$ns_ at 25.0 "$node_(2) setdest 100.0 285.0 10.0"
$ns_ at 40.0 "$node_(0) setdest 490.0 285.0 5.0"
$ns_ at 40.0 "$node_(2) setdest 1.0 285.0 5.0"
#Generating Traffic
set tcp0 [new Agent/TCP]
set sink0 [new Agent/TCPSink]
$ns_ attach-agent $node_(0) $tcp0
$ns_ attach-agent $node_(2) $sink0
$ns_ connect $tcp0 $sink0
set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0
$ns_ at 0.0 "$ftp0 start"
#Simulation Termination
for {set i 0} {$i < $val(nn) } {incr i} {
$ns_ at $val(stop) "$node_($i) reset";
}
$ns_ at $val(stop) "finish"
proc finish { } {
global tracefd namtrace ns_
$ns_ flush-trace
close $tracefd
close $namtrace
exec nam 002.nam &
exit 0}
$ns_ run


-----------------------------------------------------------------------------

8......

command-------

ns cbrgen.tcl
###ns cbrgen.tcl -type tcp -nn 25 -seed 2 -mc 10 -rate 10 > static


mobile->
cd setdest
set setdest
setdest
###setdest -v 1 -n 25 -p 10 -M 10 -t 100 -x 500 -y 500 > mobile

code--------
set val(chan) Channel/WirelessChannel
set val(prop) Propagation/TwoRayGround
set val(netif) Phy/WirelessPhy
set val(ll) LL
set val(mac) Mac/802_11
set val(ifq) Queue/DropTail/PriQueue
set val(ifqlen) 50
set val(ant) Antenna/OmniAntenna
set val(x) 500
set val(y) 500
set val(rp) AODV
set val(nn) 51
set val(stop) 100.0
set val(sc) "a3"
set val(cp) "a2"

set ns_ [new Simulator]
set tf [open 003.tr w]
$ns_ trace-all $tf
set nf [open 003.nam w]
$ns_ namtrace-all-wireless $nf $val(x) $val(y)

set prop [new $val(prop)]
set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)
set god_ [create-god $val(nn)]

$ns_ node-config -adhocRouting $val(rp)\
-llType $val(ll)\
-macType $val(mac)\
-channelType $val(chan)\
-propType $val(prop)\
-phyType $val(netif)\
-ifqLen $val(ifqlen)\
-ifqType $val(ifq)\
-antType $val(ant)\
-topoInstance $topo\
-routerTrace ON\
-macTrace ON\
-agentTrace ON


for { set i 0 } { $i < $val(nn) } { incr i } {
set node_($i) [$ns_ node]
$node_($i) random-motion 0
}


for { set i 0 } { $i<$val(nn) } { incr i } {
set xx [expr rand()*500]
set yy [expr rand()*500]
$node_($i) set X_ $xx
$node_($i) set Y_ $yy
}
for { set i 0 } { $i < $val(nn) } { incr i } {
$ns_ initial_node_pos $node_($i) 40
}
source $val(sc)
source $val(cp)
for { set i 0 } { $i < $val(nn) } { incr i } {
$ns_ at $val(stop) "$node_($i) reset";
}

$ns_ at $val(stop) "$ns_ halt";
$ns_ run

-----------------------------------------------------------------------------------

9..........

set val(chan) Channel/WirelessChannel
set val(prop) Propagation/TwoRayGround
set val(netif) Phy/WirelessPhy
set val(mac) Mac/802_11
set val(ifq) CMUPriQueue
set val(ll) LL
set val(ant) Antenna/OmniAntenna
set val(x) 700
set val(y) 700
set val(ifqlen) 50
set val(nn) 6
set val(stop) 60.0
set val(rp) DSR


set ns [new Simulator]
set tf [open 004.tr w]
$ns trace-all $tf
set nf [open 004.nam w]
$ns namtrace-all-wireless $nf $val(x) $val(y)
set prop [new $val(prop)]
set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)
set god_ [create-god $val(nn)]

$ns node-config -adhocRouting $val(rp) \
-llType $val(ll) \
-macType $val(mac) \
-ifqType $val(ifq) \
-ifqLen $val(ifqlen) \
-antType $val(ant) \
-propType $val(prop) \
-phyType $val(netif) \
-channelType $val(chan) \
-topoInstance $topo \
-agentTrace ON \
-routerTrace ON \
-macTrace ON


set n0 [$ns node]
$n0 random-motion 0
$n0 set X_ 150.0
$n0 set Y_ 300.0
$n0 set Z_ 0.0

set n1 [$ns node]
$n1 random-motion 0
$n1 set X_ 300.0
$n1 set Y_ 500.0
$n1 set Z_ 0.0

set n2 [$ns node]
$n2 random-motion 0
$n2 set X_ 500.0
$n2 set Y_ 500.0
$n2 set Z_ 0.0

set n3 [$ns node]
$n3 random-motion 0
$n3 set X_ 300.0
$n3 set Y_ 100.0
$n3 set Z_ 0.0

set n4 [$ns node]
$n4 random-motion 0
$n4 set X_ 500.0
$n4 set Y_ 100.0
$n4 set Z_ 0.0

set n5 [$ns node]
$n5 random-motion 0
$n5 set X_ 650.0
$n5 set Y_ 300.0
$n5 set Z_ 0.0



$ns initial_node_pos $n0 40
$ns initial_node_pos $n1 40
$ns initial_node_pos $n2 40
$ns initial_node_pos $n3 40
$ns initial_node_pos $n4 40
$ns initial_node_pos $n5 40



$ns at 1.0 "$n0 setdest 160.0 300.0 2.0"
$ns at 1.0 "$n1 setdest 310.0 150.0 2.0"
$ns at 1.0 "$n2 setdest 490.0 490.0 2.0"

$ns at 1.0 "$n3 setdest 300.0 120.0 2.0"
$ns at 1.0 "$n4 setdest 510.0 90.0 2.0"
$ns at 1.0 "$n5 setdest 640.0 290.0 2.0"
$ns at 4.0 "$n3 setdest 300.0 500.0 5.0"



set tcp0 [new Agent/TCP]
set sink0 [new Agent/TCPSink]
$ns attach-agent $n0 $tcp0
$ns attach-agent $n5 $sink0
$ns connect $tcp0 $sink0

set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0
$ns at 5.0 "$ftp0 start"
$ns at 60.0 "$ftp0 stop"


$ns at $val(stop) "$n0 reset";
$ns at $val(stop) "$n1 reset";
$ns at $val(stop) "$n2 reset";
$ns at $val(stop) "$n3 reset";
$ns at $val(stop) "$n4 reset";
$ns at $val(stop) "$n5 reset";


$ns at $val(stop) "finish"


proc finish {} {
global ns nf tf
$ns flush-trace
close $tf
close $nf
exec nam 004.nam &
exit 0
}

$ns run


--------------------------------------------------------------------------

10.....


set val(chan) Channel/WirelessChannel
set val(prop) Propagation/TwoRayGround
set val(netif) Phy/WirelessPhy
set val(mac) Mac/802_11
set val(ifq) Queue/DropTail/PriQueue
set val(ll) LL
set val(ant) Antenna/OmniAntenna
set val(x) 500
set val(y) 500
set val(ifqlen) 50
set val(nn) 5
set val(stop) 50.0
set val(rp) AODV


set ns_ [new Simulator]
set tracefd [open 005.tr w]
$ns_ trace-all $tracefd
set namtrace [open 005.nam w]
set cwind [open win2.tr w]
$ns_ namtrace-all-wireless $namtrace $val(x) $val(y)

set topo [new Topography]
$topo load_flatgrid $val(x) $val(y)
create-god $val(nn)


$ns_ node-config -adhocRouting $val(rp) \
-llType $val(ll) \
-macType $val(mac) \
-ifqType $val(ifq) \
-ifqLen $val(ifqlen) \
-antType $val(ant) \
-propType $val(prop) \
-phyType $val(netif) \
-channelType $val(chan) \
-topoInstance $topo \
-agentTrace ON \
-routerTrace ON \
-macTrace ON \
-IncomingErrProc "uniformErr" \
-OutgoingErrProc "uniformErr"


proc uniformErr {} {
    set err [new ErrorModel]
    $err unit pkt
    $err set rate_ 0.01
    return $err
}


for {set i 0} {$i < $val(nn)} {incr i} {
    set node_($i) [$ns_ node]
    $node_($i) random-motion 0
}


for {set i 0} {$i < $val(nn)} {incr i} {
    $ns_ initial_node_pos $node_($i) 40
}


$ns_ at 1.0 "$node_(0) setdest 10.0 10.0 50.0"
$ns_ at 1.0 "$node_(1) setdest 10.0 100.0 50.0"
$ns_ at 1.0 "$node_(4) setdest 50.0 50.0 50.0"
$ns_ at 1.0 "$node_(2) setdest 100.0 100.0 50.0"
$ns_ at 1.0 "$node_(3) setdest 100.0 10.0 50.0"

set tcp0 [new Agent/TCP]
set sink0 [new Agent/TCPSink]
$ns_ attach-agent $node_(0) $tcp0
$ns_ attach-agent $node_(2) $sink0
$ns_ connect $tcp0 $sink0
set ftp0 [new Application/FTP]
$ftp0 attach-agent $tcp0
$ns_ at 1.0 "$ftp0 start"
$ns_ at 50.0 "$ftp0 stop"


set tcp1 [new Agent/TCP]
set sink1 [new Agent/TCPSink]
$ns_ attach-agent $node_(1) $tcp1
$ns_ attach-agent $node_(2) $sink1
$ns_ connect $tcp1 $sink1
set ftp1 [new Application/FTP]
$ftp1 attach-agent $tcp1
$ns_ at 1.0 "$ftp1 start"
$ns_ at 50.0 "$ftp1 stop"

for {set i 0} {$i < $val(nn)} {incr i} {
    $ns_ at $val(stop) "$node_($i) reset"
}

$ns_ at $val(stop) "finish"
puts "Starting Simulation..."

proc plotWindow {tcpSource file} {
global ns_
set time 0.01 
set now [$ns_ now]
set cwnd [$tcpSource set cwnd_]
puts $file "$now $cwnd"
$ns_ at [expr $now+$time] "plotWindow $tcpSource $file"
} 
	
$ns_ at 2.0 "plotWindow $tcp0 $cwind"
#$ns_ at 5.0 "plotWindow $tcp1 $cwind"

proc finish {} {
global ns_ tracefd namtrace
$ns_ flush-trace
close $tracefd
close $namtrace
exec nam 005.nam &
exec xgraph win2.tr &
exit 0
}

$ns_ run

-----------------------------------------------------------------






