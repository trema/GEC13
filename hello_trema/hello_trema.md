<!SLIDE commandline>
## やってみよう: Hello Trema! ##################################################

	$ cd Tutorials/Trema
	$ trema run hello-trema.rb
	Hello Trema!   (Ctrl-C to quit)


<!SLIDE small>
# `trema run` ##################################################################

* コンパイル無しで、すぐにコントローラを起動できる
* Ctrl-C で終了
* 「実装 → 動作テスト → ...」のサイクルをタイトに回せる


<!SLIDE small>
# `trema run` の裏側 ############################################################

	$ trema run hello-trema.rb -v
	.../trema/objects/switch_manager/switch_manager \
	  --daemonize --port=6633 -- port_status::HelloTrema \
	  packet_in::HelloTrema state_notify::HelloTrema \
	  vendor::HelloTrema
	Hello Trema!
	^C
	terminated
	Shutting down switch_manager...
	kill 29703 2>/dev/null

* `trema run` は Trema 内部の複雑さを隠蔽
* スイッチとの接続に必要なすべてのデーモンを起動
* Ctrl-c ですべて停止


<!SLIDE small>
# コントローラの書き方の基本


<!SLIDE small>
# hello-trema.rb ###############################################################

	@@@ ruby
	class HelloController < Controller
	  def start
	    info "Hello Trema!"
	  end
	end

## すべてのコントローラのテンプレート


<!SLIDE small>
# Controller Class #############################################################

	@@@ ruby
	class HelloController < Controller
	  # ...
	end

* コントローラクラスは Controller クラスを継承
* コントローラに必要なメソッドがすべて取り込まれる


<!SLIDE small>
# Event Handlers ###############################################################

	@@@ ruby
	class MyController < Controller
	  def start  # コントローラの起動時に呼ばれる
	    # ...
	  end
	      
	  def packet_in dpid, msg  # packet-in 到着時に呼ばれる
	    # ...
	  end
      
	end

* 各種イベントのハンドラをメソッドとして定義
* イベント到着時に自動的にディスパッチ


<!SLIDE small>
# 比較: Floodlight #############################################################

	@@@ java
	public Command receive(IOFSwitch sw, ...) {
	  switch (msg.getType()) {
	    case PACKET_IN:
	      return this.handlePacketIn(sw, ...);
	    ...
	private Command handlePacketIn(IOFSwitch sw, ...) {
	    ...

* ハンドラを明示的にディスパッチする必要がある
* NOX も同様 ... これはとても面倒!


<!SLIDE small>
# Convention over Coding #######################################################

	@@@ ruby
	class MyController < Controller    
	  def packet_in dpid, msg  # これだけ!
	    ...
	  end
	end

* コーディングよりも規約
* ディスパッチなど決まりきったコードを書かなくて済む
* Trema の規約: 「イベントハンドラ名 = イベントの名前」


<!SLIDE small>
# 短く書こう ###################################################################

## プログラムは短く (== 少ない構文要素で) 書きたい

* プログラムの短さと生産性には強い相関
* タイプ数が少なければ、早く書けて早く読める
* e.g. Arc Programming Language [Paul Graham]

## Trema は速度よりも「いかに早く作れるか」に特化


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


<!SLIDE small transition=fade>
# イントロを終えて #############################################################

* Trema は Rails と同様の「モダン」な開発環境
* 以降では Hello World に機能を付け足して、最終的に高機能なコントローラを作ります
* その都度 Trema の強力な機能を使いながら説明します
