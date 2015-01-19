Consul Playground
=================

An attempt at playing with [Consul](http://consul.io) and NAT-T
Hosts `c1` and `c2` are in the same segment, host `c3` is in another.
`c1` has a floating IP of 4.4.4.4
`c2` has a floating IP of 5.5.5.5

# Testing Consul is Peered

As a rudimentary test, I'm writing a value on `c2` and checking it can be read on `c1`

	# Write
	curl -X PUT -d 'test' http://localhost:8500/v1/kv/web/key1
	# Read
	curl -X GET http://localhost:8500/v1/kv/web/key1

# Cleaning Up

	killall consul
	rm -r foo

# Scenarios

## Join LAN Cluster

	# On C1
	consul agent -data-dir=foo -bind 172.20.1.2 -server -bootstrap -node c1 &

	# On C2
	consul agent -data-dir=foo -bind 172.22.1.2 -server -node c2 &
	consul join 4.4.4.4

OMGWTF. We join `4.4.4.4` but we peer with `172.20.1.2`!

## Join WAN Cluster

	# On C1
	consul agent -data-dir=foo -bind 172.20.1.2 -server -bootstrap -node c1 &

	# On C2
	consul agent -data-dir=foo -bind 172.22.1.2 -server -node c2 &
	consul join -wan 4.4.4.4

OMGWTF! Claims to join the cluster and then promptly leaves claiming no leader.
This `wan` thing needs some further investigation

## Join LAN Cluster, Advertise Public IP

	# On C1
	consul agent -data-dir=foo -bind 172.20.1.2 -server -bootstrap -node c1 -advertise 4.4.4.4&
	
	# On C2
	consul agent -data-dir=foo -bind 172.20.1.2 -server -node c2 -advertise 5.5.5.5&
        consul join 4.4.4.4

SUCCESS! Now lets try `c3` on the LAN

Epic FacePalm - 4.4.4.4 and 5.5.5.5 aren't reachable :(
We should probably use a different advertise address for WAN and LAN gossip
