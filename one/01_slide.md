!SLIDE 
# Trema Tutorial (GEC13) #######################################################
## Yasuhito Takamiya
## @yasuhito


!SLIDE small
# Hello Trema ##################################################################

	@@@ ruby
	class HelloController < Controller
	  def switch_ready dpid
	    info "Hello %#x from #{ ARGV[ 0 ] }!" % dpid
	  end
	end


!SLIDE smaller
# Switch Monitor ###############################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  def start
	    @switches = []
	  end
	
	  def switch_ready dpid
	    @switches << dpid.to_hex
	    info "Switch #{ dpid.to_hex } is UP"
	  end
	
	  def switch_disconnected dpid
	    @switches -= [ dpid.to_hex ]
	    info "Switch #{ dpid.to_hex } is DOWN"
	  end
	
	  private
	  def show_switches
	    info "All switches = " + @switches.sort.join( ", " )
	  end
	end


!SLIDE smaller
# Handling Packet-in ###########################################################

	@@@ ruby
	class PacketinDumper < Controller
	  def packet_in dpid, event
	    puts "received a packet_in"
	    info "dpid: #{ datapath_id.to_hex }"
	    info "transaction_id: #{ event.transaction_id.to_hex }"
	    info "buffer_id: #{ event.buffer_id.to_hex }"
	    info "total_len: #{ event.total_len }"
	    info "in_port: #{ event.in_port }"
	    info "reason: #{ event.reason.to_hex }"
	    info "data: #{ event.data.unpack "H*" }"
	  end
	end


!SLIDE smaller
# FDB ##########################################################################

	@@@ ruby
	class FDB
	  def initialize
	    @db = {}
	  end
	
	  def lookup mac
	    @db[ mac ]
	  end
	
	  def learn mac, port_number
	    @db[ mac ] = port_number
	  end
	end


!SLIDE smaller
# Learning Switch ##############################################################

	@@@ ruby
	class LearningSwitch < Trema::Controller
	  def start
	    @fdb = FDB.new
	  end
	
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
	
	  private
	  # ...
	end


!SLIDE smaller
# Learning Switch (Cont'd) #####################################################

	@@@ ruby
	class LearningSwitch < Trema::Controller
	  private
	  def flow_mod dpid, message, port_no
	    send_flow_mod_add(
	      dpid,
	      :match => ExactMatch.from( message ),
	      :actions => Trema::ActionOutput.new( port_no )
	    )
	  end

	  def packet_out dpid, message, port_no
	    send_packet_out(
	      dpid,
	      :packet_in => message,
	      :actions => Trema::ActionOutput.new( port_no )
	    )
	  end
	
	  def flood dpid, message
	    packet_out dpid, message, OFPP_FLOOD
	  end
	end


!SLIDE smaller
# Traffic Monitor ##############################################################

	@@@ ruby
	class TrafficMonitor < Trema::Controller
	  periodic_timer_event :show_counter, 10
	
	  def start
	    @counter = Counter.new
	    @fdb = FDB.new
	  end

	  def packet_in dpid, message
	    macsa = message.macsa
	    macda = message.macda
	    @fdb.learn macsa, message.in_port
	    @counter.add macsa, 1, message.total_len
	    out_port = @fdb.lookup( macda )
	    if out_port
	      packet_out dpid, message, out_port
	      flow_mod dpid, macsa, macda, out_port
	    else
	      flood dpid, message
	    end
	  end
	
	  def flow_removed dpid, msg
	    @counter.add msg.match.dl_src, msg.packet_count, msg.byte_count
	  end
	
	  private
	  # ...
	end


!SLIDE smaller
# Traffic Monitor (Cont'd) #####################################################

	@@@ ruby
	class TrafficMonitor < Trema::Controller
	  # ...
	
	  private
	
	  def show_counter
	    puts Time.now
	    @counter.each_pair do | mac, counter |
	      puts "#{ mac } #{ counter[ :packet_count ] } packets (#{ counter[ :byte_count ] } bytes)"
	    end
	  end
	
	  def flow_mod dpid, macsa, macda, out_port
	    send_flow_mod_add(
	      dpid,
	      :hard_timeout => 10,
	      :match => Match.new( :dl_src => macsa, :dl_dst => macda ),
	      :actions => Trema::ActionOutput.new( out_port )
	    )
	  end
	
	  # ...
	end


!SLIDE 
# Questions? ###################################################################
