# cl_stim

This folder contains the Labview program that receives events from OpenEphys and times stimulation pulses. The main file is CLStimSimple.vi.

This version outputs pulses on 2 analog output channels. One of these ("real") is actually connected to the stimulator, and the other ("sham") is only connected to the Open Ephys analog in. The program receives TTL events sent via ZeroMQ from the phase crossing detector in the closed loop signal chain in Open Ephys, and this triggers real and sham stimulations according to the timeout and stimulation pattern preferences.

## Setup

1. Connect the Open Ephys clock out signal (rear BNC) to a DAQ clock input (probably PFI 0, which has a BNC connector). In the front panel, select this input under "OEP sample clock in."

2. Select any DAQ counter under "Onboard counter" (e.g. [DAQ name]/ctr0)

3. Connect an I/O board to the Open Ephys analog in HDMI port. In the signal chain, the Rhythm FPGA source should have "ADC 1-8" enabled, and there should be 2 Crossing Detectors immediately after it. These will create the TTL events corresponding to real and sham stimulations.

4. Connect one analog output DAQ channel via a BNC splitter to both the stimulator and one BNC connector of the analog in I/O board. Select this analog output under "Real stim channel." In the first Crossing Detector in Open Ephys, select the analog input channel (keeping in mind that the ADC channels are the last 8 inputs) as the input, and "2" as the output.

5. Connect a second analog output DAQ channel to another BNC connector of the analog in I/O board. Select this analog output under "Sham stim channel." "Sham TTL chan" should be 3. In the second Crossing Detector in Open Ephys, select the anlog input channel as the input and "3" as the output.

6. Set "sample rate" to equal the actual sampling rate of the FPGA, in Hz (e.g., 30000).

7. Set "event port" to match the port setting on the Event Broadcaster (e.g., 5557). The format on the Event Broadcaster should be "Raw binary"

8. Set "Crossing TTL chan" to the output channel of the Crossing Detector for the Phase Calculator output - typically 1.

9. Configure timeout and stim pattern settings, if desired:

    * "Artifact padding" - how many ms before and after each real stim to not do a real or sham stim.
    
    * "Max add'l timeout" - for each timeout period, a random number of ms between 0 and this number will be added to the artifact padding. This is intended to make stimulation times less regular and avoid phase locking at a low frequency.
    
    * Stim pattern - when "Stim enabled" is selected, the output will be the selected number of real stims, followed by the selected number of sham stims, then back to real stims, etc. For example, for the default pattern of 1 real and 1 sham, they will simply alternate.
        * (when "Stim enabled" is off, only sham stims will be output.)

## Running

* Start running the LabView program before starting acquisition in OpenEphys for the DAQ's counter to accurately hold the OpenEphys sample number and produce a correct delay plot.
* When you are ready to stimulate, turn on "Stim enabled."

## Tips

* Make sure to only process the number of channels you are using in the Phase Calculator! This can have a large performance impact.

* Makek sure to follow the instructions in blue to select "ASIO" as the audio processor and the correct sample rate and buffer length. The button to open the audio settings is to the left of the play/record buttons in Open Ephys.

## Dependencies

```
 CLStimSimple.vi                     
 |
 |- SetupCLStim_2Analog.vi
 |  |
 |  |- TTLConnect.vi [...]
 |  |
 |  |- TTLReceive.vi [...]
 |  |
 |  |- PrepareChannels_2Analog.vi    
 |     |                        
 |     |- BuildStimulusPulse.vi 
 |                           
 |- TTLConnect.vi            
 |  |                        
 |  |- EventBaseType.ctl         
 |                           
 |- TTLReceive.vi            
 |  |                        
 |  |- EventBaseType.ctl         
 |
 |- UpdatePulseType.vi
 | 
 |- TTLDisconnect.vi         
 |                           
 |- ReceiveCrossingEvent.vi  
    |                        
    |- TTLReceive.vi [...]   
```

Also requires the LabVIEW ZeroMQ and DAQmx libraries.

-Ethan B, 2019-07-22
