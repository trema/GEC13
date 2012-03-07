<!SLIDE small>
# Task A: Hello Switch #########################################################

## いよいよ OpenFlow スイッチをコントローラに接続


<!SLIDE small>
# えっ、スイッチを接続? ########################################################


<!SLIDE small>
# スイッチを持ってないけど大丈夫? ##############################################


<!SLIDE small>
# やってみよう: Hello Switch ###################################################

	$ trema run hello-switch.rb -c hello-switch.conf
	Hello 0xabc!

* dpid = 0xabc のスイッチをコントローラに接続
* 接続を受けたコントローラが `"Hello 0xabc!"` と表示
* スイッチの定義は `-c` で渡した `hello-switch.conf`


<!SLIDE small>
# hello-switch.conf ############################################################

	@@@ ruby
	# dpid == 0xabc のスイッチを定義    
	vswitch { dpid "0xabc" }    	

* このネットワーク設定ファイルを `trema run` の `-c` に渡す
* ソフトウェアスイッチが自動的に起動しコントローラに接続
* Trema は<b>フルスタック</b>: 開発はノート PC 一台で完結


<!SLIDE small>
# `trema run` の裏側 ###########################################################

	$ trema run hello-switch.rb -c hello-switch.conf -v
	.../trema/objects/switch_manager/switch_manager \
	  --daemonize --port=6633 -- port_status::HelloSwitch \
	  packet_in::HelloSwitch state_notify::HelloTrema \
	  vendor::HelloSwitch
	sudo .../trema/objects/openvswitch/bin/ovs-openflowd \
	  --detach --out-of-band --fail=closed \
	  --inactivity-probe=180 --rate-limit=40000 \
	  --burst-limit=20000 ...
	  ...

## たしかに OpenvSwitch が起動されている


<!SLIDE small>
# スイッチの接続を捕捉 #########################################################


<!SLIDE>
# `hello-switch.rb` ############################################################

	@@@ ruby
	class HelloSwitch < Controller
	  def switch_ready dpid
	    info "Hello #{ dpid.to_hex }!"
	  end
	end

### Hello Trema とほとんど同じで簡単!


<!SLIDE small>
# `switch_ready` ハンドラ ######################################################

	@@@ ruby
	class HelloSwitch < Controller
	  def switch_ready dpid  # スイッチが接続すると呼ばれる
	    info "Hello #{ dpid.to_hex }!"
	  end
	end

* スイッチがコントローラに接続したときに呼ばれるハンドラ
* 引数は接続したスイッチの datapath ID (整数)
* `.to_hex` で 16 進形式の文字列に変換


<!SLIDE smaller>
# 参考: `switch_ready` の裏側 ##################################################

## 短く書けるようにするために OpenFlow の詳細を Trema が隠蔽

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


<!SLIDE small>
# 課題: スイッチを増やしてみよう ###############################################

	@@@ ruby
	# hello-switch.conf
	vswitch { dpid "0x1" }
	vswitch { dpid "0x2" }
	vswitch { dpid "0x3" }
	  ...


	$ trema run hello-switch.rb -c hello-switch.conf
	???

* スイッチを増やして hello-switch.rb を起動するとどうなる？
* 注: スイッチの dpid は一意にしてください
