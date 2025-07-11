First cd to the installer location

![Pasted image 20250710160428.png](Pasted%20image%2020250710160428.png)

change to the root user

![Pasted image 20250710160457.png](Pasted%20image%2020250710160457.png)

In this directory, run the command `tar xvzf splunk_installer.tgz` to uncompress Splunk

run `mv splunk /opt/` to move the folder to the directory we want to work within, and cd over to the directory `/opt/splunk/bin`

Now I run Splunk and create a user account since it is the first time running

![Pasted image 20250710161215.png](Pasted%20image%2020250710161215.png)

user:coffely
pass:coffely1

![Pasted image 20250710161337.png](Pasted%20image%2020250710161337.png)

We can find helpful documentation for using splunk via the CLI now by running `./splunk help`

![Pasted image 20250710161833.png](Pasted%20image%2020250710161833.png)

To install the Linux Forwarder for Splunk, we first exit the root user so we can go to the folder where we have downloaded the installer.

![Pasted image 20250710162224.png](Pasted%20image%2020250710162224.png)

and then we sudo back into root and unpack/install splunk

![Pasted image 20250710162407.png](Pasted%20image%2020250710162407.png)

again, we move the folder into /opt/ with `mv splunkforwarder /opt/`

we run the splunkforwarder and create a user account again. The default mgmt port is already bound, so we set a new one in the first startup process.

![Pasted image 20250710162730.png](Pasted%20image%2020250710162730.png)

we go login to the splunk interface in our browser using the url and port number in the url bar `coffely:8000`

![Pasted image 20250710162913.png](Pasted%20image%2020250710162913.png)

## Adding a receiving server and Linux_host index

We now enter the Forwarding and receiving settings

![Pasted image 20250710163055.png](Pasted%20image%2020250710163055.png)

Click to add new on under "Configure receiving"

![Pasted image 20250710163327.png](Pasted%20image%2020250710163327.png)

Tell it to listen on port 9997

![Pasted image 20250710163344.png](Pasted%20image%2020250710163344.png)

We now see the created listener, and see that it is enabled.

![Pasted image 20250710163532.png](Pasted%20image%2020250710163532.png)

Now, go to the indexes settings

![Pasted image 20250710163609.png](Pasted%20image%2020250710163609.png)

Create a new index

![Pasted image 20250710163735.png](Pasted%20image%2020250710163735.png)

In this instance, we named it Linux_host

![Pasted image 20250710163829.png](Pasted%20image%2020250710163829.png)

## Adding forwarding server, and monitoring syslog

We go back to the terminal, cd to the /bin/ directory, and this this command to add the forwarder server that listens on port 9997
`./splunk add forward-server 10.10.254.109:9997`

![Pasted image 20250710164010.png](Pasted%20image%2020250710164010.png)

We enter out username and password and the forwarding is added

![Pasted image 20250710164443.png](Pasted%20image%2020250710164443.png)

Now, to setup which log files to monitor, we can tell splunkforwarder to monitor specific files, and send the data to our newly created Linux_host index
Important log files in Linux is store in `/var/log` and we can run the following command to tell splunkforwarder to monitor `syslog` and send the data to ` Linux_host`:
`./splunk add monitor /var/log/syslog -index Linux_host`

![Pasted image 20250710164915.png](Pasted%20image%2020250710164915.png)

We can enter the following directory to find out input configuration file

![Pasted image 20250710165036.png](Pasted%20image%2020250710165036.png)

Reading it with `cat` we see that everything seems to be properly configured

![Pasted image 20250710165134.png](Pasted%20image%2020250710165134.png)

To properly test that we have set everything so far up properly, we can utilize the command-line tool Logger to create a test log added to the syslog, which if everything works properly, we will be able to see in the splunk Dashboard.
`logger "coffely-is-being-logged-properly"`

![Pasted image 20250710165629.png](Pasted%20image%2020250710165629.png)

This creates the log, but we can also check the syslog manually to make sure it was added by running the following command: `tail -1 /var/log/syslog`

![Pasted image 20250710165748.png](Pasted%20image%2020250710165748.png)

Now let's see if it shows up in Splunk

![Pasted image 20250710165905.png](Pasted%20image%2020250710165905.png)

Everything works as it should.

Now we move on to adding another index `Linux_logs` and forwarding `/var/log/auth.log` to this new index.
We can do this following the same exact steps we have outlined.

## Ingesting auth.log into Linux_logs index

Again, go to the indexes settings

![Pasted image 20250710163609.png](Pasted%20image%2020250710163609.png)

I give it the name Linux_logs and it is added and enabled

![Pasted image 20250710170650.png](Pasted%20image%2020250710170650.png)

We cd back into `/opt/splunkforwarder/bin`  and run the command: `./splunk add monitor /var/log/auth.log -index Linux_logs`

![Pasted image 20250710170818.png](Pasted%20image%2020250710170818.png)

## TryHackMe tasks using our new forwarding setup

