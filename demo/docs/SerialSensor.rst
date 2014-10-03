Using the SerialSensor Module
=============================

Content within the module
-------------------------

============ ===========
Content        Description
------------ -----------
SerialSensor Used to interact with sensors via a serial port
SerialError  Passes information from sensors that raised an exception
listPorts()  Lists available serial ports on the host machine
============ ===========


Importing the module
--------------------

The SerialSensor class can be imported by itself, however sensor information is passed through SerialError when exceptions occur, so it is suggested that it be imported as well. listPorts can be imported if the available serial ports are not known.

>>> from serialsensor import SerialSensor, SerialError


Using SerialSensor
------------------

Following is an example that instantiates one sensor on ``/dev/ttyUSB0``, and prints the received data in the different available formats::

    from serialsensor import SerialSensor, SerialError
    import time

    CRLF = 0
    CR = 1
    LF = 2

    #Instatiating a Temperature sensor located on '/dev/ttyUSB0', whose measurement is given in degrees 
    #Celsius, each measurement takes 400 milliseconds, and the Baud rate of the sensor is 38400 b/s.
    sensor = SerialSensor('Temperature', 'C', '/dev/ttyUSB0', 400, 38400)

    #Print information about sensor:
    print "Reading from", sensor.getName(), "sensor:"

    #Issue a reading command. In this case, the sensor outputs a reading 400ms after the 'r\r' command.
    sensor.send('r\r')

    #Wait for the required time. Since the Wait Time is given in ms, it converted to s dividing by 1000
    time.sleep(sensor.getWaitTime()/1000)

    #Make readings:
    #Sensor data as read, converted to CRLF line break:
    read_string = sensor.readString(CRLF)

    #Sensor data in floating point values:
    read_value = sensor.readValues()

    #Sensor data in JSON dictionary format, containing the sensor's name, unit and measurement:
    read_json = sensor.readJSON()

    #Print measurements:
    print "String:", read_string
    print "Value:", read_value
    print "JSON:", read_json

Sample output::

    Reading from Temperature sensor:
    String: 21.98\r\n
    Value: 21.98
    JSON: {"Temperature":{"value":21.98, "units":"C"}}


The SerialSensor class can also handle sensor that send CSV values, just instatiate the sensor accordingly, for example:

Sensor instantiation::

    #Separate name and units by commas
    sensor = SerialSensor('Temp_c,Temp_f,Temp_k', 'C,F,K', '/dev/ttyUSB0', 400, 38400)

Sample output::

    Sensor output: 21.98,71.56,295.13\r
    String: 21.98,71.56,295.13\r\n
    Value: [21.98,71.56,295.13]
    JSON: {"Temp_c":{"value":21.98, "units":"C"}, "Temp_f":{"value":71.56, "units":"F"}, "Temp_k":{"value":295.13, "units":"K"},}


SerialError
-----------

In case of an exception, thrown either by serial errors or invalid data types, SerialSensor can throw these exceptions:

================= ============ ===========
Exception         Error number Description
----------------- ------------ -----------
Could not connect 0            Could not connect to the serial port during write or read operations
Invalid Data Type 2            Value received from sensor is of non numeric data type, and cannot be converted 
================= ============ ===========

When thrown, SerialError errors contain the following arguments:

================= ===========
Argument          Description
----------------- -----------
args              Descriptor
errno             Error number
port              Port the sensor's connected to
sensor            Sensor name as defined during initialization
msg               Arbitrary message. On errno=2, the msg field contains the string read from the sensor, for debugging purposes
================= ===========

The ``checkConnection()`` method can be invoked on a sensor to check for serial connection errors, if the connection is unavailable, ``cehckConnection(True)`` can be used to try to repair the serial connection.


Methods Details
---------------

``send()``: The send method by default sends commands with CR line breaks, even if different line breaks (or none at all) are given on the argument. An exception is thrown if the sensor cannot be reached (errno 0)

``readJSON()``: The readJSON method returns the smallest number of values available, i.e. if the number of measurement names or units provided, or the number of values read is different from one another, name, unit and value will be matched until there isn't one of each to be matched.

``readString()``: The readString mothod returns by default a CRLF ended string, CR and LF can be chosen by using 1 and 2 as arguments, respectively.













