<!SLIDE small>
# Task C: PacketIn Dumper ######################################################

## Handling Packet-In Messages


<!SLIDE small>
## Exercise: Displaying Packet-In Message Dump #################################

	$ trema run packetin-dumper.rb -c packetin-dumper.conf

* Start a Packet-In dumper controller process
* This also starts a virtual network (= one virtual switch + virtual hosts host1 and host2)


<!SLIDE small>
## Exercise: Displaying Packet-In Message Dump #################################

	$ trema send_packets --source host1 --dest host2

* Open an another terminal, then send a test packet from host1 to host2
* This causes the controller to dump the packet-in message


<!SLIDE>
## Q: How did you do that (sending test packets)? ##############################


<!SLIDE small>
# Virtual Host And Virtual Link ################################################

## Connect virtual hosts (host1, host2) to the virtual switch 0xabc

	@@@ ruby
	# Add one virtual switch
	vswitch { dpid "0xabc" }
	# Add two virtual hosts
	vhost "host1"
	vhost "host2"
	# Then connect them to the switch 0xabc
	link "0xabc", "host1"
	link "0xabc", "host2"

## Sending test packets between virtual hosts defined above

	$ trema send_packets --source host1 --dest host2


<!SLIDE small>
# Network Configuration File ###################################################

* Simple and easy test environment
* You can specify and construct any arbitrary network by just writing configurations in DSL
* Also you can send test packet by one simple command


<!SLIDE small>
# Example: More Complicated Network ############################################

	@@@ ruby
	vswitch { dpid "0x1" }
	vswitch { dpid "0x2" }
	...
	vhost "host1"
	vhost "host2"
	vhost "host3"
	vhost "host4"
	  ...    
	link "0x1", "0x2
	  ...    
	link "0x1", "host1"
	link "0x1", "host2"
	link "0x2", "host3"
	link "0x2", "host4"
	  ...    


<!SLIDE>
# Handling Packet-In ###########################################################


<!SLIDE smaller>
# `PacketinDumper#packet_in` ###################################################

	@@@ ruby
	# packetin-dumper.rb    
	class PacketinDumper < Controller
	  def packet_in dpid, message
	    info "received a packet_in"
	    info "dpid: #{ datapath_id.to_hex }"
	    info "in_port: #{ message.in_port }"
	  end
	end

* `packet_in`: arguments are dpid and a Packet-In message (`message`)
* `message.name` for inspecting the attributes of the Packet-In message


<!SLIDE smaller>
# Exercise: Inspecting Other Packet-In Attributes ##############################

	@@@ ruby
	# packetin-dumper.rb    
	class PacketinDumper < Controller
	  def packet_in dpid, message
	    info "received a packet_in"
	    info "dpid: #{ datapath_id.to_hex }"
	    info "in_port: #{ message.in_port }"
	    info "total_len: #{ message.total_len }"        
	      ...        
	  end
	end

* Display other Packet-In attributes (total_len, macsa, macda ...)
* Hint: Use `trema ruby` for the reference of PacketIn class