So, now I check Splunk to see if I can get the answer I am looking for.
The task from TryHackMe is: 
	Follow the same steps and ingest `/var/log/auth.log` file into Splunk index Linux_logs. What is the value in the sourcetype field?

However, the sourcetype in Linux_logs is showing ` auth-too_small`

![Pasted image 20250710172428.png](Pasted%20image%2020250710172428.png)

This is not the correct answer the TryhackMe lab is looking for.
I believe this to be happening because there are not enough valid events in the auth.log file, and Splunk is flagging the file as too small.

I will generate some events to `auth.log` to see if this remedies it.
First I try to do `sudo ls` as this adds events to the `auth log`

![Pasted image 20250710173329.png](Pasted%20image%2020250710173329.png)

They do show up, but still the sourcetype shows as too small

![Pasted image 20250710173409.png](Pasted%20image%2020250710173409.png)

I go restart splunk forwarder, to see if that does anything

![Pasted image 20250710173454.png](Pasted%20image%2020250710173454.png)

After this, I add a new user to make an auth event, just in case

![Pasted image 20250710173528.png](Pasted%20image%2020250710173528.png)

The sourcetype now shows as `syslog` which turns out to be the correct answer

![Pasted image 20250710173608.png](Pasted%20image%2020250710173608.png)

Maybe restarting splunkforwarder forced Splunk to re-evaluate the file and now that there were events in the file, it started logging them properly, I am not sure, but it worked in the end.

The next task asks how many events are returned as a result of user creation. While I count 7, it seems the correct answer it wants is 6. I can surmise that since the `gkr-pam` event is not guaranteed to occur on every system, it should not be counted, and as such, the valid events are these:

![Pasted image 20250710174200.png](Pasted%20image%2020250710174200.png)

- 1:`groupadd`: added to `/etc/group`
    
- 2:`groupadd`: added to `/etc/gshadow`
    
- 3:`groupadd`: new group created
    
- 4:`useradd`: new user created
    
- 5:`passwd`: password changed
    
- 6:`chfn`: user info changed

And that concludes the installation and demonstration of Splunk and Splunk Forwarder on Linux!


## Setting Splunk up on Windows

Now for installing Splunk on Windows.
First we go to the official splunk website and download an installer compatible with our Windows OS version.
We download the Splunk Enterprise instance, and confirm that it is compatible with our Windows version

![Pasted image 20250711131518.png](Pasted%20image%2020250711131518.png)

To install, we simply navigate to our specified download folder, and run the splunk-instance installer.

We accept the license agreement and press next.
You can change options here as well, such as default port, but there is no need to do so now.

![Pasted image 20250711132056.png|300](Pasted%20image%2020250711132056.png|300)

We create an admin account and press next

![Pasted image 20250711132129.png|300](Pasted%20image%2020250711132129.png|300)

Then we press `Install`
We wait for the progress bar to finish

![Pasted image 20250711132255.png|300](Pasted%20image%2020250711132255.png|300)

When finished, we will receive a message saying the installation is complete.
You can leave the box checked to launch a browser with the Splunk instance to access the login page directly

![Pasted image 20250711133022.png|300](Pasted%20image%2020250711133022.png|300)

Here, you can enter the username and password you created during installation

![Pasted image 20250711133106.png|300](Pasted%20image%2020250711133106.png|300)

If successfully entered, we will land at the dashboard

![Pasted image 20250711133156.png](Pasted%20image%2020250711133156.png)

Here you can explore the options available to you with Splunk, such as adding data:

![Pasted image 20250711133352.png](Pasted%20image%2020250711133352.png)

Accessing this menu we would be given three options for methods of adding data, let's try clicking Monitor, for example

![Pasted image 20250711133438.png](Pasted%20image%2020250711133438.png)

Clicking any option on the following screen would guide us through the process of adding data monitoring, but first, we will do something else.

![Pasted image 20250711133603.png](Pasted%20image%2020250711133603.png)

## Configuring a receiver on Splunk Windows

Press the top left corner logo to go back to the dashboard

![Pasted image 20250711133730.png](Pasted%20image%2020250711133730.png)

Then press Settings, and then Forwarding and receiving, same as on Linux. Using the GUI, there is no difference.

![Pasted image 20250711133815.png](Pasted%20image%2020250711133815.png)

Same as on Linux, press Add new

![Pasted image 20250710163327.png](Pasted%20image%2020250710163327.png)

Again, listen on default port 9997

![Pasted image 20250710163344.png](Pasted%20image%2020250710163344.png)

We will be shown a list of listeners, and see that is it enabled.

![Pasted image 20250711134113.png](Pasted%20image%2020250711134113.png)

## Installing Splunk Forwarder on Windows

For the Splunk Forwarder, we go to the splunk webpages and download the official installer for our OS version, same as before
Find it in the folder you downloaded it to, and run the installer for the forwarder
We aaccept the license agreement, and for the Forwarder, choose to install as an on-premises instance, as this is the case for our setup

