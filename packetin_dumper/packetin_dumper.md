<!SLIDE small>
# Task C: PacketIn Dumper ######################################################

## いよいよパケットインのハンドリング


<!SLIDE small>
# やってみよう: Packet-in の表示 ###############################################

## Packet-in をダンプするコントローラを起動

	$ trema run packetin-dumper.rb -c packetin-dumper.conf

## 別ターミナルでテストパケットを送信

	$ trema send_packets --source host1 --dest host2

## → パケットダンプが表示されることを確認


<!SLIDE small>
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

* Mininet よりも単純で手軽なテスト環境
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
# Packet-in に反応する #########################################################


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

* `packet_in`: 引数は dpid と Packet-in メッセージ本体 (`message`)
* `message.name` でパケットインの中身を調べることができる


<!SLIDE smaller>
# 課題: Packet-in をもっと調べよう #############################################

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

* パケットインの中身をもっと表示してみよう (total_len, macsa, macda ...)
* ヒント: `trema ruby` で API ドキュメントを開き、PacketIn クラスのメソッドを調べよう
