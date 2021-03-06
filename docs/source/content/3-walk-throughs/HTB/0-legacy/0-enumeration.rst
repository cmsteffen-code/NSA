.. _Legacy Enumeration:

Enumeration
===========

.. index::
   single: nmap

+-------------+------------------+
|**Reference**|:ref:`nmap <nmap>`|
+-------------+------------------+



Pinging the Target
------------------

After connecting to the HackTheBox VPN and initializing the `Legacy` VM, we can begin enumerating the target. We'll start by using `nmap` to ping the system, to see if it's online:

.. code-block:: none

    kali@kali:~$ nmap -sn 10.10.10.4
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-14 13:27 EDT
    Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn
    Nmap done: 1 IP address (0 hosts up) scanned in 3.07 seconds

Some systems will not respond to pings despite being online. This is often true for `Windows` systems, such as our current target. This is why `nmap` reported that the host seems to be offline. Fortunately for us, we know that `Legacy` is online and listening, because the HackTheBox dashboard tells us so.

.. figure:: images/0-legacy-online.png
   :width: 400 px
   :align: center
   :alt: Screenshot of HackTheBox, showing that `Legacy` is online.

   Screenshot of HackTheBox, showing that `Legacy` is online.



Scanning for Ports
------------------

Next, we'll identify open ports on the target, as well as what software might be listening on each of those ports. Since we know the target isn't responding to ICMP pings, we'll use the ``-Pn`` command-line argument to tell `nmap` to scan without pinging first:

.. code-block:: none

    kali@kali:~$ nmap -Pn 10.10.10.4
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-14 12:52 EDT
    Nmap scan report for 10.10.10.4
    Host is up (0.056s latency).
    Not shown: 997 filtered ports
    PORT     STATE  SERVICE
    139/tcp  open   netbios-ssn
    445/tcp  open   microsoft-ds
    3389/tcp closed ms-wbt-server

    Nmap done: 1 IP address (1 host up) scanned in 7.83 seconds

Excellent! We've identified two open ports: 139 and 445. According to `nmap`, these ports might be running `netbios-ssn` and `microsoft-ds`, respectively.



Identifying Services
--------------------

Let's see if we can learn more about these ports by using the ``-sV`` command-line argument, which enables version detection, and the ``-O`` command-line argument, which enables OS detection. We'll also use the ``-p`` command-line argument, to narrow the scan to ports 139 and 445. OS detection requires root privileges, so we'll use `sudo` to run the tool as root:

.. code-block:: none

    kali@kali:~$ sudo nmap -Pn -sV -O -p139,445 10.10.10.4
    [sudo] password for kali:
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-03-14 14:22 EDT
    Nmap scan report for 10.10.10.4
    Host is up (0.053s latency).

    PORT    STATE SERVICE      VERSION
    139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
    445/tcp open  microsoft-ds Microsoft Windows XP microsoft-ds
    Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
    Device type: general purpose
    Running (JUST GUESSING): Microsoft Windows 2000|XP|2003 (90%)
    [...]
    Aggressive OS guesses: Microsoft Windows 2000 SP4 or Windows XP SP2 or SP3 (90%), Microsoft Windows XP SP2 (89%), [...]
    No exact OS matches for host (test conditions non-ideal).
    Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

    OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 10.96 seconds

As you can see, this `nmap` scan provided significantly more detail about the target system. While it didn't tell us much more about the specific software running on the specified ports, it was able to determine that the target was running `Micrsoft Windows XP`, most likely with Service Pack 2 (SP2) or Service Pack 3 (SP3) installed.
