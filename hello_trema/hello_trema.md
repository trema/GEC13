<!SLIDE transition=toss>
# Task A: Hello Trema! #########################################################


<!SLIDE commandline transition=toss>
## Exercise: Run "Hello Trema!" ################################################

### Run Trema by typing the following command sequence:

	$ cd Tutorials/Trema
	$ trema run hello-trema.rb
	Hello Trema!   # Ctrl-C to quit


<!SLIDE small>
# One Basic Command: `trema run` ###############################################

	$ trema run [controller-file]

* Starts a controller process
* Ctrl-c to quit
* Full options list is available by `trema help run`


<!SLIDE small transition=uncover>
# Run It Quick #################################################################

* Enables the quick start/stop of controller process with one `trema run` command
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

* Simple and complete but not so much useful or interesting
* But it shows the basic code layout of controllers in Trema


<!SLIDE small>
# Controller Class #############################################################

	@@@ ruby
	class HelloController < Controller
	  # ...
	end

* All controllers are implemented as a class (`class HelloController`)
* Derived from `Controller` class defined in Trema class library
* All necessary methods for controller (flow-mod message creation etc.) are injected automatically into your class


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

* Floodlight API requires explicit handler dispatching
* Lots of boilerplate code, this easily leads to <i>code duplication</i>


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
# Convention over Coding #######################################################

* Coding conventions for concise and compact code
  * e.g., "handler name" == "message name"
* Kills boilerplate codes like event dispatching
* Reduces boring part of programming and make it fun


<!SLIDE small transition=toss>
# Write It Short ################################################################

* There is a strong correlation between the length of code (number of tokens) and programmers' productivity
  * e.g. Arc Programming Language [Paul Graham]
* With smaller code,
  * the faster you can read and write codes,
  * the less chances for bugs

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

* debug, info, warn... etc.
* You can check the API with `trema ruby` command


<!SLIDE>
# Hello Switch! ################################################################
## Connecting a OpenFlow switch to your controller


<!SLIDE>
## "Wait, I don't have any OpenFlow switch. <br /> How to do that?" 


<!SLIDE small>
# Exercise: Hello Switch Controller ############################################

	$ trema run hello-switch.rb -c hello-switch.conf
	Password: gec13user  # Enter your password here
	Hello 0xabc!  # Ctrl-c to quit

* Connects a <b>virtual switch (dpid = 0xabc)</b> to the controller
* The controller outputs `"Hello 0xabc!"`
* Virtual switch definition is in `hello-switch.conf`


<!SLIDE small>
# hello-switch.conf ############################################################

	@@@ ruby
	# Add a switch with dpid == 0xabc
	vswitch { dpid "0xabc" }
	# or
	vswitch { datapath_id "0xabc" }

* This launches a software switch and make it connect to the controller
* Trema is <b>full-stack</b>: You can develop with your laptop, no need for physical switches!


<!SLIDE small>
# The Back Side of `trema run` #################################################

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

## Certainly an Open vSwitch process is spawn defined in the .conf file


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

* What `trema run` says if adding some switches to `hello-switch.conf`?
* NOTE: dpid's of each switch must be unique


<!SLIDE small incremental transition=uncover>
# Summary So Far ###############################################################

## Trema is a "Post-Rails", modern development environment

<br />

* <i>Run It Quick</i>: `trema run`
* <i>Convention Over Coding</i>: method naming convention
* <i>Full-Stack</i>: virtual network DSL
* Useful sub-commands: `trema ruby`
