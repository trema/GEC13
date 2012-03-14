<!SLIDE>
# Task A: Hello Trema! #########################################################


<!SLIDE commandline>
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


<!SLIDE small>
# Howto write controllers in Trema #############################################


<!SLIDE small>
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


<!SLIDE small>
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
