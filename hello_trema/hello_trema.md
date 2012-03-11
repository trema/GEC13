<!SLIDE transition=toss>
# Task A: Hello Trema! #########################################################


<!SLIDE commandline transition=toss>
## Exercise: Run "Hello Trema!" ################################################

### Run Trema by typing the following command sequence:

	$ cd Tutorials/Trema
	$ trema run hello-trema.rb
	Hello Trema!   # Ctrl-C to quit


<!SLIDE small>
# `trema run` ##################################################################

	$ trema run [controller-file (.rb)]

* Starts a controller process
* Ctrl-c to quit
* Full options list is available by `trema help run`


<!SLIDE small>
# Behind `trema run` ###########################################################

## `trema run` does lots of dirty tasks in its background:

	$ trema run hello-trema.rb -v
	.../trema/objects/switch_manager/switch_manager \
	  --daemonize --port=6633 -- port_status::HelloTrema \
	  packet_in::HelloTrema state_notify::HelloTrema \
	  vendor::HelloTrema
	Hello Trema!

* Spawns several daemons and processes required for controller execution
* Ctrl-c to quit them all
* Hides the complexity of Trema internal from users


<!SLIDE small transition=uncover>
# Run It Quick #################################################################

* Easily start and stop a controller process with one command
* Test your controller quickly right after coding
* Enables the tight cycle of "Coding, test, and debug"


<!SLIDE small transition=toss>
# Howto write controllers in Trema #############################################


<!SLIDE small transition=toss>
# hello-trema.rb ###############################################################

	@@@ ruby
	class HelloController < Controller
	  def start
	    info "Hello Trema!"
	  end
	end

* Simplest controller and it does almost nothing
* But it shows the basic code layout of controllers in Trema


<!SLIDE small>
# Controller Class #############################################################

	@@@ ruby
	class HelloController < Controller
	  # ...
	end

* All controllers are implemented as a class (`class HelloController`)
* Derived from Controller class
* Derives necessary methods from the class


<!SLIDE small>
# Event Handlers ###############################################################

	@@@ ruby
	class MyController < Controller
	  def start  # start-up event handler
	    # ...
	  end
	      
	  def packet_in dpid, msg  # Packet-in received handler
	    # ...
	  end
	
	  # ...
	end

* Controllers are Written in <i>event-driven</i> model
* Each handler is implemented as a instance method


<!SLIDE small>
# Event handlers in Floodlight #################################################

	@@@ java
	// Packet-in handling in Floodlight
	public Command receive(IOFSwitch sw, ...) {
	  switch (msg.getType()) {
	    case PACKET_IN:
	      return this.handlePacketIn(sw, ...);
	    ...
	private Command handlePacketIn(IOFSwitch sw, ...) {
	    ...

* Needs explicit handler dispatching
* Easily causes <i>code duplication</i> and is against <i>DRY principle</i> <br /> (Don't repeat yourself)


<!SLIDE small>
# Auto Dispatch ################################################################

	@@@ ruby
	# Packet-in handling in Trema
	class MyController < Controller
	  def start  # automatically called at startup
	    # ...
	  end
	      
	  def packet_in dpid, msg  # automatically caled when receiving a packet-in
	    # ...
	  end
	end

* If Trema finds a handler for incoming event, Trema calls the handler automatically
* You need no explicit handler dispatch or registration


<!SLIDE small transition=uncover>
# Don't Repeat Yourself ########################################################

## Philosophy of Trema: <i>Convention over Coding</i>

* E.g., "Event handler name" == "Event name"
* Kills boilerplate codes like event dispatching
* Reduces boring part of programming and make it interesting


<!SLIDE small transition=toss>
# Write It Short ################################################################

* There is a strong correlation between the length of code (number of tokens) and programmers' productivity
* e.g. Arc Programming Language [Paul Graham]
* With less typing, the faster you can read and write codes, less bugs

<br />

## Trema is specialized for <b>programmers' productivity</b>, <br /> not for execution efficiency


<!SLIDE small>
# Logging API ##################################################################

	@@@ ruby
	class HelloController < Controller
	  def start
	    info "Hello Trema!"	 # outputs an info level message
	  end
	end

* debug, info,... etc.
* You can check its API with `trema ruby` command


<!SLIDE>
# Hello Switch! ################################################################
## Connecting a OpenSwitch switch to controllers


<!SLIDE>
## "Wait, I don't have any OpenFlow switch. <br /> How to do that?" 


<!SLIDE small>
# Exercise: Hello Switch #######################################################

	$ trema run hello-switch.rb -c hello-switch.conf
	Password: gec13user  # Enter your password here
	Hello 0xabc!  # Ctrl-c to quit

* Connects a <b>virtual switch (dpid = 0xabc)</b> to a controller
* The controller outputs `"Hello 0xabc!"`
* Virtual switch definition is in `hello-switch.conf`


<!SLIDE small>
# hello-switch.conf ############################################################

	@@@ ruby
	# Add a switch with dpid == 0xabc
	vswitch { dpid "0xabc" }
	# or
	vswitch { datapath_id "0xabc" }

* Software switch launches and connects to a controller
* Trema is <b>full-stack</b>: You can develop with one note PC


<!SLIDE small>
# Behind `trema run` ###########################################################

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

## Yes, an Open vSwitch process is spawn


<!SLIDE small>
# `hello-switch.rb` ############################################################

	@@@ ruby
	class HelloSwitch < Controller
	  def switch_ready dpid
	    info "Hello #{ dpid.to_hex }!"
	  end
	end

* `switch_ready` is an handler called when a switch connects to the controller
* the argument `dpid` is switch's ID (in integer)
* `.to_hex` converts the integer into a String in hex format 


<!SLIDE small>
# Exercise: Adding Switches ####################################################

	@@@ ruby
	# hello-switch.conf
	vswitch { dpid "0x1" }
	vswitch { dpid "0x2" }
	vswitch { dpid "0x3" }
	  ...


	$ trema run hello-switch.rb -c hello-switch.conf
	???

* What `trema run` says if adding switches to `hello-switch.conf`?
* NOTE: dpid's of each switch must be unique


<!SLIDE small incremental transition=uncover>
# Summary So Far ###############################################################

## Trema is a "Post-Rails", modern development environment

<br />

* <i>Run It Quick</i>: `trema run`
* <i>Convention over Coding</i>: method naming convention
* <i>Full-Stack</i>: virtual network DSL
* Useful sub-commands: `trema ruby`
