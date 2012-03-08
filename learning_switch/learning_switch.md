<!SLIDE small>
# Task D: Learning Switch ######################################################

## flow\_mod と packet\_out の送信


<!SLIDE smaller>
# やってみよう: パケットの統計情報 #############################################

## L2 スイッチコントローラを起動

	$ trema run learning-switch.rb -c learning-switch.conf

<br />
<br />

## 別ターミナルでテストパケットを送信し、
## 送受信したパケットの統計情報を表示
	
	$ trema send_packet --source host1 --dest host2
	$ trema show_stats host1
	$ trema show_stats host2


<!SLIDE small>
# やってみよう: フローテーブルの表示 ###########################################

	$ trema send_packet --source host2 --dest host1
	$ trema dump_flows 0xabc

## スイッチに書き込まれたフローエントリを表示


<!SLIDE small>
# 便利なサブコマンド ###########################################################

* `trema show_stats` ホスト名
* `trema dump_flows` スイッチ名

## 各種統計情報や内部情報をコマンド一発で簡単に


<!SLIDE small>
# Learning Switch のソース #####################################################


<!SLIDE smaller>
# Learning Switch ##############################################################

	@@@ ruby
	class LearningSwitch < Controller
	  # ...
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
	  # ...
	end

# まるで疑似コード (？)


<!SLIDE smaller>
# 声に出して読もう! ############################################################

	@@@ ruby
	class LearningSwitch < Controller
	  # ...
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
	  # ...
	end

* パケットインが届いたら、パケットを出したホストのマックアドレスとポート番号を FDB で学習する
* 宛先の MAC アドレスからポート番号を FDB で調べ、もしみつかった場合にはフローテーブルを書き込んでパケットアウトする
* みつからなかったらパケットをフラッディングする


<!SLIDE smaller>
# プライベートメソッド #########################################################

* `flow_mod`, `packet_out`, `flood` は Trema API ではなく、自分で定義したプライベートメソッド
* プライベートメソッドを適切に定義することで、コントローラのコードを疑似コード並みに分かりやすくできる


<!SLIDE smaller>
# Flow-Mod #####################################################################

	@@@ ruby
	class LearningSwitch < Controller
	  # ...
	  private
	  def flow_mod dpid, message, port_no
	    send_flow_mod_add(
	      dpid,
	      :match => ExactMatch.from( message ),
	      :actions => ActionOutput.new( port_no )
	    )
	  end
	  # ...
	end

* "`message` とエグザクトマッチするパケットを `port_no` へ転送する" フローをスイッチ `dpid` に書き込む。
* 短い


<!SLIDE smaller>
# Syntactic Sugar: `ExactMatch.from()` #########################################

	@@@ ruby
	ExactMatch.from( message )

# vs.

	@@@ ruby
	Match.new(
	  :in_port => message.in_port,
	  :nw_src => message.nw_src,
	  :nw_dst => message.nw_dst,
	  :tp_src => message.tp_src,
	  :tp_dst => message.tp_dst,
	  :dl_src => message.dl_src,
	  :dl_dst => message.dl_dst,
	  ...
	)


<!SLIDE smaller>
# Trema vs. NOX Python #########################################################

	@@@ ruby
	# Trema
	send_flow_mod_add(
	  dpid,
	  :match => ExactMatch.from( message ),
	  :actions => ActionOutput.new( port_no )
	)

# vs.

	@@@ python
	# NOX Python
	inst.install_datapath_flow(
	  dpid,
	  extract_flow(packet),
	  CACHE_TIMEOUT, 
	  openflow.OFP_FLOW_PERMANENT,
	  [[openflow.OFPAT_OUTPUT, [0, prt[0]]]],
	  bufid,
	  openflow.OFP_DEFAULT_PRIORITY,
	  inport,
	  buf
	)


<!SLIDE small>
# Learning Switch まとめ #######################################################


* 統計情報を表示: `trema show_stats`, `trema dump_flows`
* 短く書ける API: `ExactMatch.from`, `send_flow_mod_add`
* 次はいよいよ最後のタスクです!
