---
layout: post
title:  "Building a chicken coop dusk timer with NodeMCU and Lua"
date:   2019-09-24 01:02:03
categories: programming iot nodemcu lua
---
One of the more fun bits of hardware I have acquired in the last
couple of years is the [NodeMCU](https://www.nodemcu.com/index_en.html)
development board.

![A NodeMCU development board](https://upload.wikimedia.org/wikipedia/commons/thumb/7/7e/NodeMCU_DEVKIT_1.0.jpg/640px-NodeMCU_DEVKIT_1.0.jpg)

This little board contains the [Espressif ESP8266 WiFi-enabled
microcontroller](https://www.espressif.com/en/products/socs/esp8266/overview)
which packs a punch: a 32bit RISC processor clocked at up to 160MHz,
on board WiFi, and the NodeMCU board adds power management, 4MB
flash, and some odds and ends without increasing the footprint much
beyond a small Arduino. The best thing is that your local store
will have it for around $5 and if you source if from China for
around a dollar and change. I'm a big fan of Elixir and especially
[Nerves](https://www.nerves-project.org/), but I'm also Dutch and
therefore a cheapskate - I'll use a cheaper part if I can and the ESP8266
displaces a lot of stuff where years ago, you would only have considered
a Raspberry Pi.

NodeMCU is a development board and an SDK, and the SDK is in one of my
favorite little languages, [Lua](https://www.lua.org/). Lua was made
for embedding, it is hugely popular in a lot of industries, and rightfully
so - the whole official language and run-time environment weigh in at ~20k
lines of C and ~10k lines of Lua code, there are multiple highly compatible
implementations to choose from (including [Luerl](https://luerl.org/) which
implements Lua from scratch in Erlang), and given its size, it is not hard
to run it on really small machines, like the ESP8266. NodeMCU also comes
with [excellent documentation](https://nodemcu.readthedocs.io/en/release/)
and a library containing all the things you will need for your embedded
project.

So why am I writing about this? We have chickens. That's why.

Chickens are awesome creatures. They hatch and then they come preprogrammed
with everything they need to know to survive - we know, because we got ours
as day-olds and we certainly did not teach them how to swallow a young frog
whole. We let ours free roam and being excitable and curious creatures they
often forget to check back into the coop in time. So they enter a dark coop
around dusk, cannot see the roost, and then sleep somewhere else with the
risk of being pooped on by the ones that did make it back in time. We see
that on the coop cam (a Wyze 2, never owned a very small cube that runs Linux
and talks to me before), get up, and rectify the situation manually.

Therefore, what's needed is some extra light around dusk.

The simplest definition of "dusk" is "when it gets darker", and I started
hooking up a photoresistor to the dev board's A/D converter, which worked
fine (see, I'm an electronics whizz!) but then I thought about how to
calculate, with certainty, that it gets darker because of dusk and not because
of a thunderstorm. Yes, I could have just pivoted and turned the "dusk light"
into a "it is getting somewhat dark for any ol' reason" light, but I was not
ready to do that yet. Therefore, I needed a way to know what time it was,
probably. I said that NodeMCU comes with a rich standard library so I was
very happy to find the [`sntp`](https://nodemcu.readthedocs.io/en/release/modules/sntp/) module, a simple NTP client.

Hmm... so if I know, with NTP precision, what time it is can't I just _calculate_
dusk instead of relying on a photoresistor? Less hardware, more software, just
up my alley. Not a lot of stuff from Lua land came up, but Wikipedia has an
entry on the [Sunrise Equation](https://en.wikipedia.org/wiki/Sunrise_equation)
that felt like something I could translate into Lua. So, I started out
with a test:

```lua
function test_sunrise_sunset()
   -- Toronto
   local lat = 43.740
   local lng = -79.370

   -- Somewhere on 1 Nov 2020
   -- According to https://nrc.canada.ca/en/research-development/products-services/software-applications/sun-calculator/:
   -- Sun rise: 6:54 LT / 11:54 UTC
   -- Solar noon: 12:01 LT / 17:01 UTC
   -- Sun set: 17:07 LT / 22:07 UTC
   local jd = posix2julian(1604250457)
   local rise, noon, set = sunrise_sunset(lat, lng, jd)
   assert_equal("2459154.9966005", ""..rise)
   assert_equal("2459155.2098313", ""..noon)
   assert_equal("2459155.4230621", ""..set)
```

Note the string compares - pure laziness, because `lunit` does not have
something to compare floats with a delta and I was not going to let me
be sidetracked in finding something else/better or even writing something
from scratch. The code was a relatively simple transcription from Wikipedia,
with some thinking over where to put degree-to-radian conversions as Wikipedia
worked in degrees but most math libraries want, for good reasons, radians. Without
further ado, I landed on this:

```lua
local sunrise_sunset = function(lat, lng, jd)
   local julian_date = math.floor(jd) -- note, by the way, that ".0" equals noon,
                                      -- not midnight, as the Julian date numbering
                                      -- started at 1200 somewhere in 4713 BC

   local n = julian_date - 2451545.0 + 0.0008 -- julian day
   local j_star = n - (lng / 360.0) -- mean solar noon
   local m = math.rad((357.5291 + 0.98560028 * j_star) % 360) -- solar mean anomaly
   local c = 1.9148 * math.sin(m) + 0.0200 * math.sin(2 * m) + 0.0003 * math.sin(3 * m) -- equation of the center
   local lambda = math.rad((math.deg(m) + c + 180 + 102.9372) % 360) -- ecliptic longitude
   local j_transit = 2451545.0 + j_star + 0.0053 * math.sin(m) - 0.0069 * math.sin(2 * lambda)
   local earth_tilt = math.rad(23.44)
   local delta = math.asin(math.sin(lambda) * math.sin(earth_tilt)) -- declination of the sun
   local correction_angle = math.rad(-0.83)
   local lat_rad = math.rad(lat)
   local omega_0_divident = math.sin(correction_angle) - math.sin(lat_rad) * math.sin(delta)
   local omega_0_divisor = math.cos(lat_rad) * math.cos(delta)
   local omega_0 = math.acos(omega_0_divident / omega_0_divisor) -- hour angle
   omega_0 = math.deg(omega_0)

   local j_rise = j_transit - (omega_0 / 360.0)
   local j_set = j_transit + (omega_0 / 360.0)

   return j_rise, j_transit, j_set
end
```

Note that this works with [Julian dates](https://en.wikipedia.org/wiki/Julian_day) which I think is a concept that deserves more attention in the software world. Given that both the POSIX timestamp and the Julian date are offsets from a given epoch, the conversion is trivial:

```lua
local posix2julian = function(posix_timestamp)
   return (posix_timestamp / 86400.0) + 2440587.5
end
```

Where the `2440587.5` represents the difference between 1970 and 4713BC. One odd
thing happened with my code which was a headscratcher until I realized that
the epoch starts at noon (UTC) and not midnight, and that cost me a lot of time.
I could not understand why the outcomes where half a day off :).

I wrote the code and tests on my Linux box, but the same code worked on the
ESP8266 once I rebuilt the firmware with two options:
* Use the included Lua 5.3 instead of the default Lua 5.1; the 5.3 version
  includes the `math` standard libraries;
* Realize just how bad single precision flaoting point works for this sort
  of stuff and switch Lua to double precision.
Once that was done, everything worked fine. I now had a microcontroller that
could, theoretically, ask the Net for the exact date and calculate sunset from
there. My first computer was a TRS-80 so yeah, mind blown.

NodeMCU's "Node" comes from its borrowing Node's concurrency model, heavily
relying on callbacks. I really do not like that programming model, because
you invariably end up mixing up concurrency and business level code, but for
this it worked just perfectly. We boot, some housekeeping is done (like
attaching to WiFi), and then we ask what time it is:

```lua
function ntp_sync()
   sntp.sync(nil, ntp_callback, ntp_err_callback, nil)
end

gpios_off()
ntp_sync()
```

`gpios_off` switches off the LEDs (we'll get to the hardware later). The
`sntp` module has a `sync` command that does two things: it sets the
time in the ESP8266's RTC (so that subsequent calls to the `rtctime`
module give the right answer) and it calls a callback function with
some info on the NTP response (for now, I'm leaving the error callback
out, but it just reboots the MCU). The callback then calculates
sunset, calculates the time that the lamp should be on (I put that
half an hour before sunset), and depending on whether we're in the
lamp on interval or not, does the right thing. The whole thing is [in
Github](https://github.com/cdegroot/cooplight/blob/main/src/application.lua)
so I'm not going to copy/paste, but for example if the lamp needs to be
switched on, then this gets called:

```lua
function turn_lamp_on(time_to_lamp_off)
   log.info("Turning lamp on for "..time_to_lamp_off.." seconds")
   gpios_on()
   if not tmr.create():alarm(time_to_lamp_off * 1000, tmr.ALARM_SINGLE, function()
                                log.info("Lamp has been on enough, turning off")
                                gpios_off()
                                sleep_until_next_round()
                            end)
   then
      log.error("ERROR Could not create timer until lamp off, rebooting.")
      node.restart()
   end
end
```

Like most functions there, it ends up doing something and then setting up a timer
with a callback to do the next thing. As far as state machines with delays go,
I found it a very clean and simple to reason about way to write code, and it
was pretty much one of the first times that the callback model for multithreading
looked like it found home. The state machine breaks down in four functions that
do something and then call or schedule each other.

You would expect me to write something about testing now. I should. I won't. But
it'd be extremely trivial in Lua to have dummy implementations of `tmr` and
`sntp` and friends. Instead, I decided that there are just a couple of use
cases, it was around dusk anyway, so I just rebooted the MCU at set times to
see whether it made the right decisions. It did :). Time for hardware.

![img/cooplight-breadbord.jpg](Cooplight on a breadboard)

Breadboard first, of course. While the target hardware uses left over LED strips,
these take 12V so I verified that things worked using regular LEDs. The transistors
(2N3904 for the very good reason that I had them) are triggered through two
GPIO pins with a 1KΩ resistor as a current limited between GPIO and base,
and the LEDs are between 5V and the transistors' collectors. The emitters
are wired to ground. Needless to say, this is 101 electronics, if it even
makes that bar and everything just worked. Still, had to verify with the
blinkenlights, not?

Why two transistors, I hear you ask. Well, I only had low power
transistors and the end result would use a 12V LED strip that was
left over; it has six LEDs on it and on measuring it on my bench power
supply, it drew 50mA. 50mA times 13.6V is 680mW which is higher than the
transistor's 625mW max power dissipation. Instead of calculating and
measuring more to see how much the transistor would actually dissipate,
I decided to use the LED strip's hint and cut it in half (these strips
have a cut mark every three LEDs) and just trigger two GPIOs. Shorter
strips are simpler to work with anyway. For those that, like me, are
not electronics wizzards, the schematic:

![img/cooplight-circuit.svg](Cooplight circuit)

and, for a good laugh, the end result, mounted dustproof in an old
pickle jar:

![img/cooplight-in-a-jar.jpg](Cooplight in a jar)

The jar is dustproof, fireproof, and transparent so it is the perfect
enclosure for a chicken coop. The jar is fed by 12V from a solar+battery
combination, and I added a DC-DC voltage regulator; I once bought a handful
of these based on an LM259 and they'll eat anything up to 40V and spit out
whatever you need. The LEDs get the 12V directly and the NodeMCU 5V through
its USB port (I could have soldered the 5V on the NodeMCU but this way the
device either gets 5V or is connected to my laptop, not both at the same time
which I think is safer). Next up is install in the coop, as soon as the snow
stops :)

Why am I writing this? Mostly to get your attention for this
wonderful and cheap IoT miracle. The whole build took me maybe two
days, and for much less than a Raspbery Pi I now have something that
does the trick, is network connected, easy to upgrade (although not
yet over-the-air), and will tell me what it is doing through syslog
to a PC that is always on and has remote syslogging enabled. Using
[ESPlorer](https://github.com/4refr0nt/ESPlorer) development is highly
interactive and Lua seems to me like a language tailor-made for this
kind of device.

Also, I hope you had fun reading this, of course :)
