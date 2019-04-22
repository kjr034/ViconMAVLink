# ViconMAVLink
ViconMAVLink is an application to provide indoor positioning for networked robots using Vicon motion capture measurements. It is primarily used in the [Intellegent Robotics Lab](http://robotics.illinois.edu) at the University of Illinois as a tool to simulate indoor GPS for Linux-based autonomous vehicles, such that you can operate autonomous vehicles indoors:

_A fleet of UAVs flying with simulated GPS_:

<img src="figs/launching.gif" width="600" align="center"> 


_Heterogeneous autonomous vehicles controlled via tablet groundcontrol_:

<img src="figs/indoorGPS.gif" width="600" align="center"> 


# Dependencies
ViconMAVLink uses the following libraries:
* [Vicon DataStream SDK](https://www.vicon.com/products/software/datastream-sdk): to obtain real-time, high-precision positioning measurement for objects.
* [MAVLink library](http://qgroundcontrol.org/mavlink/start): to encode/unpack UDP data with the MAVLink protocol.
* [GeographicLib](https://geographiclib.sourceforge.io/): to convert positioning data between local coordinates and GPS locations.
* [QT5](https://www.qt.io/) (core gui widgets network concurrent): to provide a GUI, networking and multithreading.

As long as you have all required libraries added to the Qt project file. This application can be built for multiple platforms in [QtCreator](https://www.qt.io/qt-features-libraries-apis-tools-and-ide/).

# User's Guide
You can run this app directly on the Vicon server computer or on any other computers as long as they are in the same network as the Vicon server computer. After you launch the application, the main window will appear. But, the capture objects list is empty because you haven't connected to Vicon yet. 

* The `HostAddress` is the IP address of the Vicon server computer. 
* The `HostPort` usually is `801`. This is the port used by Vicon.
* The `North` mapping: Vicon uses a _Forward-Left-Up_ coornidates while the `LOCAL_POSITION_NED` message in MAVLink uses _North-East-Down_, to make the situation more confusing, GeographicLib uses a _East-North-Up_ coordinates. To convert position data correctly, you need to map your Vicon axis first. Assuming you calibrated Vicon according to the manual, just choose the correct half-axis that corresponds to North then this app will handle the mapping correctly.

__To connect to Vicon__: Click `Menu`->`Connect Vicon` then objects that are captured by Vicon will appear in the Captured Objects List.

<img src="figs/main_window_connect.gif" width="400">

__To launch a sender for a robot__: Choose an object then Click `Start a MavLink Sender`, the Sender's window appears. After you start sending data, the `Rate` slider is still adjustable to change the sending rate on-the-fly.

<img src="figs/sender_window_connect.gif" width="400">

You may launch multiple senders for a fleet of robots:

<img src="figs/fleet.gif" width="400">

# Developer's Guide
The Station object will launch a separate thread to communicate with Vicon. A Writer/Reader lock is used to synchronize data. The Station object is the writer and Sender objects are the readers who fetch raw measurement from the Station. The raw measurement includes position (unit: mm) and a quarternion encapsulated in a `mavlink_att_pos_mocap_t` object. Since Vicon does not output velocities, the Sender uses a linear Kalman filter to compute a `mavlink_local_position_ned_t` object with positions and velocities. The `GPS_HIL` data is then computed with the local position object. The Station and Sender objects and windows use the Model-View-Controller pattern.

# Use with Mission Planner
In order to use this particular system with Mission Planner/ Ardupilot, there are a couple more setup items that need to be done. First, be sure to set up the telemetry ports correctly on whatever device you are using. In my application, I used a Emlid Navio2 and had to change the /etc/default/arducopter file so that the Telemetry port 2 could receive data from a particular UART configuration. In other words, set TELEM2 command to be:

TELEM2 = "-C /dev/ttyAMA0" --> this is a non-obvious requirement for this system to work. Don't use the "external GPS" options -B and -E.

You will also need to go ahead and change some settings in Ardupilot through the mission planner. Assuming that your system has close to the default settings, go in and change the "GPS_1" value to be 14 (or really, just set the GPS input to Mavlink). You can also set the second GPS to be something different (I chose HIL, but it really doesn't matter-- you just need to have the GPS stuff done). 

Next, using this ViconMAVLink program files, go into tools (which, I assume you have already downloaded at this point onto the device that you are sending information to from the Vicon). Inside tools, use the test programs in UdaReceiver to test whether or not your connection is actually working. If you have information being received (after you start sending from the base location), then move onto the next step. Otherwise, keep debugging the issue until you figure out the problem. Be sure to use Python2 to run the program (it won't work with Python3). 

Once you've received some data using the test program, kill that program. next, go into the other folder in tools, UdpPassthrough. Inside there will be two files you can use. First, make sure the baud rate that you are using is accurate within the file UdpPass.py. Also, ensure that you have the right port on the device which will be receiving the information. Second, start running the passthrough_example.py (again, with Python2). This will start passing to the port specified in the example. Note that /dev/ttyAMA0 is the uart port (which in this case for the Navio2 was TELEM2). by using this port it will output the GPS data, and you should see a position fix occur on the system.

