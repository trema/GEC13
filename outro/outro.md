<!SLIDE>
# Summary #######################################################################


<!SLIDE small incremental transition=uncover>
# Trema: "OpenFlow on Rails" ###################################################

* <i>Run It Quick</i>: Tight loops of coding, run, and debug
  * Virtual network DSL
  * `trema {run, send_packets, show_stats, up, kill}`
* <i>Convention Over Coding</i>: Write it short
  * Class attributes: `periodic_timer_event`
  * Syntactic sugars: `ExactMatch.from`
  * Default options: `send_flow_mod_add`
* <i>Useful Sub-commands</i>
  * `trema {dump_flows, ruby}`


<!SLIDE small>
# Next Step For Developers #####################################################

* `[trema]/src/examples`
  * Simple samples demonstrating API usage
  * Good for reference for both Ruby and C
* Trema/Apps <http://github.com/trema/apps>
  * Real-world Trema applications (EXPERIMENTAL)
  * learning switch with memcached
  * routing switch
  * sliceable routing switch etc.


<!SLIDE small>
# Sliceable routing switch #####################################################

* Layer-2 network virtualization
  * Virtual flat L2 network domains + L1-4 access control list
* REST-API for slice management
  * Create slices and attach hosts by its port or MAC


<!SLIDE>
# Love Ruby? ###################################################################


<!SLIDE small>
# Trema C ######################################################################

* It's your choice ... Trema provides both Ruby and C library.
* Trema C is also simple as Trema Ruby

<br />

	$ gcc mycontroller.c `trema-config -c -l` -o my-controller
	$ trema run my-controller


<!SLIDE small>
# Sources ######################################################################

* Trema: <http://github.com/trema/>
* This Tutorial: <http://github.com/trema/GEC13/>
* Web Page: <http://trema.github.com/trema/>
* Twitter: <http://twitter.com/trema_news>
* Mailing List: <https://groups.google.com/group/trema-dev>
* Bugs: <https://github.com/trema/trema/issues>


<!SLIDE small>
# Contributors! ################################################################

* Free Trema T-shirt: Send pull-requests and get the T!


<!SLIDE>
# Questions? ###################################################################


