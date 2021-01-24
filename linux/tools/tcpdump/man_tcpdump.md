# [Man page of TCPDUMP](https://www.tcpdump.org/manpages/tcpdump.1.html)

- [Man page of TCPDUMP](#man-page-of-tcpdump)
  - [NAME](#name)
  - [SYNOPSIS](#synopsis)
  - [DESCRIPTION](#description)
  - [OPTIONS](#options)
    - [`-A`](#-a)
    - [`-b`](#-b)
    - [`-B buffer_size` | `--buffer-size=buffer_size`](#-b-buffer_size----buffer-sizebuffer_size)
    - [`-c count`](#-c-count)
    - [`--count`](#--count)
    - [`-C file_size`](#-c-file_size)
    - [`-d`](#-d)
    - [`-dd`](#-dd)
    - [`-ddd`](#-ddd)
    - [`-D` | `--list-interfaces`](#-d----list-interfaces)
    - [`-e`](#-e)
    - [`-E`](#-e-1)
    - [`-f`](#-f)
    - [`-F file`](#-f-file)
    - [`-G rotate_seconds`](#-g-rotate_seconds)
    - [`-h` | `--help`](#-h----help)
    - [`--version`](#--version)
    - [`-H`](#-h)
    - [`-i interface` | `--interface=interface`](#-i-interface----interfaceinterface)
    - [`-I` | `--monitor-mode`](#-i----monitor-mode)
    - [`--immediate-mode`](#--immediate-mode)
    - [`-j tstamp_type` | `--time-stamp-type=tstamp_type`](#-j-tstamp_type----time-stamp-typetstamp_type)
    - [`-J` | `--list-time-stamp-types`](#-j----list-time-stamp-types)
    - [`--time-stamp-precision=tstamp_precision`](#--time-stamp-precisiontstamp_precision)
    - [`--micro` | `--nano`](#--micro----nano)
    - [`-K` | `--dont-verify-checksums`](#-k----dont-verify-checksums)
    - [`-l`](#-l)
    - [`-L` | `--list-data-link-types`](#-l----list-data-link-types)
    - [`-m module`](#-m-module)
    - [`-M secret`](#-m-secret)
    - [`-n`](#-n)
    - [`-N`](#-n-1)
    - [`-#` | `--number`](#-----number)
    - [`-O` | `--no-optimize`](#-o----no-optimize)
    - [`-p` | `--no-promiscuous-mode`](#-p----no-promiscuous-mode)
    - [`--print`](#--print)
    - [`-Q direction` | `--direction=direction`](#-q-direction----directiondirection)
    - [`-q`](#-q)
    - [`-r file`](#-r-file)
    - [`-S` | `--absolute-tcp-sequence-numbers`](#-s----absolute-tcp-sequence-numbers)
    - [`-s snaplen` | `--snapshot-length=snaplen`](#-s-snaplen----snapshot-lengthsnaplen)
    - [`-T type`](#-t-type)
    - [`-t`](#-t)
    - [`-tt`](#-tt)
    - [`-ttt`](#-ttt)
    - [`-tttt`](#-tttt)
    - [`-ttttt`](#-ttttt)
    - [`-u`](#-u)
    - [`-U` | `--packet-buffered`](#-u----packet-buffered)
    - [`-v`](#-v)
    - [`-vv`](#-vv)
    - [`-vvv`](#-vvv)
    - [`-V file`](#-v-file)
    - [`-w file`](#-w-file)
    - [`-W filecount`](#-w-filecount)
    - [`-x`](#-x)
    - [`-xx`](#-xx)
    - [`-X`](#-x-1)
    - [`-XX`](#-xx-1)
    - [`-y datalinktype` | `--linktype=datalinktype`](#-y-datalinktype----linktypedatalinktype)
    - [`-z postrotate-command`](#-z-postrotate-command)
    - [`-Z user` | `--relinquish-privileges=user`](#-z-user----relinquish-privilegesuser)
    - [`expression`](#expression)
  - [EXAMPLES](#examples)
    - [To print all packets arriving at or departing from sundown](#to-print-all-packets-arriving-at-or-departing-from-sundown)
    - [To print traffic between helios and either hot or ace](#to-print-traffic-between-helios-and-either-hot-or-ace)
    - [To print all IP packets between ace and any host except helios](#to-print-all-ip-packets-between-ace-and-any-host-except-helios)
    - [To print all traffic between local hosts and hosts at Berkeley](#to-print-all-traffic-between-local-hosts-and-hosts-at-berkeley)
    - [To print all ftp traffic through internet gateway snup](#to-print-all-ftp-traffic-through-internet-gateway-snup)
    - [To print traffic neither sourced from nor destined for local hosts](#to-print-traffic-neither-sourced-from-nor-destined-for-local-hosts)
    - [To print the start and end packets (the SYN and FIN packets) of each TCP conversation that involves a non-local host](#to-print-the-start-and-end-packets-the-syn-and-fin-packets-of-each-tcp-conversation-that-involves-a-non-local-host)
    - [To print the TCP packets with flags RST and ACK both set.](#to-print-the-tcp-packets-with-flags-rst-and-ack-both-set)
    - [To print all IPv4 HTTP packets to and from port 80, i.e. print only packets that contain data, not, for example, SYN and FIN packets and ACK-only packets](#to-print-all-ipv4-http-packets-to-and-from-port-80-ie-print-only-packets-that-contain-data-not-for-example-syn-and-fin-packets-and-ack-only-packets)
    - [To print IP packets longer than 576 bytes sent through gateway snup](#to-print-ip-packets-longer-than-576-bytes-sent-through-gateway-snup)
    - [To print IP broadcast or multicast packets that were not sent via Ethernet broadcast or multicast](#to-print-ip-broadcast-or-multicast-packets-that-were-not-sent-via-ethernet-broadcast-or-multicast)
    - [To print all ICMP packets that are not echo requests/replies (i.e., not ping packets)](#to-print-all-icmp-packets-that-are-not-echo-requestsreplies-ie-not-ping-packets)
  - [OUTPUT FORMAT](#output-format)
    - [Timestamps](#timestamps)
    - [Link Level Headers](#link-level-headers)
    - [ARP/RARP Packets](#arprarp-packets)
    - [IPv4 Packets](#ipv4-packets)
    - [TCP Packets](#tcp-packets)

## NAME

`tcpdump` - dump traffic on a network  

## SYNOPSIS

    tcpdump [ -AbdDefhHIJKlLnNOpqStuUvxX# ] [ -B buffer_size ]
    
             [ -c count ] [ --count ] [ -C file_size ]
             [ -E spi@ipaddr algo:secret,... ]
             [ -F file ] [ -G rotate_seconds ] [ -i interface ]
             [ --immediate-mode ] [ -j tstamp_type ] [ -m module ]
             [ -M secret ] [ --number ] [ --print ] [ -Q in|out|inout ]
             [ -r file ] [ -s snaplen ] [ -T type ] [ --version ]
             [ -V file ] [ -w file ] [ -W filecount ] [ -y datalinktype ]
             [ -z postrotate-command ] [ -Z user ]
             [ --time-stamp-precision=tstamp_precision ]
             [ --micro ] [ --nano ]
             [ expression ]

## DESCRIPTION

`Tcpdump` prints out a description of the contents of packets on a network interface that match the Boolean expression; the description is preceded by a time stamp, printed, by default, as hours, minutes, seconds, and fractions of a second since midnight. It can also be run with the `-w` flag, which causes it to save the packet data to a file for later analysis, and/or with the `-r` flag, which causes it to read from a saved packet file rather than to read packets from a network interface. It can also be run with the `-V` flag, which causes it to read a list of saved packet files. In all cases, only packets that match expression will be processed by `tcpdump`.

`Tcpdump` will, if not run with the `-c` flag, continue capturing packets until it is interrupted by a `SIGINT` signal (generated, for example, by typing your interrupt character, typically `control-C`) or a `SIGTERM` signal (typically generated with the `kill`(1) command); if run with the `-c` flag, it will capture packets until it is interrupted by a `SIGINT` or `SIGTERM` signal or the specified number of packets have been processed.

When `tcpdump` finishes capturing packets, it will report counts of:

packets "captured" (this is the number of packets that `tcpdump` has received and processed);

packets "received by filter" (the meaning of this depends on the OS on which you're running `tcpdump`, and possibly on the way the OS was configured - if a filter was specified on the command line, on some OSes it counts packets regardless of whether they were matched by the filter expression and, even if they were matched by the filter expression, regardless of whether `tcpdump` has read and processed them yet, on other OSes it counts only packets that were matched by the filter expression regardless of whether `tcpdump` has read and processed them yet, and on other OSes it counts only packets that were matched by the filter expression and were processed by `tcpdump`);

packets "dropped by kernel" (this is the number of packets that were dropped, due to a lack of buffer space, by the packet capture mechanism in the OS on which `tcpdump` is running, if the OS reports that information to applications; if not, it will be reported as 0).

On platforms that support the `SIGINFO` signal, such as most BSDs (including macOS) and Digital/Tru64 UNIX, it will report those counts when it receives a `SIGINFO` signal (generated, for example, by typing your "status" character, typically control-T, although on some platforms, such as macOS, the "status" character is not set by default, so you must set it with `stty`(1) in order to use it) and will continue capturing packets. On platforms that do not support the `SIGINFO` signal, the same can be achieved by using the `SIGUSR1` signal.

Using the `SIGUSR2` signal along with the `-w` flag will forcibly flush the packet buffer into the output file.

Reading packets from a network interface may require that you have special privileges; see the [pcap](https://www.tcpdump.org/manpages/pcap.3pcap.html)(3PCAP) man page for details. Reading a saved packet file doesn't require special privileges.  

## OPTIONS

### `-A`

Print each packet (minus its link level header) in ASCII. Handy for capturing web pages.

### `-b`

Print the AS number in BGP packets in ASDOT notation rather than ASPLAIN notation.

### `-B buffer_size` | `--buffer-size=buffer_size`

Set the operating system capture buffer size to `buffer_size`, in units of KiB (1024 bytes).

### `-c count`

Exit after receiving count packets.

### `--count`

Print only on stderr the packet count when reading capture file(s) instead of parsing/printing the packets. If a filter is specified on the command line, `tcpdump` counts only packets that were matched by the filter expression.

### `-C file_size`

Before writing a raw packet to a savefile, check whether the file is currently larger than `file_size` and, if so, close the current savefile and open a new one. Savefiles after the first savefile will have the name specified with the `-w` flag, with a number after it, starting at `1` and continuing upward. The units of `file_size` are millions of bytes (1,000,000 bytes, not 1,048,576 bytes).

### `-d`

Dump the compiled packet-matching code in a human readable form to standard output and stop.

Please mind that although code compilation is always DLT-specific, typically it is impossible (and unnecessary) to specify which DLT to use for the dump because `tcpdump` uses either the DLT of the input pcap file specified with `-r`, or the default DLT of the network interface specified with `-i`, or the particular DLT of the network interface specified with `-y` and `-i` respectively. In these cases the dump shows the same exact code that would filter the input file or the network interface without `-d`.

However, when neither `-r` nor `-i` is specified, specifying `-d` prevents `tcpdump` from guessing a suitable network interface (see `-i`). In this case the DLT defaults to EN10MB and can be set to another valid value manually with `-y`.

### `-dd`

Dump packet-matching code as a C program fragment.

### `-ddd`

Dump packet-matching code as decimal numbers (preceded with a count).

### `-D` | `--list-interfaces`

Print the list of the network interfaces available on the system and on which `tcpdump` can capture packets. For each network interface, a number and an interface name, possibly followed by a text description of the interface, are printed. The interface name or the number can be supplied to the `-i` flag to specify an interface on which to capture.

This can be useful on systems that don't have a command to list them (e.g., Windows systems, or UNIX systems lacking `ifconfig -a`); the number can be useful on Windows 2000 and later systems, where the interface name is a somewhat complex string.

The -D flag will not be supported if tcpdump was built with an older version of libpcap that lacks the pcap_findalldevs(3PCAP) function.

### `-e`

Print the link-level header on each dump line. This can be used, for example, to print MAC layer addresses for protocols such as Ethernet and IEEE 802.11.

### `-E`

Use spi@ipaddr algo:secret for decrypting IPsec ESP packets that are addressed to addr and contain Security Parameter Index value spi. This combination may be repeated with comma or newline separation.

Note that setting the secret for IPv4 ESP packets is supported at this time.

Algorithms may be des-cbc, 3des-cbc, blowfish-cbc, rc3-cbc, cast128-cbc, or none. The default is des-cbc. The ability to decrypt packets is only present if tcpdump was compiled with cryptography enabled.

secret is the ASCII text for ESP secret key. If preceded by 0x, then a hex value will be read.

The option assumes RFC2406 ESP, not RFC1827 ESP. The option is only for debugging purposes, and the use of this option with a true `secret' key is discouraged. By presenting IPsec secret key onto command line you make it visible to others, via ps(1) and other occasions.

In addition to the above syntax, the syntax file name may be used to have tcpdump read the provided file in. The file is opened upon receiving the first ESP packet, so any special permissions that tcpdump may have been given should already have been given up.

### `-f`

Print `foreign' IPv4 addresses numerically rather than symbolically (this option is intended to get around serious brain damage in Sun's NIS server --- usually it hangs forever translating non-local internet numbers).

The test for `foreign' IPv4 addresses is done using the IPv4 address and netmask of the interface on which capture is being done. If that address or netmask are not available, available, either because the interface on which capture is being done has no address or netmask or because the capture is being done on the Linux "any" interface, which can capture on more than one interface, this option will not work correctly.

### `-F file`

Use file as input for the filter expression. An additional expression given on the command line is ignored.

### `-G rotate_seconds`

If specified, rotates the dump file specified with the `-w` option every `rotate_seconds` seconds. Savefiles will have the name specified by `-w` which should include a time format as defined by `strftime(3)`. If no time format is specified, each new file will overwrite the previous. Whenever a generated filename is not unique, `tcpdump` will overwrite the pre-existing data; providing a time specification that is coarser than the capture period is therefore not advised.

If used in conjunction with the `-C` option, filenames will take the form of `file<count>`.

### `-h` | `--help`

Print the `tcpdump` and `libpcap` version strings, print a usage message, and exit.

### `--version`

Print the `tcpdump` and libpcap version strings and exit.

### `-H`

Attempt to detect 802.11s draft mesh headers.

### `-i interface` | `--interface=interface`

Listen, report the list of link-layer types, report the list of time stamp types, or report the results of compiling a filter expression on interface. If unspecified and if the `-d` flag is not given, `tcpdump` searches the system interface list for the lowest numbered, configured up interface (excluding loopback), which may turn out to be, for example, "eth0".

On Linux systems with 2.2 or later kernels, an interface argument of "any" can be used to capture packets from all interfaces. Note that captures on the "any" device will not be done in promiscuous mode.

If the `-D` flag is supported, an interface number as printed by that flag can be used as the interface argument, if no interface on the system has that number as a name.

### `-I` | `--monitor-mode`

Put the interface in "monitor mode"; this is supported only on IEEE 802.11 Wi-Fi interfaces, and supported only on some operating systems.

Note that in monitor mode the adapter might disassociate from the network with which it's associated, so that you will not be able to use any wireless networks with that adapter. This could prevent accessing files on a network server, or resolving host names or network addresses, if you are capturing in monitor mode and are not connected to another network with another adapter.

This flag will affect the output of the `-L` flag. If `-I` isn't specified, only those link-layer types available when not in monitor mode will be shown; if `-I` is specified, only those link-layer types available when in monitor mode will be shown.

### `--immediate-mode`

Capture in "immediate mode". In this mode, packets are delivered to tcpdump as soon as they arrive, rather than being buffered for efficiency. This is the default when printing packets rather than saving packets to a ``savefile'' if the packets are being printed to a terminal rather than to a file or pipe.

### `-j tstamp_type` | `--time-stamp-type=tstamp_type`

Set the time stamp type for the capture to `tstamp_type`. The names to use for the time stamp types are given in `pcap-tstamp`(7); not all the types listed there will necessarily be valid for any given interface.

### `-J` | `--list-time-stamp-types`

List the supported time stamp types for the interface and exit. If the time stamp type cannot be set for the interface, no time stamp types are listed.

### `--time-stamp-precision=tstamp_precision`

When capturing, set the time stamp precision for the capture to `tstamp_precision`. Note that availability of high precision time stamps (nanoseconds) and their actual accuracy is platform and hardware dependent. Also note that when writing captures made with nanosecond accuracy to a savefile, the time stamps are written with nanosecond resolution, and the file is written with a different magic number, to indicate that the time stamps are in seconds and nanoseconds; not all programs that read pcap savefiles will be able to read those captures.

When reading a savefile, convert time stamps to the precision specified by timestamp_precision, and display them with that resolution. If the precision specified is less than the precision of time stamps in the file, the conversion will lose precision.

The supported values for timestamp_precision are micro for microsecond resolution and nano for nanosecond resolution. The default is microsecond resolution.

### `--micro` | `--nano`

Shorthands for `--time-stamp-precision=micro` or `--time-stamp-precision=nano`, adjusting the time stamp precision accordingly. When reading packets from a savefile, using `--micro` truncates time stamps if the savefile was created with nanosecond precision. In contrast, a savefile created with microsecond precision will have trailing zeroes added to the time stamp when `--nano` is used.

### `-K` | `--dont-verify-checksums`

Don't attempt to verify IP, TCP, or UDP checksums. This is useful for interfaces that perform some or all of those checksum calculation in hardware; otherwise, all outgoing TCP checksums will be flagged as bad.

### `-l`

Make stdout line buffered. Useful if you want to see the data while capturing it. E.g.,

    tcpdump -l | tee dat

or

    tcpdump -l > dat & tail -f dat

Note that on Windows, "line buffered" means "unbuffered", so that `WinDump` will write each character individually if `-l` is specified.

`-U` is similar to `-l` in its behavior, but it will cause output to be "packet-buffered", so that the output is written to stdout at the end of each packet rather than at the end of each line; this is buffered on all platforms, including Windows.

### `-L` | `--list-data-link-types`

List the known data link types for the interface, in the specified mode, and exit. The list of known data link types may be dependent on the specified mode; for example, on some platforms, a Wi-Fi interface might support one set of data link types when not in monitor mode (for example, it might support only fake Ethernet headers, or might support 802.11 headers but not support 802.11 headers with radio information) and another set of data link types when in monitor mode (for example, it might support 802.11 headers, or 802.11 headers with radio information, only in monitor mode).

### `-m module`

Load SMI MIB module definitions from file module. This option can be used several times to load several MIB modules into `tcpdump`.

### `-M secret`

Use secret as a shared secret for validating the digests found in TCP segments with the TCP-MD5 option (RFC 2385), if present.

### `-n`

Don't convert addresses (i.e., host addresses, port numbers, etc.) to names.

### `-N`

Don't print domain name qualification of host names. E.g., if you give this flag then `tcpdump` will print "nic" instead of "nic.ddn.mil".

### `-#` | `--number`

Print an optional packet number at the beginning of the line.

### `-O` | `--no-optimize`

Do not run the packet-matching code optimizer. This is useful only if you suspect a bug in the optimizer.

### `-p` | `--no-promiscuous-mode`

Don't put the interface into promiscuous mode. Note that the interface might be in promiscuous mode for some other reason; hence, `-p` cannot be used as an abbreviation for 'ether host {local-hw-addr} or ether broadcast'.

### `--print`

Print parsed packet output, even if the raw packets are being saved to a file with the `-w` flag.

### `-Q direction` | `--direction=direction`

Choose send/receive direction direction for which packets should be captured. Possible values are `in`, `out` and `inout`. Not available on all platforms.

### `-q`

Quick (quiet?) output. Print less protocol information so output lines are shorter.

### `-r file`

Read packets from file (which was created with the `-w` option or by other tools that write pcap or pcapng files). Standard input is used if file is "-".

### `-S` | `--absolute-tcp-sequence-numbers`

Print absolute, rather than relative, TCP sequence numbers.

### `-s snaplen` | `--snapshot-length=snaplen`

Snarf snaplen bytes of data from each packet rather than the default of 262144 bytes. Packets truncated because of a limited snapshot are indicated in the output with "[|proto]", where proto is the name of the protocol level at which the truncation has occurred.

Note that taking larger snapshots both increases the amount of time it takes to process packets and, effectively, decreases the amount of packet buffering. This may cause packets to be lost. Note also that taking smaller snapshots will discard data from protocols above the transport layer, which loses information that may be important. NFS and AFS requests and replies, for example, are very large, and much of the detail won't be available if a too-short snapshot length is selected.

If you need to reduce the snapshot size below the default, you should limit snaplen to the smallest number that will capture the protocol information you're interested in. Setting snaplen to 0 sets it to the default of 262144, for backwards compatibility with recent older versions of tcpdump.

### `-T type`

Force packets selected by "expression" to be interpreted the specified type. Currently known types are aodv (Ad-hoc On-demand Distance Vector protocol), carp (Common Address Redundancy Protocol), cnfp (Cisco NetFlow protocol), domain (Domain Name System), lmp (Link Management Protocol), pgm (Pragmatic General Multicast), pgm_zmtp1 (ZMTP/1.0 inside PGM/EPGM), ptp (Precision Time Protocol), radius (RADIUS), resp (REdis Serialization Protocol), rpc (Remote Procedure Call), rtcp (Real-Time Applications control protocol), rtp (Real-Time Applications protocol), snmp (Simple Network Management Protocol), someip (SOME/IP), tftp (Trivial File Transfer Protocol), vat (Visual Audio Tool), vxlan (Virtual eXtensible Local Area Network), wb (distributed White Board) and zmtp1 (ZeroMQ Message Transport Protocol 1.0).

Note that the pgm type above affects UDP interpretation only, the native PGM is always recognised as IP protocol 113 regardless. UDP-encapsulated PGM is often called "EPGM" or "PGM/UDP".

Note that the pgm_zmtp1 type above affects interpretation of both native PGM and UDP at once. During the native PGM decoding the application data of an ODATA/RDATA packet would be decoded as a ZeroMQ datagram with ZMTP/1.0 frames. During the UDP decoding in addition to that any UDP packet would be treated as an encapsulated PGM packet.

### `-t`

Don't print a timestamp on each dump line.

### `-tt`

Print the timestamp, as seconds since January 1, 1970, 00:00:00, UTC, and fractions of a second since that time, on each dump line.

### `-ttt`

Print a delta (microsecond or nanosecond resolution depending on the `--time-stamp-precision` option) between current and previous line on each dump line. The default is microsecond resolution.

### `-tttt`

Print a timestamp, as hours, minutes, seconds, and fractions of a second since midnight, preceded by the date, on each dump line.

### `-ttttt`

Print a delta (microsecond or nanosecond resolution depending on the `--time-stamp-precision` option) between current and first line on each dump line. The default is microsecond resolution.

### `-u`

Print undecoded NFS handles.

### `-U` | `--packet-buffered`

If the `-w` option is not specified, or if it is specified but the `--print` flag is also specified, make the printed packet output "packet-buffered"; i.e., as the description of the contents of each packet is printed, it will be written to the standard output, rather than, when not writing to a terminal, being written only when the output buffer fills.

If the `-w` option is specified, make the saved raw packet output "packet-buffered"; i.e., as each packet is saved, it will be written to the output file, rather than being written only when the output buffer fills.

The `-U` flag will not be supported if `tcpdump` was built with an older version of `libpcap` that lacks the [pcap_dump_flush](https://www.tcpdump.org/manpages/pcap_dump_flush.3pcap.html)(3PCAP) function.

### `-v`

When parsing and printing, produce (slightly more) verbose output. For example, the time to live, identification, total length and options in an IP packet are printed. Also enables additional packet integrity checks such as verifying the IP and ICMP header checksum.

When writing to a file with the `-w` option and at the same time not reading from a file with the `-r` option, report to stderr, once per second, the number of packets captured. In Solaris, FreeBSD and possibly other operating systems this periodic update currently can cause loss of captured packets on their way from the kernel to `tcpdump`.

### `-vv`

Even more verbose output. For example, additional fields are printed from NFS reply packets, and SMB packets are fully decoded.

### `-vvv`

Even more verbose output. For example, telnet SB ... SE options are printed in full. With `-X` Telnet options are printed in hex as well.

### `-V file`

Read a list of filenames from file. Standard input is used if file is "-".

### `-w file`

Write the raw packets to file rather than parsing and printing them out. They can later be printed with the `-r` option. Standard output is used if file is "-".

This output will be buffered if written to a file or pipe, so a program reading from the file or pipe may not see packets for an arbitrary amount of time after they are received. Use the `-U` flag to cause packets to be written as soon as they are received.

The MIME type application/vnd.tcpdump.pcap has been registered with IANA for pcap files. The filename extension .pcap appears to be the most commonly used along with .cap and .dmp. Tcpdump itself doesn't check the extension when reading capture files and doesn't add an extension when writing them (it uses magic numbers in the file header instead). However, many operating systems and applications will use the extension if it is present and adding one (e.g. .pcap) is recommended.

See [pcap-savefile](https://www.tcpdump.org/manpages/pcap-savefile.5.html)(5) for a description of the file format.

### `-W filecount`

Used in conjunction with the `-C` option, this will limit the number of files created to the specified number, and begin overwriting files from the beginning, thus creating a 'rotating' buffer. In addition, it will name the files with enough leading 0s to support the maximum number of files, allowing them to sort correctly.

Used in conjunction with the `-G` option, this will limit the number of rotated dump files that get created, exiting with status 0 when reaching the limit.

If used in conjunction with both `-C` and `-G`, the `-W` option will currently be ignored, and will only affect the file name.

### `-x`

When parsing and printing, in addition to printing the headers of each packet, print the data of each packet (minus its link level header) in hex. The smaller of the entire packet or snaplen bytes will be printed. Note that this is the entire link-layer packet, so for link layers that pad (e.g. Ethernet), the padding bytes will also be printed when the higher layer packet is shorter than the required padding. In the current implementation this flag may have the same effect as `-xx` if the packet is truncated.

### `-xx`

When parsing and printing, in addition to printing the headers of each packet, print the data of each packet, including its link level header, in hex.

### `-X`

When parsing and printing, in addition to printing the headers of each packet, print the data of each packet (minus its link level header) in hex and ASCII. This is very handy for analysing new protocols. In the current implementation this flag may have the same effect as `-XX` if the packet is truncated.

### `-XX`

When parsing and printing, in addition to printing the headers of each packet, print the data of each packet, including its link level header, in hex and ASCII.

### `-y datalinktype` | `--linktype=datalinktype`

Set the data link type to use while capturing packets (see `-L`) or just compiling and dumping packet-matching code (see `-d`) to datalinktype.

### `-z postrotate-command`

Used in conjunction with the `-C` or `-G` options, this will make `tcpdump` run " postrotate-command file " where file is the savefile being closed after each rotation. For example, specifying -z gzip or -z bzip2 will compress each savefile using gzip or bzip2.

Note that tcpdump will run the command in parallel to the capture, using the lowest priority so that this doesn't disturb the capture process.

And in case you would like to use a command that itself takes flags or different arguments, you can always write a shell script that will take the savefile name as the only argument, make the flags & arguments arrangements and execute the command that you want.

### `-Z user` | `--relinquish-privileges=user`

If `tcpdump` is running as root, after opening the capture device or input savefile, but before opening any savefiles for output, change the user ID to user and the group ID to the primary group of user.

This behavior can also be enabled by default at compile time.

### `expression`

selects which packets will be dumped. If no expression is given, all packets on the net will be dumped. Otherwise, only packets for which expression is `true` will be dumped.

For the expression syntax, see [pcap-filter](https://www.tcpdump.org/manpages/pcap-filter.7.html)(7).

The expression argument can be passed to `tcpdump` as either a single Shell argument, or as multiple Shell arguments, whichever is more convenient. Generally, if the expression contains **Shell metacharacters**, such as backslashes used to escape protocol names, it is easier to pass it as a single, quoted argument rather than to escape the Shell metacharacters. Multiple arguments are concatenated with spaces before being parsed.

## EXAMPLES

### To print all packets arriving at or departing from sundown

    tcpdump host sundown

### To print traffic between helios and either hot or ace

    tcpdump host helios and \( hot or ace \)

### To print all IP packets between ace and any host except helios

    tcpdump ip host ace and not helios

### To print all traffic between local hosts and hosts at Berkeley

    tcpdump net ucb-ether

### To print all ftp traffic through internet gateway snup

(note that the expression is quoted to prevent the shell from (mis-)interpreting the parentheses):

    tcpdump 'gateway snup and (port ftp or ftp-data)'

### To print traffic neither sourced from nor destined for local hosts 

(if you gateway to one other net, this stuff should never make it onto your local net).

    tcpdump ip and not net localnet

### To print the start and end packets (the SYN and FIN packets) of each TCP conversation that involves a non-local host

    tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'

### To print the TCP packets with flags RST and ACK both set.

(i.e. select only the RST and ACK flags in the flags field, and if the result is "RST and ACK both set", match)

tcpdump 'tcp[tcpflags] & (tcp-rst|tcp-ack) == (tcp-rst|tcp-ack)'

### To print all IPv4 HTTP packets to and from port 80, i.e. print only packets that contain data, not, for example, SYN and FIN packets and ACK-only packets

(IPv6 is left as an exercise for the reader.)

tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

### To print IP packets longer than 576 bytes sent through gateway snup

    tcpdump 'gateway snup and ip[2:2] > 576'

### To print IP broadcast or multicast packets that were not sent via Ethernet broadcast or multicast

    tcpdump 'ether[0] & 1 = 0 and ip[16] >= 224'

### To print all ICMP packets that are not echo requests/replies (i.e., not ping packets)

    tcpdump 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'

## OUTPUT FORMAT

The output of tcpdump is protocol dependent. The following gives a brief description and examples of most of the formats.

### Timestamps

By default, all output lines are preceded by a timestamp. The timestamp is the current clock time in the form

    hh:mm:ss.frac

and is as accurate as the kernel's clock. The timestamp reflects the time the kernel applied a time stamp to the packet. No attempt is made to account for the time lag between when the network interface finished receiving the packet from the network and when the kernel applied a time stamp to the packet; that time lag could include a delay between the time when the network interface finished receiving a packet from the network and the time when an interrupt was delivered to the kernel to get it to read the packet and a delay between the time when the kernel serviced the `new packet' interrupt and the time when it applied a time stamp to the packet.

### Link Level Headers

If the `-e` option is given, the link level header is printed out. On Ethernets, the source and destination addresses, protocol, and packet length are printed.

On FDDI `networks`, the `-e` option causes `tcpdump` to print the 'frame control' field, the source and destination addresses, and the packet length. (The 'frame control' field governs the interpretation of the rest of the packet. Normal packets (such as those containing IP datagrams) are 'async' packets, with a priority value between 0 and 7; for example, 'async4'. Such packets are assumed to contain an 802.2 Logical Link Control (LLC) packet; the LLC header is printed if it is not an ISO datagram or a so-called SNAP packet.

On Token Ring networks, the `-e` option causes tcpdump to print the 'access control' and 'frame control' fields, the source and destination addresses, and the packet length. As on FDDI networks, packets are assumed to contain an LLC packet. Regardless of whether the `-e` option is specified or not, the source routing information is printed for source-routed packets.

On 802.11 networks, the `-e` option causes tcpdump to print the 'frame control' fields, all of the addresses in the 802.11 header, and the packet length. As on FDDI networks, packets are assumed to contain an LLC packet.

(N.B.: The following description assumes familiarity with the SLIP compression algorithm described in RFC-1144.)

On SLIP links, a direction indicator ("I" for inbound, "O" for outbound), packet type, and compression information are printed out. The packet type is printed first. The three types are ip, utcp, and ctcp. No further link information is printed for ip packets. For TCP packets, the connection identifier is printed following the type. If the packet is compressed, its encoded header is printed out. The special cases are printed out as `*S+n` and `*SA+n`, where n is the amount by which the sequence number (or sequence number and ack) has changed. If it is not a special case, zero or more changes are printed. A change is indicated by U (urgent pointer), W (window), A (ack), S (sequence number), and I (packet ID), followed by a delta (+n or -n), or a new value (=n). Finally, the amount of data in the packet and compressed header length are printed.

For example, the following line shows an outbound compressed TCP packet, with an implicit connection identifier; the ack has changed by 6, the sequence number by 49, and the packet ID by 6; there are 3 bytes of data and 6 bytes of compressed header:

    O ctcp * A+6 S+49 I+6 3 (6)

### ARP/RARP Packets

ARP/RARP output shows the type of request and its arguments. The format is intended to be self explanatory. Here is a short sample taken from the start of an `rlogin` from host rtsg to host csam:

    arp who-has csam tell rtsg
    arp reply csam is-at CSAM

The first line says that rtsg sent an ARP packet asking for the Ethernet address of internet host csam. Csam replies with its Ethernet address (in this example, Ethernet addresses are in caps and internet addresses in lower case).

This would look less redundant if we had done `tcpdump -n`:

    arp who-has 128.3.254.6 tell 128.3.254.68
    arp reply 128.3.254.6 is-at 02:07:01:00:01:c4

If we had done `tcpdump -e`, the fact that the first packet is broadcast and the second is point-to-point would be visible:

    RTSG Broadcast 0806  64: arp who-has csam tell rtsg
    CSAM RTSG 0806  64: arp reply csam is-at CSAM

For the first packet this says the Ethernet source address is RTSG, the destination is the Ethernet broadcast address, the type field contained hex 0806 (type ETHER_ARP) and the total length was 64 bytes.

### IPv4 Packets

If the link-layer header is not being printed, for IPv4 packets, IP is printed after the time stamp.

If the `-v` flag is specified, information from the IPv4 header is shown in parentheses after the IP or the link-layer header. The general format of this information is:

    tos tos, ttl ttl, id id, offset offset, flags [flags], proto proto, length length, options (options)

tos is the type of service field; if the ECN bits are non-zero, those are reported as ECT(1), ECT(0), or CE. ttl is the time-to-live; it is not reported if it is zero. id is the IP identification field. offset is the fragment offset field; it is printed whether this is part of a fragmented datagram or not. flags are the MF and DF flags; + is reported if MF is set, and DF is reported if F is set. If neither are set, . is reported. proto is the protocol ID field. length is the total length field. options are the IP options, if any.

Next, for TCP and UDP packets, the source and destination IP addresses and TCP or UDP ports, with a dot between each IP address and its corresponding port, will be printed, with a > separating the source and destination. For other protocols, the addresses will be printed, with a > separating the source and destination. Higher level protocol information, if any, will be printed after that.

For fragmented IP datagrams, the first fragment contains the higher level protocol header; fragments after the first contain no higher level protocol header. Fragmentation information will be printed only with the -v flag, in the IP header information, as described above.

### TCP Packets

(N.B.:The following description assumes familiarity with the TCP protocol described in RFC-793. If you are not familiar with the protocol, this description will not be of much use to you.)

The general format of a TCP protocol line is:

    src > dst: Flags [tcpflags], seq data-seqno, ack ackno, win window, urg urgent, options [opts], length len
















TODO tcpdump tttttttttttttttttttttttttttttttttt