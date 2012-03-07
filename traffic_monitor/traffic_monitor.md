<!SLIDE small>
# Task E: Traffic Monitor ######################################################

## flow_removed からトラフィック情報を取得


!SLIDE smaller
# やってみよう: トラフィックの表示 #############################################

	$ trema run traffic-monitor.rb -c traffic-monitor.conf
	  ...
	
	$ trema send_packet --source host1 --dest host2
	$ trema send_packet --source host2 --dest host3
	$ trema send_packet --source host3 --dest host1


<!SLIDE smaller>
# トラフィック量の取得 #########################################################

	@@@ ruby
	class TrafficMonitor < Controller
	  # ...
	  def flow_removed dpid, message
	    @counter.add message.match.dl_src, message.byte_count
	  end
	  # ...
	end

* フローが消えるときに出る flow_removed メッセージをハンドル
* これにはフローで転送されたトラフィック量などの情報が乗っている
* カウンタ `@counter` にこれを記録


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
