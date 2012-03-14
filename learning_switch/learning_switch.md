<!SLIDE small>
# Task D: Learning Switch ######################################################

## Sending flow\_mod and packet\_out


<!SLIDE smaller>
# Exercise: Packet Stats Info ##################################################

## Launch a L2 switch controller:

	$ trema run learning-switch.rb -c learning-switch.conf

<br />
<br />

## Send a test packet on another terminal,
## then `show_stats` to view sent/received packet stats information
	
	$ trema send_packet --source host1 --dest host2
	$ trema show_stats host1
	$ trema show_stats host2


<!SLIDE center>
![overview](show_stats.png)


<!SLIDE small>
# Exercise: View the Flow Table ####################################################

	$ trema send_packet --source host2 --dest host1
	$ trema dump_flows 0xabc

## The above command displays the flow-table entries stored in the switch `0xabc`


<!SLIDE center>
![overview](dump_flows.png)


<!SLIDE small>
# Useful Sub-Commands ##########################################################

* `trema show_stats HOST_NAME`
* `trema dump_flows SWITCH_NAME`

## Various stats and internal info can be displayed with one simple command


<!SLIDE small>
# Source Code: Learning Switch #################################################


<!SLIDE smaller>
# Learning Switch ##############################################################

	@@@ ruby
	class LearningSwitch < Controller
	  # ...
	  def packet_in dpid, message
	    @fdb.learn message.macsa, message.in_port
	    port_no = @fdb.lookup( message.macda )
	    if port_no
	      flow_mod dpid, message, port_no
	      packet_out dpid, message, port_no
	    else
	      flood dpid, message
	    end
	  end
	  # ...
	end

# Reads smoothly like a pseudo code!?


<!SLIDE smaller>
# Read It Aloud! ###############################################################

	@@@ ruby
	class LearningSwitch < Controller
	  # ...
	  def packet_in dpid, message
	    @fdb.learn message.macsa, message.in_port
	    port_no = @fdb.lookup( message.macda )
	    if port_no
	      flow_mod dpid, message, port_no
	      packet_out dpid, message, port_no
	    else
	      flood dpid, message
	    end
	  end
	  # ...
	end

* When a packet comes in as a packet-in message, make FDB learn its macsa and the in_port
* Look up the destination's port number from packet's macda
* If found: update switch's flow-table and do packet-out the packet
* If not-found: flood the packet


<!SLIDE smaller>
# Private Methods ##############################################################

* Actually `flow_mod`, `packet_out`, `flood` are not part of the Trema API, but  defined as private methods here
* Proper use of names for private methods makes your code clean and can be read like pseudo-code


<!SLIDE smaller>
# Flow-Mod #####################################################################

	@@@ ruby
	class LearningSwitch < Controller
	  # ...
	  private
	  def flow_mod dpid, message, port_no
	    send_flow_mod_add(
	      dpid,
	      :match => ExactMatch.from( message ),
	      :actions => ActionOutput.new( port_no + 1 )
	    )
	  end
	  # ...
	end

* Echoes the packet that exactly matches with `message` to the next port
* Short and clean


<!SLIDE smaller>
# Syntactic Sugar: `ExactMatch.from()` #########################################

	@@@ ruby
	ExactMatch.from( message )

# vs.

	@@@ ruby
	Match.new(
	  :in_port => message.in_port,
	  :nw_src => message.nw_src,
	  :nw_dst => message.nw_dst,
	  :tp_src => message.tp_src,
	  :tp_dst => message.tp_dst,
	  :dl_src => message.dl_src,
	  :dl_dst => message.dl_dst,
	  ...
	)


<!SLIDE smaller>
# Trema vs. NOX Python #########################################################

	@@@ ruby
	# Trema
	send_flow_mod_add(
	  dpid,
	  :match => ExactMatch.from( message ),
	  :actions => ActionOutput.new( port_no )
	)

# vs.

	@@@ python
	# NOX Python
	inst.install_datapath_flow(
	  dpid,
	  extract_flow(packet),
	  CACHE_TIMEOUT, 
	  openflow.OFP_FLOW_PERMANENT,
	  [[openflow.OFPAT_OUTPUT, [0, prt[0]]]],
	  bufid,
	  openflow.OFP_DEFAULT_PRIORITY,
	  inport,
	  buf
	)


<!SLIDE small>
# Learning Switch: Summary #####################################################

* Stats info: `trema show_stats`, `trema dump_flows`
* "Write it short" API: `ExactMatch.from`, `send_flow_mod_add`
