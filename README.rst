::

           ___  __        __     __                          ___
    |\/| |  |  /__` |  | |__) | /__` |__| |    | __  |\/| | |__  \  /
    |  | |  |  .__/ \__/ |__) | .__/ |  | |    |     |  | | |___  \/
                    __                __        __
                   /  `  /\  |\ |    |__) |  | /__`
                   \__, /~~\ | \|    |__) \__/ .__/


This project aims to document the OBD-II codes for Mitsubishi I-Miev
electric vehicle.

Conventions used in this document:

- Message ID is always 3 hex characters
- Data bytes are zero-indexed: D0, D1, etc

OBD-II Traffic analysis
~~~~~~~~~~~~~~~~~~~~~~~

Periodically occurring PIDs:

- 1000ms (1 fps):
  01C [#note_testmode]_
- 200ms (5 fps):
  568 [#note_testmode]_
- 100ms (10 fps):
  101, 286, 298, 29A, 2F2, 374, 375, 384, 385, 389 [#note_testmode]_,
  38A [#note_testmode]_, 3A4, 408, 412_, 695, 696, 697, 6FA, 75A, 75B
- 50ms (20 fps):
  38D, 564, 565, 5A1, 6D0, 6D1, 6D2, 6D3, 6D4, 6D5, 6D6, 6DA
- 40ms (25 fps):
  424, 6E1, 6E2, 6E3, 6E4
- 20ms (50 fps):
  119, 149, 156, 200, 208_, 210, 212, 215, 231_, 300, 308, 325, 346, 418
- 10ms (100fps):
  236_, 285, 288, 373

.. [#note_testmode] Possibly only sent in debug mode.

PID descriptions
~~~~~~~~~~~~~~~~

101 - Key status
-----------------
Transmitted every 100ms. Data bits:

- D0: ``0x04`` Key is turned on, else ``0x00``

.. _208:

208 - Brake pedal
-----------------
Transmitted every 20ms. Data :

- D0: ``0x00`` (const?)
- D1: ``0x20`` (const?)
- D2-D3: pedal position, ``60:00`` is zero position, max seems to be around ``61:bf``, ``(D2 * 256 + D3) - 24576.0) / 640 * 100.0``,
- D4: ``0xc0`` (const?)
- D5: ``0x00`` (const?)
- D6: ``0xc0`` (const?)
- D7: ``0x00`` (const?)

210 - Accelerator pedal
-----------------
Transmitted every 20ms. Data bits:

- D0: ``0x00`` (const?)
- D1: ``0x20`` (const?)
- D2: Accelerator pedal position: ``D2*0.4``
- D3: ``0xc0`` (const?)
- D4: ``0xc0`` (const?)
- D5: ``0x00`` (const?)
- D6: ``0xc0`` (const?)
- D7: ``0x00`` (const?)

.. _231:

231 - Brake pedal switch sensor
-------------------------------
Transmitted every 20ms. Data bits:

- D0-D3: ``0x00`` (const?)
- D4: ``0x00`` if brake is free, ``0x02`` if brake pedal is pressed
- D5-D7: ``0x00`` (const?)

.. _236:

236 - Steering wheel sensor
---------------------------
Transmitted every 10ms. Data bits:

- D0-D1: Steering wheel position with 0.5 degree accuracy, center point ``(0.0 degrees) = D0:0x10, D1:0x00``. ``(((D0 * 256) + D1) - 4096) / 2`` = steering wheel position in degrees). Negative angle - right, positive angle left.
- D2-D3: possibly represents rate of change, defaults to ``D2:0x10, D3:0x00`` when steering wheel is at rest. ``(((D2 * 256) + D3) - 4096) / 2`` = Steering movement
- D4: counter, but only high-nibble bits (4-7) are used, ``D4[0:3]=0``
- D5: ``0x00`` (const?)
- D6: ``0x00`` (const?)
- D7: ?

286 - Charger/Inverter temperature
---------------------------
Transmitted every 10ms. Data bits:

- D0-D2: ``0x00`` (const?)
- D3: Charger/Inverter temperature ``D3-40``
- D4-D7: ``0x00`` (const?)

298 - Charger/Inverter temperature
---------------------------
Transmitted every 100ms. Data bits:

- D0: ``0x34`` (const?)
- D1: ``0x35`` (const?)
- D2: ``0x34`` - ``0x35``
- D3: Motor temperature: ``D3-40``
- D4: ``0xc0`` (const?)
- D5: ``0x00`` (const?)
- D6 - D7: Motor RPM: ``((D6*256.0)+D7)-10000.0``

29A - VIN
---------------------------
Transmitted every 100ms. Data bits:

- D0:
- D1:
- D2:
- D3:
- D4:
- D5:
- D6
- D7:

346 - Estimated range / Handbrake
---------------------------
Transmitted every 20ms. Data bits:

- D0: ``0x27`` (const?)
- D1: ``0x10`` (const?)
- D2: ``0x57`` (const?)
- D3: ``0x20`` HandBrake is not used
- D4: ``0x20`` (const?)
- D5: ``0x00`` (const?)
- D6: ``0x00`` (const?)
- D7: Estimated range ``D7``

373 - Main Battery Voltage and Current
---------------------------
Transmitted every 10ms. Data bits:

- D0: ``0x00`` (const?)
- D1: ``0x00`` (const?)
- D2 - D3: Main battery current Ampers: ``((((D2*256.0)+D3)-32768)/100.0)``
- D4 - D5: Main battery voltage Volts: ``((D4*256.0+D5)/10.0)``
- D6: ``0x00`` (const?)
- D7: ``0x00`` (const?)

374 - Main Battery SoC
---------------------------
Transmitted every 100ms. Data bits:

- D0: ``0x00`` (const?)
- D1: SoC ``(D1-10)/2``
- D2: ``0x00`` (const?)
- D3: ``0x00`` (const?)
- D4: ``0x00`` (const?)
- D5: ``0x00`` (const?)
- D6: ``0x00`` (const?)
- D7: ``0x00`` (const?)

384 - Heating currents and temps / AC current
---------------------------
Transmitted every 100ms. Data bits:

- D0 - D1: AC current: ``(D0*256.0+D1)/1000.0``
- D2: ?
- D3: ?
- D4: Heating current: D4/10
- D5: Heating temp return: old: ``((D5-32)/1.8)-3`` new: ``(D5 * 0.6) - 40.0``
- D6: Heating temp flow: ``((D6-32)/1.8)-3``, new: ``(D6 * 0.6) - 40.0d``
- D7: ?
old: bau year 2011 or older

389 - Charger voltage and current
---------------------------
Transmitted every 100ms. Data bits:

- D0:?
- D1: Charger voltage (Type1): ``D1``
- D2: ?
- D3: ?
- D4: ?
- D5: ?
- D6: Charger current (Type1): ``D6/10.0``
- D7: ?

3A4 - Climate console
---------------------------
Transmitted every 100ms. Data bits:

- D0:
  - heating level:
    - ``D0<<4 == 112`` : middle
    - ``D0<<4 > 112`` : Heating
    - ``D0<<4 < 112`` : Cooling
  - AC ON:
    - ``D0>>7 == 1``

- D1: ventilation direction
  - (D1 & 0xf0) >> 4 == x
    - ``x=1,2`` Face
    - ``x=3,4`` Legs + Face
    - ``x=5,6`` Legs
    - ``x=7,8`` Legs + Windwshield
    - ``x=9`` Windshield
- D2: ?
- D3: ?
- D4: ?
- D5: ?
- D6: ?
- D7: ?

.. _412:

412 - Speed + Odometer value
----------------------------
Transmitted every 100ms. Data bits:

- D0: ``0x7f`` (const?)
- D1: speed in km/h: ``D1``
- D2-D4: Odometer display in km. ``(D2<<16)+(D3<<8)+D4``
- D5: ``0x00`` (const?)
- D6: ?
- D7: ``0x06`` (const?)

418 - Transmission
----------------------------
Transmitted every 20ms. Data bits:

- D0:
    - P: ``D0 = 0x50``
    - R: ``D0 = 0x52``
    - N: ``D0 = 0x4E``
    - D: ``D0 = 0x44``
    - B: ``D0 = 0x83``
    - C: ``D0 = 0x32``
- D1: ``0x00`` (const?)
- D2: ``0x00`` (const?)
- D3: ``0x00`` (const?)
- D4: ``0x00`` (const?)
- D5: ``0x00`` (const?)
- D6: ``0x00`` (const?)
- D7: ``0x06`` (const?)

424 - Lights
----------------------------
Transmitted every 100ms. Data bits:

- D0:
  - ``D0 & 0x02 != 0`` Automatic Light
  - ``D0 & 0x04 != 0`` Night Headlights
  - ``D0 & 0x08 != 0`` Front Fog lights
  - ``D0 & 0x10 != 0`` Rear Fog Lights
- D1:
  - ``D1 & 0x00 != 0`` Hazard Light
  - ``D1 & 0x01 != 0`` Blinker Right
  - ``D1 & 0x02 != 0`` Blinker Left
  - ``D1 & 0x04 != 0`` HighBeam
  - ``D1 & 0x04 != 0`` Night Headlights
  - ``D1 & 0x05 != 0`` SideLights
  - ``D1 & 0x20 != 0`` Headlight
  - ``D1 & 0x40!= 0`` ParkingLight
- D2:
  - ``D2 & 0x01 != 0`` Any Door OPEN
  - ``D2 & 0x02 != 0`` Front Left Door OPEN
- D3: ``0x00`` (const?)
- D4: ``0x05`` (const?)
- D5:
- D6:
  - ``D6 & 0x04 != 0`` Rear windows heating
- D7: ``0xff`` (const?)

6E1/6E2/6E3/6E4 - Battery Voltages and temperatures
----------------------------
Transmitted every 40ms. Data bits:
No temp data in 6E4

- D0: counter
- D1: ``0x00`` (const?) only 6E1, else: Temp: ``D1-50``
- D2: Temp: D2-50
- D3: Temp (only 6E1): D3-50, else ``0x00`` (const?)
- D4 - D5: Voltages ((D4*256+D5)/200)+2.1
- D6 - D7: Voltages ((D6*256+D7)/200)+2.1
