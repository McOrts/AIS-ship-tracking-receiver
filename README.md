# AIS-ship-tracking-receiver
**Marine traffic monitoring system based on AIS for Marine Traffic.**

For those who are not marine. AIS was sadly known during the crisis of ship kidnapping in Somalia. I have always wanted to have a receiving station. First I experimented with DVB-T-DAB-FM receiver USB stick. 

This year I have known that the Greek company Marine Traffic offers a free appliance for receive AIS if you are living in an strategic place and send the data to them.

What's AIS?
===========
The marine Automatic Identification System [(AIS)](https://en.wikipedia.org/wiki/Automatic_identification_system) is a location and vessel information reporting system. It allows vessels equipped with AIS to automatically and dynamically share and regularly update their position, speed, course and other information such as vessel identity with similarly equipped vessels or shore installations. Position is derived from the Global Positioning System (GPS) and communication uses Very High Frequency (VHF) digital transmissions.

My station
==========

I've deployed an AIS station based on:
* SLR350Ni AIS Receiver Unit based on Raspberry Pi
* GP-3E ANTENNA
![alt text](https://github.com/McOrts/AIS-ship-tracking-receiver/images/blob/master/AIS-ship-tracking-receiver_Receiver.JPG?raw=true)
You can look at the [station dashboard's](http://www.marinetraffic.com/en/ais/details/stations/4050 "Marine Traffic") via Marine Traffic

Decode AIS
==========

The [GPSD Project](http://catb.org/gpsd/AIVDM.html) has documented the AIS protocol on this page

Small extract of this great decoding manual from Eric S. Raymond [@esrtweet](https://twitter.com/esrtweet "Twitter")
Don't hesitate to help the project on [Gratipay](https://gratipay.com/~esr/)


**Fast explanation about what the fields mean:**

`!AIVDM,1,1,,B,177KQJ5000G?tO``K>RA1wUbN0TKH,0*5C`

* **Field 1**, !AIVDM, identifies this as an AIVDM packet.

* **Field 2** (1 in this example) is the count of fragments in the currently accumulating message. The payload size of each sentence is limited by NMEA 0183’s 82-character maximum, so it is sometimes required to split a payload over several fragment sentences.

* **Field 3** (1 in this example) is the fragment number of this sentence. It will be one-based. A sentence with a fragment count of 1 and a fragment number of 1 is complete in itself.

* **Field 4** (empty in this example) is a sequential message ID for multi-sentence messages.

* **Field 5** (B in this example) is a radio channel code. AIS uses the high side of the duplex from two VHF radio channels: AIS Channel A is 161.975Mhz (87B); AIS Channel B is 162.025Mhz (88B). In the wild, channel codes 1 and 2 may also be encountered; the standards do not prescribe an interpretation of these but it’s obvious enough..

* **Field 6** (177KQJ5000G?tO`K>RA1wUbN0TKH in this example) is the data payload. We’ll describe how to decode this in later sections.

* **Field 7** (0) is the number of fill bits requires to pad the data payload to a 6 bit boundary, ranging from 0 to 5. Equivalently, subtracting 5 from this tells how many least significant bits of the last 6-bit nibble in the data payload should be ignored. Note that this pad byte has a tricky interaction with the <[ITU-1371]> requirement for byte alignment in over-the-air AIS messages; see the detailed discussion of message lengths and alignment in a later section.
The *-separated suffix (*5C) is the NMEA 0183 data-integrity checksum for the sentence, preceded by "*". It is computed on the entire sentence including the AIVDM tag but excluding the leading "!".

For comparison, here is an example of a multifragment sentence with a nonempty message ID field:

`!AIVDM,2,1,3,B,55P5TL01VIaAL@7WKO@mBplU@<PDhh000000001S;AJ::4A80?4i@E53,0*3E`

`!AIVDM,2,2,3,B,1@0000000000000,2*55`

Technically, NMEA0183 does not actually require that a !-led sentence be AIS. This format can be used for any encapsulated data. The syntax and semantics of fields 1-4 are fixed, and the fill-bit field and NEA checksum are required, but the payload fields may contain any encapsulated data.
It is, however, a safe bet that any such sentence containing an A or B channel code in field 5 is AIVDM/AIVDO.
                                                                                                               

**You can decode AIS NMEA message with many tools:**

[**libais**](https://pypi.python.org/pypi/libais) is a good start in Python there are references to other tools on the project page.


**Two examples in Python**

**One line message:**

`2016-04-01 00:09:59, !AIVDM,1,1,,A,240UuRhP1RP716JL4:sLlOwnb6hT,0*3C`

Extract Only the data payload

`import ais`

`ais.decode('240UuRhP1RP716JL4:sLlOwnb6hT',0)`
```json
{
u'slot_timeout': 1L,
u'sync_state': 0L,
u'true_heading': 511L,
u'utc_spare': 0L,
u'sog': 9.800000190734863,
u'rot': -731.386474609375,
u'nav_status': 0L,
u'repeat_indicator': 0L,
u'raim': True,
u'id': 2L,
u'utc_min': 9L,
u'spare': 2L,
u'cog': 328.1000061035156,
u'timestamp': 59L,
u'y': 49.04743194580078,
u'x': 1.5329283475875854,
u'position_accuracy': 1L,
u'utc_hour': 22L,
u'rot_over_range': True,
u'mmsi': 269057419L,
u'special_manoeuvre': 1L
}
```
**Two lines message:**

`2016-04-01 10:46:06, !AIVDM,2,1,4,A,53GR=4400000Ho??C40@D9E==T0000000000001?90>27t@PJ08888888888,0*61`

`2016-04-01 10:46:06, !AIVDM,2,2,4,A,88888888880,2*20`


_TODO: Concatenate the two data payload below and don't forget to replace the 0 args at the end by 2_

`import ais`

`ais.decode('540UuRl00000PF3OC7UHTdTpN18Tp@622222220t4iQ7651<04TSmAC```888888888888880',2)`

	{
	u'destination': u'ROUEN               ',
	u'dim_d': 6L,
	u'name': u'VIKING RINDA        ',
	u'eta_hour': 12L,
	u'ais_version': 1L,
	u'draught': 1.7999999523162842,
	u'mmsi': 269057419L,
	u'repeat_indicator': 0L,
	u'dim_b': 97L,
	u'dim_c': 7L,
	u'dte': 0L,
	u'dim_a': 38L,
	u'eta_day': 2L,
	u'eta_minute': 0L,
	u'callsign': u'HE 7419',
	u'spare': 0L,
	u'eta_month': 4L,
	u'type_and_cargo': 60L,
	u'fix_type': 1L,
	u'id': 5L,
	u'imo_num': 0L
	}`

Licence
=======               
                                                                       

**The data are published under the licence [Creative Commons 4 By](http://creativecommons.org/licenses/by/4.0/)**

###You are free to:

*   Share — copy and redistribute the material in any medium or format
   Adapt — remix, transform, and build upon the material
   for any purpose, even commercially.

*   The licensor cannot revoke these freedoms as long as you follow the license terms.

###Under the following terms:

*   Attribution — You must give appropriate credit, provide a link to the license, and indicate if changes were made.
   You may do so in any reasonable manner, but not in any way that suggests the licensor endorses you or your use.

*   No additional restrictions — You may not apply legal terms or technological measures that legally restrict others from doing anything the license permits.

###Notices:

You do not have to comply with the license for elements of the material in the public domain or where your use is permitted by an applicable exception or limitation.
No warranties are given. The license may not give you all of the permissions necessary for your intended use.
For example, other rights such as publicity, privacy, or moral rights may limit how you use the material
