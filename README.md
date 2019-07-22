# closed-loop-stim

This repository mainly houses LabVIEW code for receiving TTL events from the Open Ephys GUI and outputting stimulation pulses on an NI DAQ in response. There are also general VIs for interfacing with Open Ephys in other ways:

* Receiving and sending TTL and text events
* Replaying data via a DAQ's analog output (for input through an Open Ephys ADC channel)
* Replaying stimulations from the events file of a closed-loop run

    * This can now also be done by simply using DAQWriteData on a recording of the stimulations themselves, i.e. one of the ADC channels in a closed-loop run (however, the pulses may be less consistent due to imperfect recording/downsampling)
