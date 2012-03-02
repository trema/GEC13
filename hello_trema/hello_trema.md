<!SLIDE commandline>
## やってみよう: Hello Trema! ######################################################

	@@@ ruby
	# hello.rb
	class Hello < Controller
	  def start
	    info "Hello Trema!"
	  end
	end


	$ trema run hello.rb
	Hello Trema!


<!SLIDE small>
# trema run foo.rb #############################################################

* コンパイル無しで、すぐにコントローラを起動できる
* 「実装 → テスト → デバッグ → ...」のサイクルをタイトに回せる
* では Trema でのコントローラの書き方を見て行こう


<!SLIDE bullets small>
# Controller Class #############################################################

	@@@ ruby
	# Controller クラスを継承した Foo クラス
	class Foo < Controller
	  # ...
	end

## コントローラは Controller クラスを継承したクラスとして定義


<!SLIDE small>
# Event Handlers ###############################################################

	@@@ ruby
	class Foo < Controller
	  # コントローラの起動時に呼ばれる
	  # ハンドラ start を定義
	  def start
	    # ...
	  end
	end

* 各種イベントのハンドラをメソッドとして定義
* イベント到着時に自動的にディスパッチされる
* イベントの種類: コントローラの起動、メッセージの受信、etc.


!SLIDE small
# ポイント: ハンドラの自動ディスパッチ ####################################################

<br />

## Floodlight ではハンドラの明示的なディスパッチが煩雑...

	@@@ java
	public Command receive(IOFSwitch sw, ...) {
	  switch (msg.getType()) {
	    case PACKET_IN:
	      return this.handlePacketIn(sw, ...);
	    ...
	private Command handlePacketIn(IOFSwitch sw, ...) {
	    ...

## Trema ではハンドラを書くと自動的に登録される!

	@@@ ruby
	class Foo < Controller    
	  def packet_in dpid, msg  # これだけ!
	    ...
	  end
	end


!SLIDE small
# Trema の設計思想 ###############################################################

## 同じ処理をより短く (== 少ない構文要素数で) 書けるように

* プログラムの短さと生産性には強い相関
* e.g. Arc Programming Language [Paul Graham]
* 実行速度よりも「いかに早く作れるか」に特化


!SLIDE small
# Logging API ##################################################################

	@@@ ruby
	class Foo < Controller
	  def start
	    # ログを適切な場所に出力
	    info "Hello Trema!"
	  end
	end

* ロギング API (debug, info,... など)
* ログはディレクトリ xyz に保存される
* 詳しくは `trema ruby` コマンドで API ドキュメントを参照
