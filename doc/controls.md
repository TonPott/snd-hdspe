ALSA control elements
=====================

As usual

     alsactl -f asound.state store

stores the actual ALSA control elements with all metadata and exact enumeration values 
in the file name asound.state.

See [hdspeconf](https://github.com/PhilippeBekaert/hdspeconf) for an example application using (and showing how to use) these elements.

The controls correspond for most part with the similarly named controls in the Windows and MAC OSX control applications provided by RME.
See the RME HDSPe sound card user guides, and [hdspeconf](https://github.com/PhilippeBekaert/hdspeconf) documentation
for more information on these controls.

**Acces modes**:

The access values in the tables below are a combination of the following symbols:

| Symbol | Meaning |
| :- | :- |
| R | Control element is readable |
| W | Control element is writable |
| V | Control element is volatile |


Controls common to all supported cards
--------------------------------------

| Interface | Name | Access | Value Type | Description |
| :- | :- | :- | :- | :- |
| CARD | Card Revision | R | Int | PCI class revision. Uniquely identifies card model.  | 
| CARD | Firmware Build | R | Int | Firmware version.           | 
| CARD | Serial | R | Int | Card serial number.            | 
| CARD | TCO Present | R | Bool | Whether or not TCO module is present.            | 
| CARD | Capture PID | RV | Int | Current capture process ID, or -1.            | 
| CARD | Playback PID | RV | Int | Current playback process ID, or -1.            | 
| CARD | Running | RV | Bool | Whether or not some process is capturing or playing back.            | 
| CARD | Buffer Size | RV | Int | Sample buffer size, in frames.            | 
| CARD | Status Polling | RWV | Int | See below **Status Polling**            | 
| HWDEP | DDS | RW | Int | See below **DDS**            | 
| HWDEP | Raw Sample Rate | RV | Int64 | See below **DDS**            | 
| CARD | Clock Mode | RW | Enum | Master or AutoSync.            | 
| CARD | Preferred AutoSync Reference | RW | Enum | Preferred clock source, if in AutoSync mode.            | 
| CARD | Current AutoSync Reference | RV | Enum | Current clock source. | 
| CARD | AutoSync Status | RV | Enum | AutoSync clock status: N/A, No Lock, Lock or Sync, for all sources.            | 
| CARD | AutoSync Frequency | RV | Enum | Current clock source sample rate class, for all sources: 32 kHz, 44.1 kHz, 48 kHz, 64 kHz, 88.2 kHz, 96 kHz, 128 kHz 176.4 kHz 192 kHz. Note: MADI cards only report this for the MADI input and not for the other sources. | 
| CARD | Internal Frequency | RW | Enum | Internal sampling rate class: 32 kHz, 44.1 kHz, 48 kHz etc....           | 

**Status Polling**

Use this control element to enable or disable kernel space status polling. The value of this element is
the frequency at which to perform status polling in the driver, or 0 to disable the feature. 
If non-zero, the driver will poll for card changes in the value of volatile control elements
at approximately the indicated frequency.
A notification event is generated on any status ALSA control elements that has changed. If any
have changed, the value of the Status Polling control element is reset to 0, notifying client 
applications, and effectively disabling
status polling until a client application enables it again by setting a non-zero value. Status
polling is also automatically disabled after a few seconds. When automatically disabled, a notification
is sent as well, so that client applications can re-enable it. Whenever an application receives 
a "Status Polling" event notification, it shall set the value of this control element to the maximum
of its reported value and the applications desired value to avoid ping-ponging changes with other
applications.

**DDS**

The HDSPe cards report effective sampling frequency as a ratio of a fixed frequency constant 
typical for the card, and the content of a register. This ratio is returned
in the "Raw Sample Rate" control element. The numerator is the first value, and is a true 64-bit value. The denominator is a 32-bit value, and provided as the second value.

The "DDS" control element enables setting the DDS register, determining internal sample rate to sub-Hz accuracy. The DDS register value is the denominator of the desired sample rate, given as a ratio
with same numerator as the "Raw Sample Rate" control element, i.o.w. the numerator is the first value
of the "Raw Sample Rate" control element.
This can be used to synchronise the cards internal clock to e.g. a system clock.


TCO controls
------------

| Interface | Name | Access | Value Type | Description |
| :- | :- | :- | :- | :- |
| CARD | TCO Firmware | RO | Int | TCO module firmware version |
| CARD | LTC In | RV | Int64 | Incoming LTC code - see below **LTC control** | 
| CARD | LTC In Drop Frame | RV | Bool | Whether incoming LTC is drop frame format or not | 
| CARD | LTC In Frame Rate | RV | Enum | Incoming **LTC frame rate**: 24, 25 or 30 fps | 
| CARD | LTC In Pull Factor | RV | Int | Incoming **LTC frame rate** deviation from standard | 
| CARD | LTC In Valid | RV | Bool | Whether or not valid LTC input is detected | 
| CARD | LTC Out | W | Int64 | LTC output control - see below **LTC control** |
| CARD | LTC Time | RV | Int64 | Current periods end LTC time - see below **LTC control** | 
| CARD | LTC Run | RW | Bool | Pauze / restart LTC output | 
| CARD | LTC Frame Rate | RW | Enum | TCO LTC engine frame rate: 24, 25, 29.97, 29.97 DF or 30 fps | 
| CARD | LTC Sample Rate | RW | Enum | TCO LTC engine audio sample rate: 44.1 kHz, 48 kHz, **From App** | 
| CARD | TCO Lock | RV | Bool | Whether or not the TCO is locked to LTC, Video or Word Clock | 
| CARD | TCO Pull | RW | Enum | Pull Up / Pull Down factor |
| CARD | TCO Sync Source | RW | Enum | TCO preferred synchronisation source: LTC, Video or Word Clock | 
| CARD | TCO Video Format | RV | Enum | Video format reference signal detected: PAL or NTSC blackburst. Firmware 11 or higher detect SDI reference signals of potential other frame rates, use TCO Video Frame Rate control if TCO firmware >- 11|
| CARD | TCO Video Frame Rate | RV | Enum | Video frame rate detected (meaningful only if TCO firmware >= 11) |
| CARD | TCO WordClk Conversion | RW | Enum | Word clock rate conversion 1:1, 44.1 -> 48 kHz, 48 -> 44.1 kHz | 
| CARD | TCO WordClk Term | RW | Bool | Whether or not to 75 Ohm terminate the word clock/video input BNC | 
| CARD | TCO WordClk Valid | RV | Bool | Whether or not a valid word clock signal is detected | 
| CARD | TCO WordClk Speed | RV | Enum | Detected input word clock speed |
| CARD | TCO WorldClk Out Speed | RW | Enum | Output word clock speed |

**LTC Control**

If a valid LTC signal is presented to the TCO module, the 'LTC In' control
will report two 64-bit values: the current LTC 64-bit code and the LTC time
at which that current code started.

The current LTC code is the code of the last fully received LTC frame, incremented by one frame. The current LTC start time in fact is the time at which that last fully received LTC frame ended. This is correct, and what a user expects, for continuous and forward running LTC. For stationary, jumping, or backward running LTC, the code shall be corrected appropriately.

LTC output is started by writing the 'LTC Out' control. 
The 'LTC Out' control contains two 64-bit values similar to the 'LTC In' control
element: the LTC 64-bit code
to start the LTC output with, and the LTC time at which that code shall be
started. If the indicated time is in the past or further in the future
than a few LTC frame durations, the start time and time code are adapted
accordingly to create output that would result if the indicated time code
were started at the indicated time (if in the past) or that will result in
the indicated time code being generated at the indicated time (if in the
future).

The SMPTE 12-1 standard defines LTC as a 80-bit code, with 64 data bits and
16 synchronisation bits. The 64-bit LTC code mentioned above corresponds to
the 64 data bits of an SMPTE 12-1 80-bit code. The data bits are described
in the [SMPTE 12-1 standard](https://ieeexplore.ieee.org/document/7291029/definitions?anchor=definitions) and on [wikipedia](https://en.wikipedia.org/wiki/Linear_timecode).

The LTC time is the number of audio frames processed since the snd-hdspe driver
was started. The 'LTC Time' control allows to read the LTC time corresponding to
the end of the current period. In a jack audio connection kit application
process callback, an application would query the current period jack audio frame
counter using the jack_get_cycle_times() call and add the period size, on
the one hand side. The application would read the LTC time from the 'LTC Time'
control on the other hand size. The thus obtained counters represent the
same moment in time, and allow to convert the time reported in a 'LTC In'
control to a jack audio frame count, or to calculate the
LTC time at which LTC output shall be started from a jack audio frame count.

*Note* ALSA probably provides mechanisms to query clocks, such as the here
described LTC time, more efficiently than through a control element. In
future, the 'LTC Time' control element may be replaced by such more efficient
mechanism.

Examples:
- positional time code is generated by starting code 00:00:00:00 at time 0.
The output time code will reflect the time since the start of the driver.
- jam sync: set 'LTC Out' time code and time to the current 'LTC In' time code
and time.

Outputting wall clock LTC: set 'LTC Out' time code to the special value -1,
and the time to the number of seconds east of GMT corresponding to the
time zone, and corrected for daylight saving time if in effect, if the computers
real time clock is set to UTC and not to local time zone.
The driver will replace a hexadecimal time code of 3f:7f:7f:3f by the exact
value of the real-time clock at the time the request gets processed, and add
the number of seconds indicated in the time field of the control. Such
hexadecimal time code results from setting the first value of the 'LTC Out'
control to -1.

The deviation of local time w.r.t. UTC can be queried with the localtime_r() GNU libc call.

**LTC frame rate**

SMPTE 12-1 time codes contain control bits indicating the frame rate of the LTC: 24, 25 or 30 fps. The frame rate standard of incoming LTC is
reported in the 'LTC In Frame Rate' control. 

The effective frame rate may however deviate from what the frame rate bits in the LTC codes indicate. For instance, NTSC 29.97 fps is reported
as 30 fps. The deviation between actual and standard frame rate is reported in the 'LTC In Pull Factor' control. This control returns a value of
1000 for nominal speed, less than 1000 for slower rates and greater than 1000 for higher effective rate. The value results from measuring the
actual LTC frame duration in the driver.

Example: 29.97 NTSC pull down LTC will be reported with a pull factor of 999. 

The 'LTC Frame Rate' property controls the TCO LTC engine frame rate. Usually, 'LTC Frame Rate' and 'TCO Pull' shall be set to match the incoming LTC effective frame rate, in order to produce a clean 44.1 kHz or 48 kHz sample clock synchronisation. But it also sets the frame rate for LTC output.

**From App**

The 'From App' LTC sample rate setting will set the TCO LTC engine sample rate
to match the audio card sample rate class: 44.1 kHz if the sound card is
running at 44.1 kHz, and 48 kHz otherwise (the TCO does not support 32 kHz
sample rate).


AES controls:
-------------

| Interface | Name | Access | Value Type | Description |
| :- | :- | :- | :- | :- |
| CARD | Double Speed Mode | RW | Enum | Double speed mode: Single Wire or Double Wire |
| CARD | Quad Speed Mode | RW | Enum | Quad speed mode: Single Wire, Double Wire or Quad Wire | 
| CARD | Professional | RW | Bool | If true, output professional mode AES (5V, professional mode status bits). If false, outputs 2V and consumer status bits, compatible with S/PDIF HiFi equipment e.g. | 
| CARD | Emphasis | RW | Bool | Enable high frequency emphasis status bit in output. | 
| CARD | Non Audio | RW | Bool | Enable non-audio (dolby/AC3) status bits in output. |
| CARD | Line Out | RW | Bool | On by default. Disable for AC3 output. |
| CARD | Single Speed WordClk Out | RW | Bool | Output single-speed word clock signal, also when running in double or quad speed mode | 
| CARD | Clear TMS | RW | Bool | Clear track-marker and status bits from AES and ADAT audio samples. If not set, these bits are available as the least significant bits of PCM data. | 


AIO controls
------------

| Interface | Name | Access | Value Type | Description |
| :- | :- | :- | :- | :- |
| CARD | Input Level | RW | Enum | Analog audio input reference level: -10 dBV (with 12 dB headroom), +4 dBu (with 9dB headroom), Lo Gain (+4 dBu with 15 dB headroom) | 
| CARD | Output Level | RW | Enum | Analog audio output reference level: -10 dBV (with 12 dB headroom), +4 dBu (with 9dB headroom), Hi Gain (+4 dBu with 15 dB headroom) | 
| CARD | XLR Breakout Cable | RW | Enum | Analog output breakout cable: XLR or RCA. -6 dB gain correction on XLR for correct reference level |
| CARD | Phones Level | RW | Enum | Headphones output level: same options as Output Level | 
| CARD | S/PDIF In | RW | Enum | S/PDIF input connector: coaxial, optical or internal | 
| CARD | S/PDIF Out Optical | RW | Bool | Output S/PDIF over TOSLINK | 
| CARD | S/PDIF Out Professional | RW | Bool | Output professional mode S/PDIF | 
| CARD | ADAT Internal | RW | Bool | Use the internal connector for ADAT, with AEB or TEB expansion board | 
| CARD | Single Speed WordClk Out | RW | Bool | Output single-speed word clock signal, also when running in double or quad speed mode | 
| CARD | Clear TMS | RW | Bool | Clear track-marker and status bits from AES and ADAT audio samples. If not set, these bits are available as the least significant bits of PCM data. | 
| CARD | AO4S Present | RO | Bool | AO4S-192 analog output extension board present |
| CARD | AI4S Present | RO | Bool | AI4S-192 analog input extension board present |


AIO Pro controls
----------------

| Interface | Name | Access | Value Type | Description |
| :- | :- | :- | :- | :- |
| CARD | Input Level | RW | Enum | Analog audio **input level** | 
| CARD | Output Level | RW | Enum | Analog audio **output level**            | 
| CARD | Phones Level | RW | Enum | Headphones output level: High power or Low power | 
| CARD | S/PDIF In | RW | Enum | S/PDIF input connector: coaxial, optical or internal | 
| CARD | S/PDIF Out Optical | RW | Bool | Output S/PDIF over TOSLINK | 
| CARD | S/PDIF Out Professional | RW | Bool | Output professional mode S/PDIF | 
| CARD | ADAT Internal | RW | Bool | Use the internal connector for ADAT, with AEB or TEB expansion board | 
| CARD | Single Speed WordClk Out | RW | Bool | Output single-speed word clock signal, also when running in double or quad speed mode | 
| CARD | Clear TMS | RW | Bool | Clear track-marker and status bits from AES and ADAT audio samples. If not set, these bits are available as the least significant bits of PCM data. |

**Input level**

Full scale PCM input data for analog input coresponds to +4, +13, +19 or +24 dBu level.

**Output level**

Full scale PCM output data for analog output corresponds to +4, +13, +19 or +24 dBu level if outputting balanced audio (using the XLR breakout
cable), or -2, +4, +13 or +19 dBu level if outputting unbalanced audio (using the RCA breakout cable).


MADI controls
-------------

| Interface | Name | Access | Value Type | Description |
| :- | :- | :- | :- | :- |
| CARD | External Frequency | RV | Enum | Frequency class of the current autosync reference: 32kHz, 44.1kHz, 48kHz, etc... (MADI cards do not report the frequency class of each autosync reference individually, like other cards do.) |
| CARD | Preferred Input | RW | Enum | Preferred MADI input: Optical, Coaxial |
| CARD | Autoselect Input | RW | Bool | Whether or not to automatically switch over input if preferred input is not available (a.k.a. safe mode) |
| CARD | Current Input | RV | Enum | Current MADI input: Optical, Coaxial |
| CARD | RX 64 Channels Mode | RV | Bool | Whether or not we're currently receiving 64 channels mode (true) or 56 channels mode (false) MADI input |
| CARD | TX 64 Channels Mode | RW | Bool | Transmit 64 channels mode (true) or 56 channels mode (false) |
| CARD | Double Wire Mode | RW | Bool | Double speed mode: 48K frame mode (= S/MUX or double wire mode) if true, 96K frame (single wire) mode if false |
| CARD | Line Out | RW | Bool | Enable/disable headphone output |
| CARD | Single Speed WordClk Out | RW | Bool | Output single-speed word clock signal, also when running in double or quad speed mode | 
| CARD | Clear TMS | RW | Bool | Clear track-marker and status bits from MADI audio samples. If not set, these bits are available as the least significant bits of PCM data. |


RayDAT controls
---------------

| Interface | Name | Access | Value Type | Description |
| :- | :- | :- | :- | :- |
| CARD | S/PDIF In | RW | Enum | S/PDIF input connector: coaxial, optical from ADAT4 connector, or internal | 
| CARD | S/PDIF Out Optical | RW | Bool | Output S/PDIF on ADAT4 connector | 
| CARD | S/PDIF Out Professional | RW | Bool | Output professional mode S/PDIF | 
| CARD | ADAT1 Internal | RW | Bool | Use the internal ADAT1 connector instead of optical, for AEB or TEB expansion board | 
| CARD | ADAT2 Internal | RW | Bool | Use the internal ADAT2 connector instead of optical, for AEB or TEB expansion board | 
| CARD | Single Speed WordClk Out | RW | Bool | Output single-speed word clock signal, also when running in double or quad speed mode | 
| CARD | Clear TMS | RW | Bool | Clear track-marker and status bits from AES and ADAT audio samples. If not set, these bits are available as the least significant bits of PCM data. |
