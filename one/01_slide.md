!SLIDE small
# Example #3: PacketIn Dumper ##################################################

## パケットインの捕捉


!SLIDE smaller
# やってみよう: Packet-in の表示 #####################################################

	@@@ ruby
	# packetin-dumper.conf    
	vswitch { dpid "0xabc" }
	vhost "host1"
	vhost "host2"
	link "0xabc", "host1"
	link "0xabc", "host2"


	$ trema run packetin-dumper.rb -c packetin-dumper.conf
	  ...
	
	$ trema send_packet --source host1 --dest host2


!SLIDE commandline
## Network DSL ################################################################

### 仮想スイッチ 0xabc に仮想ホスト (host1, host2) を接続

	@@@ ruby
	# packetin-dumper.conf    
	vswitch { dpid "0xabc" }
	vhost "host1"
	vhost "host2"
	link "0xabc", "host1"
	link "0xabc", "host2"

### trema send_packet でテストパケットを送信

	$ trema send_packet --source host1 --dest host2


!SLIDE commandline
## Why Trema? ##################################################################

### DSL を書くだけで手軽にネットワークが組める
### コマンド一発でテストパケットを送れる

### 一方 MiniNet では... (TODO)


!SLIDE commandline
## 課題: ホストをたくさんつなごう ########################################################

	@@@ ruby
	# packetin-dumper.conf    
	vswitch { dpid "0xabc" }
	vhost "host1"
	vhost "host2"
	vhost "host3"
	vhost "host4"
	  ...    
	link "0xabc", "host1"
	link "0xabc", "host2"
	link "0xabc", "host3"
	link "0xabc", "host4"
	  ...    


!SLIDE smaller
# Handling Packet-in ###########################################################

	@@@ ruby
	# packetin-dumper.rb    
	class PacketinDumper < Controller
	  def packet_in dpid, msg
	    info "received a packet_in"
	    info "dpid: #{ datapath_id.to_hex }"
	    info "in_port: #{ msg.in_port }"
	  end
	end

## packet_in: パケットイン到着時に呼ばれるハンドラ。引数は dpid とパケットインメッセージ本体
## ハンドラ内でパケットインの中身を調べてあれこれする


!SLIDE commandline
## 課題: パケットインを調べよう #########################################################

	@@@ ruby
	# packetin-dumper.rb    
	class PacketinDumper < Controller
	  def packet_in dpid, msg
	    info "received a packet_in"
	    info "dpid: #{ datapath_id.to_hex }"
	    info "in_port: #{ msg.in_port }"
	    info "total_len: #{ msg.total_len }"        
	      ...        
	  end
	end

### パケットインのアトリビュートをもっと表示してみよう (total_len, macsa, macda など)
### ヒント: trema ruby で API ドキュメントを開き、PacketIn クラスのメソッドを調べよう


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
