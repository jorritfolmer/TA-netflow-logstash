# Splunk App for Netflow

This Splunk App provides search knowledge for Netflow v9 and IPFIX (v10) streams that are captured
by Logstash acting as a netflow collector.
It is a work in progress, so please mind the gap. Currently, Logstash only supports v10 flows after applying the ipfix branch from the [Matt Dainty's fork for of logstash-codec-netflow](https://github.com/bodgit/logstash-codec-netflow/tree/ipfix)

It also offers advanced flow analytics for finding cyclical patterns in the netflow data, using the [R for Splunk app](https://github.com/rfsp/r)

## Installation

0. Logstash Netflow collector:
    * Install Logstash 
    * Put this config in /etc/logstash/conf.d/netflow.conf:

        ````
        input {
          udp {
            port => 2055
            codec => netflow {
              versions => [5,9,10]
            }
          }
        }

        output {
          file {
            codec => "json"
            path => "/var/log/netflow/netflow.json"
          }
        }
        ````

    * Install the ipfix branch from @bodgit:

        ````
        cd /opt/logstash/vendor/bundle/jruby/1.9/gems/
        rm -fr logstash-codec-netflow-1.0.0
        git clone https://github.com/bodgit/logstash-codec-netflow.git
        mv logstash-codec-netflow logstash-codec-netflow-1.0.0
        cd logstash-codec-netflow-1.0.0
        git checkout ipfix
        ```` 

    * Install splunk-forwarder on this machine

1. Splunk indexer:
    * Install this Splunk TA on your indexer(s), or manually create index(es) called "netflow"
2. Splunk search head:
    * Install this Splunk TA on your search head to get the main netflow dashboard
    * Install the R for Splunk app from @rfsp through the Splunk app store, or `git clone https://github.com/rfsp/r.git`
3. Splunk deployment server:
    * Install this Splunk TA on your deployment server, or manually create an app with the appropriate inputs.conf

        ````
        cd $SPLUNK_HOME/etc/deployment-apps`
        git clone https://github.com/jorritfolmer/splunk_ta_netflow_logstash.git
        ````

    * Enable the inputs on the deployment server by setting `disabled = 0` in `$SPLUNK_HOME/etc/deployment-apps/splunk_ta_netflow_logstash/default/inputs.conf`
    * Create a serverclass (e.g. `netflow`) on the deployment-server
    * Assign this app to the serverclass
    * Assign clients the serverclass

4. Deploy the app from the Splunk deployment server to the netflow collector
    * `/opt/splunk/bin/splunk reload deploy-server`

## Configuration

None necessary, but if you care about your Splunk license usage, it may be a good idea to switch Logstash from JSON to CSV output. However, this means modifying both Logstash and Splunk configuration, and losing the advantage of the flexible (but expensive) JSON output to the efficient but fixed CSV output.

CSV efficiency: (kb/flows) 95136802/820751 = 116 bytes / flow in Splunk
JSON efficiency: (kb/flows) 246288299/493548 = 499 bytes / flow in Splunk

## Netflow probes

This app has been tested with the following netflow probes:

| netflow probe | v5 | v9 | v10 / IPFIX | output fields
|---------------|----|----|------|----
| fprobe        | y  |  N | N    | Only netflow v5 so no ipv6
| softflowd     | y  |  y | N    | 
| nprobe        | y  |  y | y    |
| ipt_NETFLOW   | y  |  y | y    |

Every probe has its own timeout settings. For example nprobe 120 sec, ipt_NETFLOW 1800 sec, softflowd 3600 sec.
So in Splunk we should sum over the Flow Keys to get the "real" flow duration/bytes_in/...

## Splunk CIM compliance

The default output from the above Netflow probes is standardised into the following Splunk CIM fields:

* duration
* src_ip
* dest_ip
* src_port
* dest_port
* bytes_in
* pkts_in
* transport
* protocol_version
* icmp_type


This Splunk App is compatible with Splunk Enterprise 6.2+.

### Netflow fields

The original fields are also still available through the `netflow.` prefix, followed by their Netflow field names:

#### nprobe (v10 output)
version
sourceIPv4Address
destinationIPv4Address
ipNextHopIPv4Address
sourceTransportPort
destinationTransportPort
tcpControlBits
ingressInterface
egressInterface
packetDeltaCount
octetDeltaCount
flowStartMilliseconds
flowEndMilliseconds
protocolIdentifier
ipClassOfService
flowEndReason
tcpOptions

#### ipt_NETFLOW (v10 output)

version
sourceIPv4Address
destinationIPv4Address
ipNextHopIPv4Address
sourceTransportPort
destinationTransportPort
tcpControlBits
ingressInterface
egressInterface
packetDeltaCount
octetDeltaCount
flowStartMilliseconds
flowEndMilliseconds
protocolIdentifier
ipClassOfService
flowEndReason

#### nprobe (v9 output)

version
flow_seq_num
flowset_id
in_bytes
in_pkts
protocol
src_tos
tcp_flags
l4_src_port
ipv4_src_addr
src_mask
input_snmp
l4_dst_port
ipv4_dst_addr
dst_mask
output_snmp
ipv4_next_hop
src_as
dst_as
last_switched
first_switched

#### softflowd (v9 output)

version
flow_seq_num
flowset_id
ipv4_src_addr | ipv6_dst_addr
83.149.69.228
ipv4_dst_addr | ipv6_dst_addr
last_switched
first_switched
in_bytes
in_pkts
input_snmp
output_snmp
l4_src_port
l4_dst_port
protocol
tcp_flags
ip_protocol_version

#### fprobe (v5 output)

TODO

## Contribute

Any feedback in the form of patches, feature requests, bug reports or just an email is most welcome.
If you want to provide patches, please do so through a Pull Request.
If you have any bug reports or feature requests, please submit them to the issue tracker.
