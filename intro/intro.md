<!SLIDE title-slide>
# Trema Tutorial ###############################################################

### Yasuhito Takamiya  `@yasuhito`
### Yasunobu Chiba  `@chibacchie`
### Hideyuki Shimonishi  `@hide_shimonishi`


<!SLIDE small>
# Today's Goal #################################################################

* Implement "L2 switch with traffic monitoring" controller
* This tutorial is composed of small four iterations starting from "Hello World"
* Let's go through the entire cycle of OpenFlow controller development using Trema


<!SLIDE small incremental>
# What's Trema? ################################################################

* New OpenFlow programming framework in Ruby and C
  * GPL2
  * <http://github.com/trema/trema>
* Designed to be highly productive for this "Post-Rails" era
  * <i>Run It Quick</i>: Tight loop of coding, run, and debug
  * <i>Convention over Coding</i>: Write it short
  * <i>Integrated Unit-Testing</i>


<!SLIDE>
# Trema =
## `trema` command
## +
## Ruby and C library
## +
## Network Emulator
