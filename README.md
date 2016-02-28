# Splunk TA for Netflow, using Logstash as netflow collector

This CIM compliant Splunk TA can be used with Splunk Enterprise Security and
provides field extractions, aliases, and tags for Netflow v5, v9 and IPFIX data
that has been collected by Logstash.

## Installation

1. Install this Splunk TA on your Splunk (Enterprise Security) search head. Make sure to rename it Splunk_TA_netflow_logstash or TA-netflow-logstash otherwise ES won't eat it.
2. Install Logstash on an instance that will receive netflow data from the various netflow probes
3. Install a Splunk Universal Forwarder on the Logstash instance

## Configuration 

1. Configure Logstash to output a .json file for the received netflow data, for example with the following config file:

    ```
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
    ```

2. Have the Splunk Universal Forwarder index the `netflow.json` file, for example with the following Splunk inputs.conf:

    ```
[monitor:///var/log/netflow/netflow.json]
disabled = false
sourcetype = netflow_raw
index=netflow_raw
    ```

3. Create 2 Splunk indexes:

    * `netflow_raw`
       
       This index will temporarily hold the Netflow data, so you can keep it small at e.g. 1GB.

    * `netflow`

       This index will hold the flow objects built from the netflow_raw index, so size it for proper retention. This index will be filled through a scheduled Splunk search that runs every minute.


## Netflow probes

Logstash has been tested with the following netflow probes:

| netflow probe | v5 | v9 | v10 / IPFIX | output fields
|---------------|----|----|------|----
| fprobe        | y  |  N | N    | Only netflow v5 so no ipv6
| softflowd     | y  |  y | y    | 
| nprobe        | y  |  y | y    |
| ipt_NETFLOW   | y  |  y | y    |

## CIM 

The TA provides fields compatible with the Splunk Common Information Model (CIM):

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

### Netflow fields

The original fields are also still available through the `netflow.` prefix, followed by their Netflow field names:

#### nprobe (v10 output)

* version
* sourceIPv4Address
* destinationIPv4Address
* ipNextHopIPv4Address
* sourceTransportPort
* destinationTransportPort
* tcpControlBits
* ingressInterface
* egressInterface
* packetDeltaCount
* octetDeltaCount
* flowStartMilliseconds
* flowEndMilliseconds
* protocolIdentifier
* ipClassOfService
* flowEndReason
* tcpOptions

#### ipt_NETFLOW (v10 output)

* version
* sourceIPv4Address
* destinationIPv4Address
* ipNextHopIPv4Address
* sourceTransportPort
* destinationTransportPort
* tcpControlBits
* ingressInterface
* egressInterface
* packetDeltaCount
* octetDeltaCount
* flowStartMilliseconds
* flowEndMilliseconds
* protocolIdentifier
* ipClassOfService
* flowEndReason

#### nprobe (v9 output)

* version
* flow_seq_num
* flowset_id
* in_bytes
* in_pkts
* protocol
* src_tos
* tcp_flags
* l4_src_port
* ipv4_src_addr
* src_mask
* input_snmp
* l4_dst_port
* ipv4_dst_addr
* dst_mask
* output_snmp
* ipv4_next_hop
* src_as
* dst_as
* last_switched
* first_switched

#### softflowd (v9 output)

* version
* flow_seq_num
* flowset_id
* ipv4_src_addr | ipv6_dst_addr
* 83.149.69.228
* ipv4_dst_addr | ipv6_dst_addr
* last_switched
* first_switched
* in_bytes
* in_pkts
* input_snmp
* output_snmp
* l4_src_port
* l4_dst_port
* protocol
* tcp_flags
* ip_protocol_version

#### fprobe (v5 output)

TODO

## Contribute

Any feedback in the form of patches, feature requests, bug reports or just an email is most welcome.
If you want to provide patches, please do so through a Pull Request.
If you have any bug reports or feature requests, please submit them to the issue tracker.

