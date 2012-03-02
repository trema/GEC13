<!SLIDE small>
# Task B: Switch Monitor #######################################################

## スイッチの死活監視


<!SLIDE commandline>
## やってみよう: スイッチの死活監視 #####################################################

	@@@ ruby
	# switch-monitor.conf
	vswitch { dpid "0x1" }    	
	vswitch { dpid "0x2" }    	
	vswitch { dpid "0x3" }    	


	$ trema run switch-monitor.rb -c switch-monitor.conf
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	  ...


<!SLIDE commandline>
## やってみよう: スイッチを殺す #########################################################

### trema kill や trema up でスイッチを殺す/再起動する
### trema run のターミナルの出力はどうなる？

	$ trema run switch-monitor.rb -c switch-monitor.conf
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	  ...
	
	$ trema kill 0x1
	$ trema up 0x1

### TODO: trema up の実装


<!SLIDE smaller>
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


<!SLIDE smaller>
# スイッチ一覧の管理 ###############################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  ...
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
	  ...      
	end

* @switches は生きているスイッチの一覧を持つインスタンス変数
* [] は Ruby のリスト (<< で要素の追加、-= で削除)
* switch_disconnected: スイッチが切断したときに呼ばれるハンドラ


<!SLIDE smaller>
# `switch_connected`, `disconnected` ###########################################

* OpenFlow プロトコルの詳細を Trema が隠蔽
* 短く書けるようにするための工夫

<pre>
            switch                        controller
              |                                |
              | secchan                        |
              |------------------------------->|
              |                                |
              |                          HELLO |
              |&lt;-------------------------------|
              | HELLO                          |
              |------------------------------->|
              |                                |
              |               FEATURES REQUEST |
              |&lt;-------------------------------|
              | FEATURES REPLY                 |
              |------------------------------->|
              |                                |
              |                           Init |
              |&lt;-------------------------------|
              |                                |
              |                                | switch_connected
</pre>


<!SLIDE smaller>
# スイッチ一覧の表示 ################################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  # ...
	      
	  private
	  def show_switches
	    info "All switches = " + @switches.sort.join( ", " )
	  end
	end

* periodic\_timer\_event: 10 秒ごとに、show_switches メソッドを呼ぶ
* private 以降はプライベートメソッド
* show\_switches: スイッチのリストをソートして出力


<!SLIDE smaller>
# 課題: 表示タイミングの変更 ##########################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  # ...
	end

* 一秒間隔でスイッチ一覧を更新するには？
* スイッチ一覧に変化があったときだけ表示するようにするには？
