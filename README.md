# check-common-ups
this file: README.md
snmp checks for common UPS with perfdata (currently APC or CyberPower)
auto detect UPS brand, and run snmp checks for icinga2. Should also work for nagios and icinga1

THE PROGRAM IS DISTRIBUTED IN THE HOPE THAT IT WILL BE USEFUL, BUT WITHOUT
ANY WARRANTY. IT IS PROVIDED "AS IS" WITHOUT WARRANTY OF ANY KIND, EITHER
EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES 
OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE. THE ENTIRE RISK AS 
TO THE QUALITY AND PERFORMANCE OF THE PROGRAM IS WITH YOU. SHOULD THE 
PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF ALL NECESSARY SERVICING, 
REPAIR OR CORRECTION. IN NO EVENT WILL THE AUTHOR BE LIABLE TO YOU FOR 
DAMAGES, INCLUDING ANY GENERAL, SPECIAL, INCIDENTAL OR CONSEQUENTIAL DAMAGES
ARISING OUT OF THE USE OR INABILITY TO USE THE PROGRAM (INCLUDING BUT NOT 
LIMITED TO LOSS OF DATA OR DATA BEING RENDERED INACCURATE OR LOSSES SUSTAINED 
BY YOU OR THIRD PARTIES OR A FAILURE OF THE PROGRAM TO OPERATE WITH ANY OTHER 
PROGRAMS), EVEN IF THE AUTHOR HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH 
DAMAGES.

The 2 files needed are:
/usr/share/icinga2/include/plugins-contrib.d/common_ups.conf
/usr/lib64/nagios/plugins/check_common_ups

With graphite, the default settings will result in 6 graphs per UPS, all on one page:
  Temp - battery temperature
  Line_Voltage - Input voltage to the UPS
  Load - power load reported 
  RunTime - battery runtime in minutes
  Diagnostic - days since last diagnostic
  Calibration - days since last calibration
The last 2, Diagnostic and Calibration days, can be disabled.

If you have a cyberpower 202 or 303 card or another card that does not
support battery status checks, you need to disable the battery
replace status check with ups_noreplaceck.

You can also disble Diagnostic and Calibration tests, but I have found
this causes some issues to be missed, like "wiring fault".

A sample config /etc/icinga2/conf.d/my-ups.conf:
```
object Host "my-ups" {
import "generic-host"
display_name = "my-ups"
address = "10.1.2.3"
vars.ups_address = "10.1.2.3"
vars.ups_community = "readonly"
groups = [ "mygroup","ups" ]
}
```
continued in /etc/icinga2/conf.d/groups.conf:
```ini
object HostGroup "ups" {
  display_name = "UPS"
}

object HostGroup "mygroup" {
  display_name = "This is me"
}
```
continued in /etc/icinga2/conf.d/services.conf:
```
apply Service "ups_stats" {
  import "generic-service"
  check_command = "check_common_ups"
  assign where "ups" in host.groups
}
```
# A sample ups that changes a lot of the defaults:
```
object Host "sample-ups" {
import "generic-host"

display_name = "sample-ups"
address = "10.1.2.3"
vars.ups_address = "10.1.2.3"
vars.ups_community = "readonly"  
vars.ups_noreplaceck = "true"    /* disable battery replace status check cyber 202 & 303 cards */
vars.ups_degrees = "c"           /* celsius (default: fahrenheit) */
vars.ups_warntemp = "40"         /* warning when temp is above 40c (default: 45c or 113f)
                                    note: this is in celsius because ups_degrees is set to "c"
                                          if ups_degrees is f, enter degrees in fahrenheit */
vars.ups_crittemp = "45"         /* critical when temp is above 45c (default: 50c or 122f) */
vars.ups_onbattiscrit = "y"      /* critical when UPS is on battery (default: warning) */
vars.ups_inlow = "105"           /* warning when input voltage drops below 105 (default: 106)(see ups_voltcrit) */
vars.ups_inhigh = "130"          /* warning when input voltage is above 130 (default: 133)(see ups_voltcrit) */
vars.ups_voltcrit = "y"          /* above input voltage warnings will be critical instead of warning */
vars.ups_warnload = "50"         /* warning when load is above 50 (default: 70) */
vars.ups_critload = "75"         /* critical when load is above 75 (default: 80) */
vars.ups_warnrun = "15"          /* warning when runtime is below 15 minutes (default: 10) */
vars.ups_critrun = "10"          /* critical when runtime is below 10 minutes (default: 8) */
vars.ups_diagwarn = "8"          /* warning when days since last diagnostic test is > 8 (default: 15) */
vars.ups_diagcrit = "9"          /* critical when days since last diagnostic test is > 9 (defaut: 20) */
vars.ups_calibwarn = "180"       /* warning when days since last calibration > 180 (default: 365) */
vars.ups_calibcrit = "195"       /* critical when days since last calibration > 195 (default: 400) */
vars.ups_nodiagdata = "y"        /* disable diagnostic graph data */
vars.ups_nocalibdata = "y"       /* disable calibration graph data */

groups = [ "mygroup","ups" ]
}
```
