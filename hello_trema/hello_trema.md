<!SLIDE>
# Task A: Hello Trema! #########################################################


<!SLIDE commandline>
## Exercise: Run "Hello Trema!" ################################################

### Run Trema by typing the following commands:

	$ cd Tutorials/Trema
	$ trema run hello-trema.rb
	Hello Trema!   # Ctrl-C to quit


<!SLIDE small>
# One Basic Command: `trema run` ###############################################

	$ trema run [controller-file]

* Starts a controller process
* Ctrl-c to quit
* Full options list available by `trema help run`


<!SLIDE small>
# Run It Quick #################################################################

* Faster execution/termination of controller process with one `trema run` command
* Highly tailored testing of your controller immediately after coding
* Enables the tight cycle of "Coding, test, and debug"


<!SLIDE small>
# How-to write controllers in Trema #############################################


<!SLIDE small>
# hello-trema.rb ###############################################################

	@@@ ruby
	class HelloController < Controller
	  def start
	    info "Hello Trema!"
	  end
	end

* Simple and complete but not so much useful or interesting
* But it demonstrates how to craft controllers in Trema


<!SLIDE small>
# Controller Class #############################################################

	@@@ ruby
	class HelloController < Controller
	  # ...
	end

* All controllers implemented as class objects (`class HelloController`)
* Subclassed from `Controller` class defined in Trema class library
* All controller's supported methods (flow-mod message creation etc.) injected automatically into your class


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

* Controllers follow the <i>event-driven</i> paradigm
* Each event handler implemented as a instance method


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

* Trema uses retrospection to dispatch events to registered handlers
* No need to explicitly dispatch or register handlers just define it


<!SLIDE small transition=uncover>
# Convention over Coding #######################################################

* Coding conventions for concise and compact code
  * e.g., "handler name" == "message name"
* Eliminates boilerplate code like event dispatching
* Reduces the tedious and boring parts of programming hence fun to program


<!SLIDE small>
# Write It Short ################################################################

* There is a strong correlation between the length of code (number of tokens) and programmers' productivity
  * e.g. Arc Programming Language [Paul Graham]
* With smaller code
  * less time to write consistant code
  * less chances for bugs

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

* debug, info and other logging levels in descreasing verbosity
* Browse through the Logging API and rest of API by invoking `trema ruby`
