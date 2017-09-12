# kalarm-sysfs
ulti-Alarm Clock module with SYSFS access

The previous assignment entailed the implementation of a Simple Clock Module & Calendar User Interface, and adds multi-alarm capabilities.

The program consists of two parts:  A user-space component and a kernel module.  
The user-space component creates a kernel timer using itimerval to keep track of the time,
and prints the current date and time to the screen when it receives a signal from the kernel module.
The kernel module is responsible for creating a file in the /proc filesystem; /proc/myclock.  
This file is a read-only file which, when read, returns Unix Time in seconds, and milliseconds since the last recorded second, 
separated by a comma and a single space in the form:

seconds_since_jan_1_1970, milliseconds_since_last_second

The user-space component of our simple clock initially prints out the time, and then waits for a signal from our kernel module after
a pre-designated period of time (here three seconds).  The user-space component then goes through the process of printing out the 
value that it reads from /proc/myclock in a standardized format similar to:
	Wed, Mar 23, 10:22:42 pm 2012
And then waits for the next kernel signal before repeating this process for a grand total of 300 times, 
over a total of 15 minutes!  At the end of the run, the time at which the timer initially started is displayed alongside 
the time at which it finished for ease of comparison.   

The program was run on a Gentoo virtual box.

For this assignment, we added further functionality to our previous clock and calendar homework:  Homework #4.  In the previous 
assignment, we added a read-only /proc filesystem entry which, when read from, returned two simple values separated by a comma:  
The time in milliseconds since Midnight Jan 1, 1970, and the number of milliseconds since the last second which had just passed; 
with a single solitary space between the comma and the number of milliseconds.  For Homework #5, we further added a sysfs entry 
into the folder /sys/kernel using kernel objects (kobjects) and an internal array that allowed the user to set-up up to ten 
separate alarms. 
The sysfs is a virtual filesystem representation of the system’s device topology which allows for easy traversal or enumeration 
of all of a systems devices, including all busses and interconnections.  This tree of devices is enabled through kobjects 
(kernel objects), which are structs found in <linux/kobject.h>.   The kobject struct is likened to an Object class.  It contains
a basic template for the creation of a hierarchy of objects.    
            struct kobject { 
            const char*name;//name of this kobject 
            struct list_headentry; 
            struct kobject*parent;//kobjects parent 
            struct kset*kset; 
            struct kobj_type*ktype; 
            struct sysfs_dirent*sd;//struct containing inode structure representing kobject in sysfs. 
            struct krefkref;// 
            unsigned intstate_initialized:1; 
            unsigned intstate_in_sysfs:1; 
            unsigned intstate_add_uevent_sent:1; 
            unsigned intstate_remove_uevent_sent:1; 
            unsigned intuevent_suppress:1; 
            } 
 
The sysfs entry created for this assignment was a virtual directory called myClock2 (/sys/kernel/myclock2), which contained 
two attributes:  the first was a replication of homework 4’s /proc filesystem entry, in the sysfs, while the second was a writeable 
attribute which acts as an interface for our new kernel timer.  The first node, renamed real_TimeClock (name changed due to an 
ambiguity between the original name “myClock”, and the new “myClock2” entry) returned the time in the format seconds, milliseconds 
exactly as the /proc filesystem entry.  The second attribute is the new and exciting part of this project.  The sysfs entry myClock2 
is writable.  Not only is the virtual entry writable, but writing any integer number to this file starts a kernel timer for the 
number of seconds specified, at the end of which time a real-time alarm is sent to user-space, along with a copy of the value of 
the original timer assignment.   
The user space component of this program loops for 300 seconds (0-299), printing the time every second.  As alarm signals are sent from the kernel-space to the user-space, the signals are caught by the user-space program and the amount of time that the alarm had been set for is printed.  As the user-space program counts from 0-299, it is off by one second.  This is the one second of difference that is seen between the timer alarm messages and the second counts.
