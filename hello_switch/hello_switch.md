<!SLIDE small>
# Task A: Hello Switch #########################################################

## いよいよ OpenFlow スイッチをコントローラに接続


<!SLIDE small>
# スイッチを接続? ##############################################################


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


<!SLIDE commandline>
## やってみよう: Hello Switch ######################################################

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


<!SLIDE small>
# `switch_ready` ハンドラ #########################################################

	@@@ ruby
	class HelloSwitch < Controller
	  # スイッチが接続すると呼ばれる
	  def switch_ready dpid
	    info "Hello #{ dpid.to_hex }!"
	  end
	end

* スイッチがコントローラに接続したときに呼ばれるハンドラ
* 引数は接続したスイッチの datapath ID (整数)
* `.to_hex` で 16 進形式の文字列に変換


<!SLIDE smbullets commandline>
## Network DSL #################################################################

### 設定ファイルとしてネットワーク構成を記述
### (仮想スイッチ、仮想ホスト、仮想リンク)

	@@@ ruby
	# hello-switch.conf
	vswitch { dpid "0xabc" }    	

	$ trema run hello-switch.rb -c hello-switch.conf
	Hello 0xabc!


<!SLIDE small>
# ポイント: Trema はフルスタック #######################################################

* Trema には開発に必要なツールがすべて入っている
* OpenFlow スイッチを持っていなくても、ノート PC 一台で開発やデバッグができる
* スイッチを持っている人にとっても、便利な機能


<!SLIDE commandline>
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
