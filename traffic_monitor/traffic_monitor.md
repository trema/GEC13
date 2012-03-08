<!SLIDE small>
# Task E: Traffic Monitor ######################################################

## flow_removed からトラフィック情報を取得


<!SLIDE smaller>
# やってみよう: トラフィックの表示 #############################################

	$ trema run traffic-monitor.rb -c traffic-monitor.conf


	# 別ターミナルを開く
	$ trema send_packet --source host1 --dest host2
	$ trema send_packet --source host2 --dest host3
	$ trema send_packet --source host3 --dest host1

* トラフィック監視付き L2 スイッチコントローラを起動
* テストパケットを適当に送る
* ホストごとのパケット送信量が表示される


<!SLIDE smaller>
# トラフィック量の取得 #########################################################

	@@@ ruby
	class TrafficMonitor < Controller
	  # ...
	  def flow_removed dpid, message
	    @counter.add message.match.dl_src, message.byte_count
	  end
	      
	  private
	      
	  def flow_mod dpid, macsa, macda, out_port
	    send_flow_mod_add(
	      dpid,
	      :hard_timeout => 10,  # フローの寿命は 10 秒
	      :match => Match.new( :dl_src => macsa, :dl_dst => macda ),
	      :actions => ActionOutput.new( out_port )
	    )
	  end
	  # ...
	end


* フローの寿命を 10 秒に設定
* フローが消えるときに出る flow_removed メッセージをハンドル
* フローで転送されたトラフィック量などの情報を記録


<!SLIDE smaller>
# トラフィック情報の表示 #######################################################

	@@@ ruby
	class TrafficMonitor < Controller
	  periodic_timer_event :show_counter, 10
	
	  # ...
	
	  private
	
	  def show_counter
	    puts Time.now
	    @counter.each_pair do | mac, nbytes |
	      puts "#{ mac } #{ nbytes } bytes"
	    end
	  end
	
	  # ...
	end

* 10 秒ごとに現在時刻とカウンタの中身を表示


<!SLIDE small>
# Traffic Monitor まとめ #######################################################

* Learning Switch の応用
* flow_removed のトラフィック情報を利用
