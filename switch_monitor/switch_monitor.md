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

* Detects three switches defined in `.conf`
* Updates switch list in every 10 seconds


<!SLIDE small>
# Exercise: Killing Switches ###################################################

	$ trema kill 0x1  # Kill switch 0x1
	$ trema up 0x1    # Start switch 0x1

* Open another terminal and execute `trema kill` and `trema up`
* How the output of `trema run` look like?


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

* Almost same with other handlers
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

* `@switches` is an instance variable that holds a list of switches alive
* `[]` is a list in Ruby
* "`<< element`" to add an element, "`-= [ element ]`" to remove from the list


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

* `periodic_timer_event`: call `show_switches` method in every 10 sec.
* `show_switches`: outputs a sorted switch list


<!SLIDE smaller>
# Timer Attribute ##############################################################

	@@@ ruby
	class SwitchMonitor < Controller
	  periodic_timer_event :show_switches, 10
	
	  # ...
	end

* You can define timer handlers like a class attribute
* Don't need to implement timer handling by yourself using threads etc.
* Another example of <i>convention over coding</i>


<!SLIDE smaller>
# Final Code ###################################################################

## Clean, not redundant, only necessary elements included.
## Easy to read and write.

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
