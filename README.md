# kalarm-sysfs
ulti-Alarm Clock module with SYSFS access

usage: alarm_user ([N])*  
N is the number of seconds to set the alarm for  you may have up to ten distinct alarm times per command line argument. 
	ex:  ./kalarm-sysfs 4 2 5 7 20
		will set a timer with five separate time periods: 2 seconds, 4s, 5s, 7, and 20s

Even though some of our current computer systems can have huge amounts of memory installed, memory management is still a crucial aspect of Linux systems programming.  Especially with the rise in popularity of embedded devices that rely on Linux as their operating system, such as those in cellular phones, efficient memory management is as important as ever.  Although computer hardware is always improving, we still need to take into consideration the way we handle our limited memory resources, in order to decrease the amount of memory fragmentation that can otherwise occur.   
A system kernel can waste an enormous amount of processor cycles allocating, initializing, and freeing objects on order to both (1) keep them in a usable state, and (2) allow for the possibility of running a program that is larger than physical memory.   This waste of cycles can be neatly minimized by caching the structure of more frequently used objects so that they can be used without the need for freeing and re-allocating:  saving precious processor cycles and allowing for a more efficient memory management subsystem.  For this final incremental addition to our kernel timer project, we added a circular linked-list kernel data object, as a slab allocator.  
 A slab allocator is a method of pre-allotting a doubly-linked circular linked list of structures in memory, for the purpose of efficiently maintaining the basic structure of those objects in a constantly initialized state; bypassing the need to free and re-initialize a structure that will be re-used often. This efficient memory caching routine eliminates the fragmentation caused by the deprecated heap-style memory allocation routines which required careful memory reclamation routines in order to deal with the fragmentation caused by the multiple allocations and de-allocations of the multitudes of disparate object types that are constantly being swapped into and out of memory. 
The slab cache structure is created using: 
static struct kmem_cache *my_cachep; 
 
 
 
 
The user space component of this program loops for 300 seconds, printing the time every second.  As alarm signals are propagated by the kernel, my timer callback function gets called: 
void my_timer_callback(unsigned long data) // KERNEL SPACE TIMER FUNCTION CALL 
{ 
intret; 
printk("kobject_tkp_myClock2: my_timer_callback called, send signal to pid %d \n", t[num_alarms-1]-> pid); 
memset(&info,0,sizeof(struct siginfo));//clear the struct 
info.si_signo=SIG_TEST;// set the signal number to our specified value:  44 
info.si_code=SI_QUEUE; 
info.si_int=alm_array[--num_alarms];//both decrease #/alarms and return that alarms TIME  
ret=send_sig_info(SIG_TEST,&info,t[num_alarms]); 
if(ret < 0) { 
printk("kobject_tkp_myClock2: error sending signal\n"); 
} 
} 
This function sends the siginfo struct (struct siginfo info) to the user space.  The user space can analyze the struct member contents and act accordingly:  in this case outputting the amount of time that the alarm had been set for, which is the value stored in the si_int member.








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
