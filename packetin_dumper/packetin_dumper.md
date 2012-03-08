<!SLIDE small>
# Task C: PacketIn Dumper ######################################################

## いよいよ Packet-In をハンドリング


<!SLIDE small>
# やってみよう: Packet-In の表示 ###############################################

	$ trema run packetin-dumper.rb -c packetin-dumper.conf

* Packet-In をダンプするコントローラを起動
* 仮想スイッチ + 仮想ホスト host1, host2 も起動


<!SLIDE small>
# やってみよう: Packet-In の表示 ###############################################

	$ trema send_packets --source host1 --dest host2

* 別ターミナルで host1 → host2 にテストパケットを送信
* → コントローラがパケットダンプを表示


<!SLIDE>
## Q: どうやってテストパケットを送ったの？ #####################################


<!SLIDE small>
# 仮想ホストと仮想リンク #######################################################

## 仮想スイッチ 0xabc に仮想ホスト (host1, host2) を接続

	@@@ ruby
	# packetin-dumper.conf    
	vswitch { dpid "0xabc" }
	# 仮想ホストを 2 台定義
	vhost "host1"
	vhost "host2"
	# それぞれ仮想スイッチにつなぐ
	link "0xabc", "host1"
	link "0xabc", "host2"

## 定義した仮想ホスト間でテストパケットを送信

	$ trema send_packets --source host1 --dest host2


<!SLIDE small>
# ネットワーク設定ファイル #####################################################

* 単純で手軽なテスト環境
* DSL を書くだけで手軽に好きなネットワークが組める
* コマンド一発でテストパケットを送れる


<!SLIDE small>
# 課題: 複雑なネットワークを作ろう #############################################

	@@@ ruby
	vswitch { dpid "0x1" }
	vswitch { dpid "0x2" }
	...
	vhost "host1"
	vhost "host2"
	vhost "host3"
	vhost "host4"
	  ...    
	link "0x1", "0x2
	  ...    
	link "0x1", "host1"
	link "0x1", "host2"
	link "0x2", "host3"
	link "0x2", "host4"
	  ...    


<!SLIDE>
# Packet-In に反応する #########################################################


<!SLIDE smaller>
# `PacketinDumper#packet_in` ###################################################

	@@@ ruby
	# packetin-dumper.rb    
	class PacketinDumper < Controller
	  def packet_in dpid, message
	    info "received a packet_in"
	    info "dpid: #{ datapath_id.to_hex }"
	    info "in_port: #{ message.in_port }"
	  end
	end

* `packet_in`: 引数は dpid と Packet-In メッセージ本体 (`message`)
* `message.name` で Packet-In の中身を調べることができる


<!SLIDE small>
# PacketIn Dumper まとめ #######################################################

* DSL で手軽に仮想ネットワークを組みコントローラを実行
* コマンド一発でテストパケットを送信
* `PacketIn#in_port` 等で Packet-In の中身を調べられる


<!SLIDE smaller>
# 課題: Packet-In をもっと調べよう #############################################

	@@@ ruby
	# packetin-dumper.rb    
	class PacketinDumper < Controller
	  def packet_in dpid, message
	    info "received a packet_in"
	    info "dpid: #{ datapath_id.to_hex }"
	    info "in_port: #{ message.in_port }"
	    info "total_len: #{ message.total_len }"        
	      ...        
	  end
	end

* Packet-In の中身をもっと表示してみよう (total_len, macsa, macda ...)
* ヒント: `trema ruby` で PacketIn クラスのメソッドを調べよう
