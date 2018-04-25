Closed loop code - this folder contains the Labview program that receives events from OpenEphys and times stimulation pulses. Within ClosedLoopStim.lvproj, the main VI is main.vi. The basic idea is:

* Keep track of the current OpenEphys acquisition board sample number (via the BNC connector on the back) in a DAQ onboard counter.
* Receive TTL events from OpenEphys on a specified channel (via ZeroMQ), which indicate when we want to fire a pulse. This includes the sample number of when the crossing ocurred in the metadata.
Note: as of August 2017, when the VI is started, it sends an event to the board through the digital channel specified in "OEP timestamp probe port" and the OEP channel in "probe OEP channel," and simulaneously queries the DAQ's counter. The difference is saved as the offset of the GUI's timestamps from the counter and this is subtracted from the timestamps of all subsequent received events.
* Calculate the difference between the current sample number and the one with the event (the "current OEP to LV delay"), and keep track of the maximum such delay.
* For each event, wait until the current sample number minus the event sample number equals that maximum delay (plus an optional extra delay), and then fire a pulse through the DAQ (as well as a digital signal). 

The waiting step allows us to equalize the recording-to-pulse delay even when the information about the event takes a varying amount of time to reach LabView. The tradeoff is that this delay is longer on average than it would otherwise be (but future improvements in FPGA-to-PC transfer rate, processing time, etc. would improve this).

Setup:
1. Connect the FPGA clock out signal (rear BNC) to a DAQ clock input (probably PFI 0, which has a BNC connector). In the front panel, select this input under "OEP sample clock in."
2. Select any DAQ counter under "Onboard counter" (e.g. [DAQ name]/ctr0)
3. Connect a digital I/O port on the DAQ to an OEP digital in channel (via an I/O board) - this is for the timestamp probe event. Select the digital port in "OEP timestamp probe port" and the OEP channel in "probe OEP channel". Be sure this channel is different from the crossing detector channel.
3. On the left of the back panel, double-click on the PrepareChannels_v2 VI to configure its front panel.
  a. Connect another digital I/O port on the DAQ, via an OpenEphys I/O board, to the OpenEphys digital in HDMI port - this is for reporting when a pulse is fired. Be sure the channel it is connected to on the I/O board is different from both the crossing detector and timestamp probe channels. Select the digital port in the PrepareChannels_v2 front panel under "DO Physical channels."
  b. Connect an analog output on the DAQ to either the OpenEphys analog in port via an I/O board (if doing testing) or to a stimulation box (if doing actual stimulation). Select the analog port in the PrepareChannels_v2 front panel under "AO Physical channels." 
  c. Be sure all the other settings in the PrepareChannels_v2 VI are correct. In particular, if doing testing, increase the pulse width to about 0.9 ms as this improves pulse detection when analyzing in Matlab.
4. Set "sample rate" to equal the actual sampling rate of the FPGA, in Hz.
5. Set "port" to match the port setting on the Event Broadcaster.
6. Set "event channel" to the stimulation timing event channel.
7. If you want to get an accurate phase offset estimate, set "Avg signal freq" to the average frequency of the signal we are calculating the phase of, e.g. ~6 for theta band.
8. Make sure "Delay lock" is off when starting the VI.

Running:
-Start running the LabView program before starting acquisition in OpenEphys for the DAQ's counter to accurately hold the OpenEphys sample number.
-Watch the graph of OEP to LV delay - it will probably be very irregular and cycle up and down over a period of a few minutes. (Update in summer 2017: it no longer seems to have this cycling problem, but it still has a high variance.) Wait until it looks like this delay has approximately reached a global maximum. We want pulses to be delayed enough so that the delay is consistent, i.e. it never takes too long for the event to arrive for the pulse to occur on schedule.
--As of 2018, you can also turn on "limit delay" and specify the maximum delay in order to limit the running maximum (potentially allowing some outlier events, which will be skipped for stimulation). This limit includes the extra delay described below; the "max observed allowable delay" does not.
-At this point, you can add an extra delay to be safe - then the total OEP-to-pulse delay will be the max delay plus the extra delay (in samples).
-If the average signal frequency is accurate, the "Phase offset" field will show the number of degrees back to set the crossing detector from the target phase in order to stimulate on the target phase.
-Press delay lock. Then you can start recording.

Miscellaneous tips:
* Make sure to only process the number of channels you are using in the Phase Calculator! This can have a large performance impact.
* Changing the "Audio device type" and buffer size in the OpenEphys GUI can make a difference in performance and accuracy (smaller buffer size is better, although there is a processing power tradeoff). To access these settings, click on the text that says "LATENCY: xx MS" to the right of the "Gate" slider in the GUI.

Dependencies:

--ClosedLoopStim.lvproj----------
|                               |
|   main.vi                     |
|   |                           |
|   |- PrepareChannels_v2.vi    |
|   |  |                        |
|   |  |- BuildStimulusPulse.vi |
|   |                           |
|   |- TTLConnect.vi            |
|   |  |                        |
|   |  |- EventType.ctl         |
|   |                           |
|   |- TTLReceive.vi            |
|   |  |                        |
|   |  |- EventType.ctl         |
|   |                           |
|   |- TTLDisconnect.vi         |
|   |                           |
|   |- ReceiveCrossingEvent.vi  |
|      |                        |
|      |- TTLReceive.vi [...]   |
|                               |
---------------------------------

Also requires the LabVIEW ZeroMQ and DAQmx libraries.

-Ethan, Sep. 8, 2016
Updated Apr. 25, 2018