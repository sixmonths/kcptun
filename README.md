# <img src="logo.png" alt="kcptun" height="60px" /> 
[![Release][1]][2] [![Powered][7]][8] [![Build Status][3]][4] [![Go Report Card][5]][6]
[1]: https://img.shields.io/github/release/xtaci/kcptun.svg
[2]: https://github.com/xtaci/kcptun/releases/latest
[3]: https://travis-ci.org/xtaci/kcptun.svg?branch=master
[4]: https://travis-ci.org/xtaci/kcptun
[5]: https://goreportcard.com/badge/github.com/xtaci/kcptun
[6]: https://goreportcard.com/report/github.com/xtaci/kcptun
[7]: https://img.shields.io/badge/KCP-Powered-blue.svg
[8]: https://github.com/skywind3000/kcp

<img src="kcptun.png" alt="kcptun" height="200px"/>

### QuickStart
Download precompiled [Releases](https://github.com/xtaci/kcptun/releases).

```
KCP Client: ./client_darwin_amd64 -r "SERVER_IP:4000" -l ":8388" -mode fast2
KCP Server: ./server_linux_amd64 -t "TARGET_IP:8388" -l ":4000" -mode fast2
```
The above command can establish 8388/tcp port forwarding (through 4000/udp port).


### Performance
<img src="fast.png" alt="fast.com" height="256px" />       

### Usage
Help output under MacOS X:
```
$ ./client_darwin_amd64 -h
NAME:
   kcptun - client(with SMUX)

USAGE:
   client_darwin_amd64 [global options] command [command options] [arguments...]

VERSION:
   20170120

COMMANDS:
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --localaddr value, -l value      local listen address (default: ":12948")
   --remoteaddr value, -r value     kcp server address (default: "vps:29900")
   --key value                      pre-shared secret between client and server (default: "it's a secrect") [$KCPTUN_KEY]
   --crypt value                    aes, aes-128, aes-192, salsa20, blowfish, twofish, cast5, 3des, tea, xtea, xor, none (default: "aes")
   --mode value                     profiles: fast3, fast2, fast, normal (default: "fast")
   --conn value                     set num of UDP connections to server (default: 1)
   --autoexpire value               set auto expiration time(in seconds) for a single UDP connection, 0 to disable (default: 0)
   --mtu value                      set maximum transmission unit for UDP packets (default: 1350)
   --sndwnd value                   set send window size(num of packets) (default: 128)
   --rcvwnd value                   set receive window size(num of packets) (default: 512)
   --datashard value, --ds value    set reed-solomon erasure coding - datashard (default: 10)
   --parityshard value, --ps value  set reed-solomon erasure coding - parityshard (default: 3)
   --dscp value                     set DSCP(6bit) (default: 0)
   --nocomp                         disable compression
   --snmplog value                  collect snmp to file, aware of timeformat in golang, like: ./snmp-20060102.log
   --snmpperiod value               snmp collect period, in seconds (default: 60)
   --log value                      specify a log file to output, default goes to stderr
   -c value                         config from json file, which will override the command from shell
   --help, -h                       show help
   --version, -v                    print the version

$ ./server_darwin_amd64 -h
NAME:
   kcptun - server(with SMUX)

USAGE:
   server_darwin_amd64 [global options] command [command options] [arguments...]

VERSION:
   20170120

COMMANDS:
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --listen value, -l value         kcp server listen address (default: ":29900")
   --target value, -t value         target server address (default: "127.0.0.1:12948")
   --key value                      pre-shared secret between client and server (default: "it's a secrect") [$KCPTUN_KEY]
   --crypt value                    aes, aes-128, aes-192, salsa20, blowfish, twofish, cast5, 3des, tea, xtea, xor, none (default: "aes")
   --mode value                     profiles: fast3, fast2, fast, normal (default: "fast")
   --mtu value                      set maximum transmission unit for UDP packets (default: 1350)
   --sndwnd value                   set send window size(num of packets) (default: 1024)
   --rcvwnd value                   set receive window size(num of packets) (default: 1024)
   --datashard value, --ds value    set reed-solomon erasure coding - datashard (default: 10)
   --parityshard value, --ps value  set reed-solomon erasure coding - parityshard (default: 3)
   --dscp value                     set DSCP(6bit) (default: 0)
   --nocomp                         disable compression
   --snmplog value                  collect snmp to file, aware of timeformat in golang, like: ./snmp-20060102.log
   --snmpperiod value               snmp collect period, in seconds (default: 60)
   --log value                      specify a log file to output, default goes to stderr
   -c value                         config from json file, which will override the command from shell
   --help, -h                       show help
   --version, -v                    print the version
```

#### Parameters by Layers

<p align="left"><img src="layeredparams.png" alt="params" height="450px"/></p>

***Both sides must agree on the following parameters:***
* datashard
* parityshard
* nocomp
* key
* crypt

other parameters can be set independently.

*How to optimize*：
> Step 1：Increase client `-rcvwnd` & server `-sndwnd` simultaneously & gradually。       
> Step 2：Try download something and watch network IO to see if it helps.     

***NOTICE: if too much retranmission happens, it's quite possible the windows are too large***

### Embeded Mode

Aggresiveness On Retransmission:

*fast3 > fast2 > fast > normal > default*    

Manual control is supported with hidden parameters, you must understand KCP protocol before doing this. eg:

```
 -mode manual -nodelay 1 -resend 2 -nc 1 -interval 20
```

### Forward Error Correction

In coding theory, the Reed–Solomon code belongs to the class of non-binary cyclic error-correcting codes. The Reed–Solomon code is based on univariate polynomials over finite fields.

It is able to detect and correct multiple symbol errors. By adding t check symbols to the data, a Reed–Solomon code can detect any combination of up to t erroneous symbols, or correct up to ⌊t/2⌋ symbols. As an erasure code, it can correct up to t known erasures, or it can detect and correct combinations of errors and erasures. Furthermore, Reed–Solomon codes are suitable as multiple-burst bit-error correcting codes, since a sequence of b + 1 consecutive bit errors can affect at most two symbols of size b. The choice of t is up to the designer of the code, and may be selected within wide limits.

![reed-solomon](rs.png)

Setting parameters of RS-Code with ```-datashard m -parityshard n```

### DSCP

Differentiated services or DiffServ is a computer networking architecture that specifies a simple, scalable and coarse-grained mechanism for classifying and managing network traffic and providing quality of service (QoS) on modern IP networks. DiffServ can, for example, be used to provide low-latency to critical network traffic such as voice or streaming media while providing simple best-effort service to non-critical services such as web traffic or file transfers.

DiffServ uses a 6-bit differentiated services code point (DSCP) in the 8-bit differentiated services field (DS field) in the IP header for packet classification purposes. The DS field and ECN field replace the outdated IPv4 TOS field.[1]

setting each side with ```-dscp value```.


### Security

No matter what encryption you are using for application layer, if you specify ```-crypt none``` to kcptun, 
the header will be ***PLAINTEXT*** to everyone; I suggest ```-crypt aes-128``` for encryption at least .

NOTICE: ```-crypt xor``` is also insecure, do not use this unless you know what you are doing.

### Memory Control

Routers, mobile devices are sensitive to memory consumption; by setting GOGC environment(eg: GOGC=20) will lower memory consumption.
Reference: https://blog.golang.org/go15gc

### Traffic Control

***Intended audience : for those server's bandwidth is quite limited.***      

Example: To limit outgoing bandwidth to 32mbit/s on server. 
```
root@kcptun:~# cat tc.sh
tc qdisc del dev eth0 root
tc qdisc add dev eth0 root handle 1: htb
tc class add dev eth0 parent 1: classid 1:1 htb rate 32mbit
tc filter add dev eth0 protocol ip parent 1:0 prio 1 handle 10 fw flowid 1:1
iptables -t mangle -A POSTROUTING -o eth0  -j MARK --set-mark 10
root@kcptun:~#
```

### Snappy Stream Compression

> Snappy is a compression/decompression library. It does not aim for maximum
> compression, or compatibility with any other compression library; instead,
> it aims for very high speeds and reasonable compression. For instance,
> compared to the fastest mode of zlib, Snappy is an order of magnitude faster
> for most inputs, but the resulting compressed files are anywhere from 20% to
> 100% bigger.

> Reference: http://google.github.io/snappy/

disable compression by setting ```-nocomp``` on both side.

> Tips: Turning off compression may reduce latency.

### SNMP

```go
// Snmp defines network statistics indicator
type Snmp struct {
	BytesSent        uint64 // payload bytes sent
	BytesReceived    uint64
	MaxConn          uint64
	ActiveOpens      uint64
	PassiveOpens     uint64
	CurrEstab        uint64
	InErrs           uint64
	InCsumErrors     uint64 // checksum errors
	InSegs           uint64
	OutSegs          uint64
	OutBytes         uint64 // udp bytes sent
	RetransSegs      uint64
	FastRetransSegs  uint64
	EarlyRetransSegs uint64
	LostSegs         uint64
	RepeatSegs       uint64
	FECRecovered     uint64
	FECErrs          uint64
	FECSegs          uint64 // fec segments received
}
```

Sending a signal by ```kill -SIGUSR1 pid``` will give SNMP information for KCP，useful for fine-grained adjustment.
Of which ```RetransSegs,FastRetransSegs,LostSegs,OutSegs``` is the most useful.

### References

1. https://github.com/skywind3000/kcp -- KCP - A Fast and Reliable ARQ Protocol.
2. https://github.com/klauspost/reedsolomon -- Reed-Solomon Erasure Coding in Go.
3. https://en.wikipedia.org/wiki/Differentiated_services -- DSCP.
4. http://google.github.io/snappy/ -- A fast compressor/decompressor.
5. https://www.backblaze.com/blog/reed-solomon/ -- Reed-Solomon Explained.
6. http://www.qualcomm.cn/products/raptorq -- RaptorQ Forward Error Correction Scheme for Object Delivery.
7. https://en.wikipedia.org/wiki/PBKDF2 -- Key stretching.
8. http://blog.appcanary.com/2016/encrypt-or-compress.html -- Should you encrypt or compress first?
9. https://github.com/hashicorp/yamux -- Connection multiplexing library.
10. https://tools.ietf.org/html/rfc6937 -- Proportional Rate Reduction for TCP.
11. https://tools.ietf.org/html/rfc5827 -- Early Retransmit for TCP and Stream Control Transmission Protocol (SCTP).
12. http://http2.github.io/ -- What is HTTP/2?
13. http://www.lartc.org/ -- Linux Advanced Routing & Traffic Control
