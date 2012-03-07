<!SLIDE small>
# Task B: Switch Monitor #######################################################

## スイッチの死活監視


<!SLIDE small>
# やってみよう: スイッチの死活監視 #############################################

	$ trema run switch-monitor.rb -c switch-monitor.conf
	Switch 0x3 is UP
	Switch 0x2 is UP
	Switch 0x1 is UP
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	  ...

* ネットワーク中のスイッチ (.conf に記述) を検知
* 10 秒ごとにスイッチ一覧を更新


<!SLIDE small>
# やってみよう: スイッチを殺すと？ #############################################

	$ trema kill 0x1  # スイッチ 0x1 を殺す
	$ trema up 0x1    # スイッチ 0x1 を起動する

* 別ターミナルを開き、trema kill や trema up を実行
* trema run の出力はどうなる？


<!SLIDE small>
# スイッチの切断を捕捉 #########################################################


<!SLIDE small>
# `SwitchMonitor#switch_disconnected` ##########################################

	@@@ ruby
	class SwitchMonitor < Controller
	  # ...	
	
	  def switch_disconnected dpid
	    @switches -= [ dpid.to_hex ]
	    info "Switch #{ dpid.to_hex } is DOWN"
	  end

	  # ...	
	end

* `switch_ready` と同じ構造
* `@switches` については後述


<!SLIDE small>
# スイッチの一覧の管理 #########################################################


<!SLIDE smaller>
# `switch-monitor.rb` ##########################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  # ...
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
	  # ...
	end

* `@switches` は生きているスイッチの一覧を持つインスタンス変数
* `[]` は Ruby のリスト。`<<` で要素の追加、`-=` で削除


<!SLIDE small>
# スイッチの一覧の表示 #########################################################


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

* `periodic_timer_event`: 10 秒ごとに、show_switches メソッドを呼ぶ
* `show_switches`: スイッチのリストをソートして出力
* `private` 以降はプライベートメソッド


<!SLIDE small>
# 完成形 #######################################################################


<!SLIDE smaller>
# switch-monitor.rb ############################################################

## 「必要なことだけ」が書かれた、書きやすく読みやすいコード

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
# 課題: 表示タイミングの変更 ##########################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  # ...
	end

* 一秒間隔でスイッチ一覧を更新するには？
* スイッチ一覧に変化があったときだけ表示するようにするには？