![Pasted image 20250711134424.png|300](Pasted%20image%2020250711134424.png|300)

Uncheck "generate random password" and enter your wanted credentials

![Pasted image 20250711134532.png|300](Pasted%20image%2020250711134532.png|300)

This next step is optional if you are not planning on installing the forwarder on multiple hosts, but we can enter localhost here and the default port 8089

![Pasted image 20250711134626.png|300](Pasted%20image%2020250711134626.png|300)

Give the installer the information for the server here. In this case, we will use localhost and default port number 9997, as specified earlier

![Pasted image 20250711134852.png|300](Pasted%20image%2020250711134852.png|300)

When finished, we will be met with this screen, where you can directly find more info on forwarding if you want
Splunk Universal Forwarder will not be installed in `C:\Program Files\SplunkUniversalForwarder`

![Pasted image 20250711135103.png|300](Pasted%20image%2020250711135103.png|300)

In the Splunk instance in the browser, we can now go see our host details.

![Pasted image 20250711135319.png](Pasted%20image%2020250711135319.png)

If installed correctly, the host details will show up here after Splunk notices it

![Pasted image 20250711140639.png](Pasted%20image%2020250711140639.png)

## Selecting the forwarder in Windows
Go to settings and add data

![Pasted image 20250711140740.png](Pasted%20image%2020250711140740.png)

Choose the Forward option

![Pasted image 20250711140810.png](Pasted%20image%2020250711140810.png)

Click on the host on the left box to add it to the right box, and add a server class name, and press next

![Pasted image 20250711141032.png](Pasted%20image%2020250711141032.png)

From Local Event Logs, out of these 5 local event logs, let's add Application, Security, and System and go next

![Pasted image 20250711141215.png](Pasted%20image%2020250711141215.png)

Create a new index

![Pasted image 20250711141354.png](Pasted%20image%2020250711141354.png)

Name it win_logs and press Save

![Pasted image 20250711141321.png](Pasted%20image%2020250711141321.png)

Select it from the dropdown menu

![Pasted image 20250711141426.png](Pasted%20image%2020250711141426.png)

Press "Review" to move on to the next step

The summary should now look like this

![Pasted image 20250711141516.png](Pasted%20image%2020250711141516.png)

Press "Submit" to finish setting this up
Click "Start Searching" to go to the Search app

![Pasted image 20250711141610.png](Pasted%20image%2020250711141610.png)

It will take us to the Search app and already have the search filtered according to the setup we just finished.
`source="WinEventLog:*" index="win_logs"`
We can see everything showing up here, and the three sourcetypes we specified are showing up properly

![Pasted image 20250711141836.png](Pasted%20image%2020250711141836.png)

Now we can look around and search for events, for example events with the EventCode `4624`, which will show up all events where someone successfully logged on.

![Pasted image 20250711142151.png](Pasted%20image%2020250711142151.png)

## Adding a weblog source

Again, go to Settings, and Add Data

![Pasted image 20250711142338.png](Pasted%20image%2020250711142338.png)

Select "Forward"
Name it `web_logs` this time

![Pasted image 20250711142458.png](Pasted%20image%2020250711142458.png)

The web log files are stored in `C:\inetpub\logs\LogFiles\W3SVC*,` where the * represents a number.
To find out in which number and directory the web logs are stored on our host, open your File Explorer, and enter `C:\inetpub\logs\LogFiles\W3SVC` in the Path Display bar

![Pasted image 20250711142727.png](Pasted%20image%2020250711142727.png)

The File Explorer will auto-complete and suggest a directory matching what we have typed out so far.
For us, the web_logs are stored in `C:\inetpub\logs\LogFiles\W3SVC1`, so enter that into the File or Directory field

![Pasted image 20250711143032.png](Pasted%20image%2020250711143032.png)

the source type we are looking for is an IIS server, so choose Select, Select Source Type, and type in `iis` to find the option we want

![Pasted image 20250711143223.png](Pasted%20image%2020250711143223.png)

You can also create a new `web_logs` index here and choose it from the dropdown menu

![Pasted image 20250711150357.png](Pasted%20image%2020250711150357.png)

on the next screen, we will see this summary

![Pasted image 20250711150418.png](Pasted%20image%2020250711150418.png)

You can press "Start Searching" and Splunk will take us to the search app with a relevant filter, just like before.
Now we will now go to the coffely.thm website to generate some logs by ordering some coffee, and wait for them to propagate, so we can see if everything works as it should.

![Pasted image 20250711143526.png](Pasted%20image%2020250711143526.png)

After waiting a while, we also see that new events are showing up filtering by our newly created index

![Pasted image 20250711153852.png](Pasted%20image%2020250711153852.png)

There is also a not-so-secret flag somewhere in `http://coffely.thm/secret-flag.html`

![Pasted image 20250711154147.png](Pasted%20image%2020250711154147.png)

The flag is  is `{COffely_Is_Best_iN_TOwn}`


