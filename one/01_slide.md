!SLIDE small
# Example #5: Traffic Monitor ##################################################

## flow_removed の捕捉


!SLIDE smaller
# やってみよう: トラフィックの表示 ########################################################

	$ trema run traffic-monitor.rb -c traffic-monitor.conf
	  ...
	
	$ trema send_packet --source host1 --dest host2
	$ trema send_packet --source host2 --dest host3
	$ trema send_packet --source host3 --dest host1


!SLIDE smaller
# Traffic Monitor ##############################################################

	@@@ ruby
	class TrafficMonitor < Controller
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
	    @counter.add msg.match.dl_src, msg.byte_count
	  end
	
	  private
	  # ...
	end


!SLIDE smaller
# Traffic Monitor (Cont'd) #####################################################

	@@@ ruby
	class TrafficMonitor < Controller
	  # ...
	
	  private
	
	  def show_counter
	    puts Time.now
	    @counter.each_pair do | mac, nbytes |
	      puts "#{ mac } #{ nbytes } bytes"
	    end
	  end
	
	  def flow_mod dpid, macsa, macda, out_port
	    send_flow_mod_add(
	      dpid,
	      :hard_timeout => 10,
	      :match => Match.new( :dl_src => macsa, :dl_dst => macda ),
	      :actions => ActionOutput.new( out_port )
	    )
	  end
	
	  # ...
	end

!SLIDE commandline
## TODO: コードの説明 ##############################################################

### @counter
### flow_removed
### hard_timeout
### Match.new
### show_counter


!SLIDE
# まとめ ##########################################################################


!SLIDE
# 今日説明しなかったこと #############################################################


!SLIDE
# 情報源 ########################################################################


!SLIDE 
# Questions? ###################################################################
