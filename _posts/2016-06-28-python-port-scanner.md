---
title:  "Python Port Scanner"
categories: [security]
tags: ["python", "dev", "code", "port scanner", "security", "network", "cyber security"]
---
Summary: A quick and easy port scanner tool implemented in Python, which is easy to modify and expand

*Why would we create a custom port scanner?*

Sometimes and under some circumstance it is not viable to download or install a tool to run a quick port scan, and a practical tool such as this is a great way to explore python networking basics and demonstrate how easy it is to interact with native sockets on linux and Windows machines through Python.

### Python Port Scanner with ~ 50 lines of code

Firstly we are only using 'standard' python libraries, meaning we do not need to install anything in addition to having a standard python environment running, for Mac OSX this is out-of-the box.

We import the following standard libraries

```python
#! /usr/bin/python3
# Filename : test_port_scanner.py

"""
A basic port scanner in python, using standard socket libraries.
"""

import socket
import subprocess
import sys
import argparse
from datetime import datetime
```
__sockets__ - This is the main python library to interact with system socket level commands and calls, (TCP stack). - Low-level networking interface. for more information on calls and detail refer to python docs - [python sockets](https://docs.python.org/3.5/library/socket.html)

__subprocess__ - This module allows us to spawn new processes and interact with input/output and return codes within the current system - [python subprocess](https://docs.python.org/3.5/library/subprocess.html?highlight=subprocess)

__sys__ __argparse__ and __datetime__ are very common python libraries and used for most tools, applications and scripts, more information can be found @ [docs.python.org](https://docs.python.org)

Next create a function to:

1. Collect user input
2. Scan ports entered via the input
3. Print the information to the screen
4. Begin the scan
5. Print the results per port number to screen
6. Check for errors / or keyboard interrupt
7. Complete the scan with time it took to complete

__Collect user input__

The user is asked for IP address or hostname (our app supports FQDN entries) and a range of port numbers, to enter just a single port number 80 for example the user will enter (79, 80)

```python
def scan_ports(remoteServer, start_port='1', end_port='1024'):

    # Clear the screen
    """

    :rtype : object
    """
    subprocess.call('clear', shell=True)

    # Ask for input
    remoteServer = input("Enter a remote host to scan: ")
    start_port = input("Enter beginning port number: ")
    end_port = input("Enter end port number: ")
    start_port = int(start_port)
    end_port = int(end_port)

```
__Print information to screen__

Here we are simply taking the user input from the previous section and printing a banner for formatting, and displaying the scan details.

```python
    # Print a banner with information on which host we are about to scan
    print ("-" * 60)
    print ("Please wait, scanning remote host", remoteServer)
    print ("On ports: ", start_port, "to", end_port)
    print ("-" * 60)
    print ('\n')

    # Check what time the scan started
    begin_time = datetime.now()
```
__Begin the Scan__

Next we check the current time (this will be used to determine the amount of time the scan took, and begin the scan with a 'try' function and 'for' loop. We do this so we can catch exceptions and errors rather than just hitting the scan function without checking for errors.

Here we use the 'socket' library with the 'sock.' command, the return codes such as 0, 61 are collected by the subprocess to determine the result [0 = success, port open, 61 = TCP SYN rejected, any other = time out]

The specific function to map IP/hostname `remoteServerIP = socket.gethostbyname(remoteServer)`

Then try the connection (talk to the underlying systems TCP stack is `sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)`

We can then set a default timeout for the connection `sock.settimeout(1)`

Provide the results to a new object `result = sock.connect_ex((remoteServerIP, port))`

The complete code for this section is below:

```python
    # Check what time the scan started
    begin_time = datetime.now()

    # Take the user input of 'start_port' and 'end_port' numbers and place them in a range
    # These are the port numbers to be scanned
    try:
        remoteServerIP = socket.gethostbyname(remoteServer)
        for port in range(start_port, end_port):
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(1)
            result = sock.connect_ex((remoteServerIP, port))
            # print(result) - Can be used to test reply codes, 0=ok, 61=TCP RST, etc...
            if result == 0:
                print("Port {}: \t Open".format(port))
            elif result == 61:
                print("Port {}: \t Rejected by Host".format(port))
            else:
                print("Port {}: \t Timed Out".format(port))
            sock.close()
```

__Check for errors and print results__

Once we have the results we can print them to the screen or print the relevant error codes if the scan failed due to system issues or user interrupts

```python
    # Error handling in the event host cannot be reached or no DNS available
    except KeyboardInterrupt:
        print("You pressed Ctrl+C")
        sys.exit()

    except socket.gaierror:
        print('Hostname could not be resolved. Exiting')
        sys.exit()

    except socket.error:
        print('Socket creation failed. Error code: ' + str(err_msg[0]) + ' Error message: ' + err_msg[1])
        sys.exit()

    # Check the time once scan is complete, and compare the start - end times.
    end_time = datetime.now()
    total = end_time - begin_time

    # Print the scan time information
    print ('\n')
    print ('-' * 60)
    print ('Scanning Completed in: ', total)
    print('Remote Host IP: ', remoteServerIP)
    print ('-' * 60)
```

The rest is generic argrparse input and script execution logic, used to determine defaults for when the script is run

```python
if __name__ == "__main__":
    # Command line arguments
    parser = argparse.ArgumentParser(description='Remote Port Scanner')
    parser.add_argument('--remoteServer', action="store", dest="remoteServerIP", default='localhost')
    parser.add_argument('--start-port', action="store", dest="start_port", default=1, type=int)
    parser.add_argument('--end-port', action="store", dest="end_port", default=100, type=int)
    # Parse arguments
    given_args = parser.parse_args()
    remoteServerIP, start_port, end_port = given_args.remoteServerIP, given_args.start_port, given_args.end_port
    scan_ports(remoteServerIP, start_port, end_port)
```

#### Results and test scan

Below is a simple scan to google.com, as mentioned we resolve domain names also within the script. We could just use an IP address here also. The scan is to ports 79 and 80, we have not modified the counter to +1 so it is using the standard python count from 0 for the first integer.

We can confirm that port 79 was rejected by the host, meaning we did receive a response from google, and it did not just drop our TCP SYN packet...

```bash
$ python3 test_port_scanner.py

Enter a remote host to scan: www.google.com
Enter beginning port number: 79
Enter end port number: 81
------------------------------------------------------------
Please wait, scanning remote host www.google.com
On ports:  79 to 81
------------------------------------------------------------


Port 79: 	 Rejected by Host
Port 80: 	 Open


------------------------------------------------------------
Scanning Completed in:  0:00:00.131264
Remote Host IP:  216.58.220.132
------------------------------------------------------------
```

#### Improvements and additions

The sample provided is just a small and quick script (about 50 lines of actual code), this can be expanded and improved by adding some of the following;

* Append results to a file, txt, csv
* Graph the results with something like matplotlib or Pyplot
* Create a GUI for the tool with Tkinter or something supporting multiple platforms like Kivy
* Turn this into a simple .exe file and make it portable
