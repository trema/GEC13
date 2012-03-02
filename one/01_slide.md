!SLIDE 
# Trema Tutorial ###############################################################
### Yasuhito Takamiya (@yasuhito)
### Yasunobu Chiba
### Hideyuki Shimonishi

### GEC13


!SLIDE small
# Today's Goal #################################################################

* 「トラフィック監視付き L2 スイッチ」を OpenFlow で実現
* Trema の Ruby バインディングを使います
* Trema を使った開発方法がひととおり身に付きます


!SLIDE small
# Why Trema? ###################################################################

* 生産性の高い OpenFlow フレームワーク
* 短く書けて、すぐ動かせる
* 実際に手を動かしながら生産性の高さを体験しよう


!SLIDE small
# チュートリアルの進めかた #############################################################

* やってみよう: 画面のとおりに実行
* 課題: コードを書き換えて実験
* Why Trema?: Trema のすごいところを解説

## MEMO: 何かパッとわかるアイコンがほしいな。


!SLIDE commandline
# Hello Trema ##################################################################

	@@@ ruby
	# hello.rb
	class Hello < Controller
	  def start
	    info "Hello Trema!"
	  end
	end


	$ trema run hello.rb
	Hello Trema!


!SLIDE small
# trema run controller.rb ######################################################

* すぐコントローラを起動できる (コンパイル不要)
* 「実装 → テスト → デバッグ」のサイクルをタイトに回せます
* では Trema でのコントローラの書き方を見て行きます


!SLIDE bullets small
# Controller Class #############################################################

	@@@ ruby
	# Controller クラスを継承した Foo クラス
	class Foo < Controller
	  # ...
	end

## コントローラは Controller クラスを継承したクラスとして定義


!SLIDE bullets small
# Event Handlers ###############################################################

	@@@ ruby
	# コントローラの起動時に呼ばれるハンドラ start を定義
	class Foo < Controller
	  def start
	    # ...
	  end
	end

* 各種イベントのハンドラをメソッドとして定義
* イベント到着時に自動的にディスパッチされる


!SLIDE small
# Why Trema? ###################################################################

## 明示的なディスパッチが不要!

	@@@ ruby
	# Trema
	class MyController < Controller    
	  def packet_in dpid, msg  # これだけ!
	    ...
	  end
	end


## 一方、Floodlight では...

	@@@ java
	// Floodlight
	public Command receive(IOFSwitch sw, ...) {
	  switch (msg.getType()) {
	    case PACKET_IN:
	      return this.handlePacketIn(sw, ...);
	    ...
	private Command handlePacketIn(IOFSwitch sw, ...) {
	    ...


!SLIDE bullets small
# Logging API ##################################################################

	@@@ ruby
	# ログを適切な場所に出力
	class Foo < Controller
	  def start
	    info "Hello Trema!"
	  end
	end


* ロギング API (debug, info,... など)
* ログはディレクトリ xyz に保存される
* 詳しくは "trema ruby" で API ドキュメントを


!SLIDE small
# Example #1: Hello Switch #####################################################

## OpenFlow スイッチをコントローラに接続


!SLIDE commandline
## Hello Switch ################################################################

	@@@ ruby
	# hello-switch.rb
	class HelloSwitch < Controller
	  def switch_ready dpid
	    info "Hello #{ dpid.to_hex }!"
	  end
	end
	
	# hello-switch.conf
	vswitch { dpid "0xabc" }    	


	$ trema run hello-switch.rb -c hello-switch.conf
	Hello 0xabc!


!SLIDE commandline
## switch_ready dpid ###########################################################

	@@@ ruby
	# hello-switch.rb
	class HelloSwitch < Controller
	  def switch_ready dpid
	    info "Hello #{ dpid.to_hex }!"
	  end
	end

### スイッチが接続したときに呼ばれるハンドラ
### 引数は接続したスイッチの datapath ID


!SLIDE commandline
## Network DSL ################################################################

	@@@ ruby
	# hello-switch.conf
	vswitch { dpid "0xabc" }    	


	$ trema run hello-switch.rb -c hello-switch.conf
	Hello 0xabc!

### 仮想ネットワークの構成を記述
### trema run の -c オプションに渡す
### 仮想スイッチ、仮想ホスト、仮想リンクが記述可能 (後述)


!SLIDE commandline
## 課題: スイッチを増やそう ###########################################################

### スイッチを増やして hello-switch.rb を起動するとどうなる？

	@@@ ruby
	# hello-switch.conf
	vswitch { dpid "0x1" }
	vswitch { dpid "0x2" }
	vswitch { dpid "0x3" }
	  ...


	$ trema run hello-switch.rb -c hello-switch.conf
	Hello 0x1!
	Hello 0x2!
	Hello 0x3!
	  ...


!SLIDE small
# Example #2: Switch Monitor ###################################################

## スイッチの接続と切断を監視


!SLIDE commandline
## Switch Monitor ##############################################################

	@@@ ruby
	# switch-monitor.conf
	vswitch { dpid "0x1" }    	
	vswitch { dpid "0x2" }    	
	vswitch { dpid "0x3" }    	


	$ trema run switch-monitor.rb -c switch-monitor.conf
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	  ...


!SLIDE commandline
## やってみよう: スイッチを殺す #########################################################

### 別ターミナルを開き、trema kill や trema up でスイッチを殺す/再起動する
### trema run のターミナルの出力はどうなる？
### TODO: trema up の実装

	$ trema run switch-monitor.rb -c switch-monitor.conf
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	  ...
	
	$ trema kill 0x1
	$ trema up 0x1


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

## @switches はインスタンス変数
## [] は Ruby のリスト (<< で要素の追加、-= で削除)
## switch_disconnected: スイッチが切断したときに呼ばれるハンドラ


!SLIDE smaller
# switch_connected, disconnected ###############################################

## これらは Trema 独自のイベントで、裏でどんな OpenFlow メッセージが飛んでいるかをざっくり説明
## これらのごちゃごちゃを Trema が隠蔽してくれててうれしいねという説明


!SLIDE smaller
# Switch Monitor ###############################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  # ...
	      
	  private
	  def show_switches
	    info "All switches = " + @switches.sort.join( ", " )
	  end
	end

## periodic_timer_event: 10 秒ごとに、show_switches メソッドを呼ぶ
## private 以降はプライベートメソッド
## show_switches: スイッチのリストをソートして出力


!SLIDE smaller
# 課題: タイマー間隔を変更 ##########################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  # ...
	end

## タイマー間隔を変更してよりリアルタイムに更新されるようにしよう
## 逆に、スイッチ一覧に変化があったときだけ一覧を表示するようにするには？


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
	    @counter.add msg.match.dl_src, msg.packet_count, msg.byte_count
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
	    @counter.each_pair do | mac, counter |
	      puts "#{ mac } #{ counter[ :packet_count ] } packets (#{ counter[ :byte_count ] } bytes)"
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


!SLIDE
# まとめ ##########################################################################


!SLIDE 
# Questions? ###################################################################
