<!SLIDE small transition=toss>
# さっそくやってみよう! ########################################################

# 定番の Hello World


<!SLIDE commandline transition=toss>
# Hello Trema! #################################################################

### 次のコマンドを打ち込んで Trema を動かしてみよう

	$ cd Tutorials/Trema
	$ trema run hello-trema.rb
    Password: gec13user
	Hello Trema!   # Ctrl-C to quit


<!SLIDE small>
# `trema run` ##################################################################

	$ trema run [コントローラファイル (.rb)]

* コントローラを起動
* Ctrl-c で終了
* オプション一覧は `trema help run` を実行


<!SLIDE small>
# `trema run` の裏側 ############################################################

## `-v` オプションを付けて実行してみると...

	$ trema run hello-trema.rb -v
	.../trema/objects/switch_manager/switch_manager \
	  --daemonize --port=6633 -- port_status::HelloTrema \
	  packet_in::HelloTrema state_notify::HelloTrema \
	  vendor::HelloTrema
	Hello Trema!

* 動作に必要なデーモンやプロセスをいろいろと起動
* Ctrl-c ですべてを停止
* Trema 内部の複雑さをユーザから隠蔽


<!SLIDE small transition=fadeZoom>
# Run It Quick #################################################################

* コマンド一発で簡単にコントローラを起動・停止
* 書いたらすぐ実行して試せる
* 「実装 → 動作テスト → ...」のサイクルをタイトに回せる


<!SLIDE small transition=toss>
# Trema でのコントローラの書き方 ###############################################
# 基本編


<!SLIDE small transition=toss>
# hello-trema.rb ###############################################################

	@@@ ruby
	class HelloController < Controller
	  def start
	    info "Hello Trema!"
	  end
	end

## 最も簡単な、ほとんど何もしないコントローラ


<!SLIDE small>
# Controller クラス ############################################################

	@@@ ruby
	class HelloController < Controller
	  # ...
	end

* すべてのコントローラはクラスとして定義 (`HelloController`)
* Controller クラスを継承
* → 必要な機能がすべて取り込まれる


<!SLIDE small>
# イベントハンドラ #############################################################

	@@@ ruby
	class MyController < Controller
	  def start  # 起動時のハンドラ
	    # ...
	  end
	      
	  def packet_in dpid, msg  # Packet-in 到着時のハンドラ
	    # ...
	  end
	
	  # ...
	end

* コントローラはイベントドリブンモデル
* 各種イベントに対するハンドラはメソッドとして定義


<!SLIDE small>
# Floodlight でのイベントハンドリング ##########################################

	@@@ java
	public Command receive(IOFSwitch sw, ...) {
	  switch (msg.getType()) {
	    case PACKET_IN:
	      return this.handlePacketIn(sw, ...);
	    ...
	private Command handlePacketIn(IOFSwitch sw, ...) {
	    ...

* ハンドラを明示的にディスパッチする必要がある
* フレームワークが自動でやってくれればいいのに...


<!SLIDE small>
# NOX Python でのイベントハンドリング ##########################################

	@@@ python
	class pyswitch(Component):
	    ...
	
	    def install(self):
	        inst.register_for_packet_in(packet_in_callback)
	        inst.register_for_datapath_leave(datapath_leave_callback)
	        inst.register_for_datapath_join(datapath_join_callback)
	    ...

* ハンドラを明示的に登録する必要がある
* こういう長いのを何度も書かされるのは退屈


<!SLIDE small>
# 自動ディスパッチ #############################################################

	@@@ ruby
	# Trema の場合
	class MyController < Controller
	  def start  # 起動時に自動で呼ばれる
	    # ...
	  end
	      
	  def packet_in dpid, msg  # Packet-in 到着時に自動で呼ばれる
	    # ...
	  end
	end

* イベントに応じたハンドラがあれば、自動的に呼ばれる
* ハンドラの登録やディスパッチ処理は書かなくて良い


<!SLIDE small transition=fadeZoom>
# Convention over Coding #######################################################

## Trema の設計思想: 「コーディングよりも規約」

* 例: イベントハンドラ名 == イベントの名前
* ディスパッチ処理などの定型的なコードを大幅に省ける
* 退屈な部分を減らして楽しくプログラミング!


<!SLIDE small transition=toss>
# 短く書こう ###################################################################

* プログラムの短さ (トークン数) と生産性には強い相関
* e.g. Arc Programming Language [Paul Graham]
* タイプ数が少ないほど早く書けて早く読め、バグらない

<br />

## Trema は速度よりも<b>「いかに早く作れるか」</b>に特化


<!SLIDE small>
# Logging API ##################################################################

	@@@ ruby
	class HelloController < Controller
	  def start
	    info "Hello Trema!"	 # info レベルのメッセージを出力
	  end
	end

* ロギング API (debug, info,... など)
* 詳しくは API ドキュメントを参照 (`trema ruby` コマンド)


<!SLIDE>
# イントロを終えて #############################################################


<!SLIDE small incremental transition=uncover>
# 実行速度よりも生産性 #########################################################

## Trema はいわゆる「Rails 以後」のモダンな開発環境

<br />

* <i>Run It Quick</i>: 書いたらすぐ実行しよう
* <i>Convention over Coding</i>: 短く書こう
* 便利なサブコマンド群: `trema ruby` などなど


<!SLIDE small transition=fade>
# チュートリアルの進め方 #######################################################

* Hello Trema! から出発
* 徐々に機能を追加し、最終的に高機能なコントローラを作成
* その都度 Trema の強力な機能を使いながら紹介
