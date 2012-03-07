!SLIDE small
# Task C: PacketIn Dumper ######################################################

## いよいよパケットインのハンドリング


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
