<!SLIDE small>
# Task B: Switch Monitor #######################################################

## Monitoring switch liveness


<!SLIDE small>
# Excercise: Run Switch Monitor ################################################

	$ trema run switch-monitor.rb -c switch-monitor.conf
	Switch 0x3 is UP
	Switch 0x2 is UP
	Switch 0x1 is UP
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	All switches = 0x1, 0x2, 0x3
	  ...

* Detects three switches as defined in `.conf`
* Updates liveness information every 10 seconds


<!SLIDE small>
# Exercise: Killing Switches ###################################################

	$ trema kill 0x1  # Kill switch 0x1
	$ trema up 0x1    # Start switch 0x1

* On another terminal execute `trema kill` and `trema up`
* Observe how the output of `trema run` looks like?


<!SLIDE small>
# Handling Switch Disconnection ################################################


<!SLIDE small>
# `SwitchMonitor#switch_disconnected` ##########################################

	@@@ ruby
	class SwitchMonitor < Controller
	  # ...	
	
	  def switch_disconnected dpid
	    @switches -= [ dpid.to_hex ]
	    info "Switch #{ dpid.to_hex } is DOWN"
	  end

	  # ...	
	end

* Almost identical code used in other handlers
* `@switches` is explained later


<!SLIDE smaller>
# Managing Switch List #########################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  # ...
	  def start
	    @switches = []
	  end
	
	  def switch_ready dpid
	    @switches << dpid.to_hex
	    info "Switch #{ dpid.to_hex } is UP"
	  end
	
	  def switch_disconnected dpid
	    @switches -= [ dpid.to_hex ]
	    info "Switch #{ dpid.to_hex } is DOWN"
	  end
	  # ...
	end

* `@switches` is an instance variable that holds a list of switches that are alive
* `[]` is an array in Ruby
* "`<< element`" to add an element, "`-= [ element ]`" to remove from the array


<!SLIDE smaller>
# Listing Switches #############################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  # ...
	      
	  private  # following methods are private
	  def show_switches
	    info "All switches = " + @switches.sort.join( ", " )
	  end
	end

* `periodic_timer_event`: calls `show_switches` method every 10 sec.
* `show_switches`: outputs the sorted switches array


<!SLIDE smaller>
# Timer Attribute ##############################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  # ...
	end

* Define timer handlers using a class attribute
* Don't need to implement timer handling by yourself using threads etc.
* Another example of <i>convention over coding</i>


<!SLIDE smaller>
# Final Code ###################################################################

## Clean complete and full-functional with no redundancy
## Easy to read and reuse

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  def start
	    @switches = []
	  end
	
	  def switch_ready dpid
	    @switches << dpid.to_hex
	    info "Switch #{ dpid.to_hex } is UP"
	  end
	
	  def switch_disconnected dpid
	    @switches -= [ dpid.to_hex ]
	    info "Switch #{ dpid.to_hex } is DOWN"
	  end
	
	  private
	  def show_switches
	    info "All switches = " + @switches.sort.join( ", " )
	  end
	end
