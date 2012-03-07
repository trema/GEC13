!SLIDE small
# Example #4: Learning Switch ##################################################

## flow_mod と packet_out の送信


!SLIDE smaller
# やってみよう: パケットの送受信を確認 ####################################################

	$ trema run learning-switch.rb -c learning-switch.conf
	  ...
	
	$ trema send_packet --source host1 --dest host2
	$ trema show_stats host1
	$ trema show_stats host2


!SLIDE smaller
# やってみよう: フローテーブルの表示 #####################################################

	$ trema run learning-switch.rb -c learning-switch.conf
	  ...
	
	$ trema send_packet --source host1 --dest host2
	$ trema dump_flows 0xabc


!SLIDE commandline
## Why Trema? ##################################################################

### 仮想ネットワークの各種統計も trema コマンド一発で簡単に取れる


!SLIDE smaller
# Learning Switch ##############################################################

	@@@ ruby
	class LearningSwitch < Controller
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

## 疑似コードのようにスラスラ読めますね？


!SLIDE smaller
# Learning Switch (Cont'd) #####################################################

	@@@ ruby
	class LearningSwitch < Controller
	  private
	  def flow_mod dpid, message, port_no
	    send_flow_mod_add(
	      dpid,
	      :match => ExactMatch.from( message ),
	      :actions => ActionOutput.new( port_no )
	    )
	  end

	  def packet_out dpid, message, port_no
	    send_packet_out(
	      dpid,
	      :packet_in => message,
	      :actions => ActionOutput.new( port_no )
	    )
	  end
	
	  def flood dpid, message
	    packet_out dpid, message, OFPP_FLOOD
	  end
	end


!SLIDE commandline
## Why Trema? ##################################################################

### flow_mod や packet_out が簡単に送れる
### TODO: 一方 NOX (Python) では...


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
