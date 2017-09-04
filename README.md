nagios-html-email
=================

A script to send html email notifications from Nagios.

![example-notification](https://github.com/vazudevan/nagios-html-email/raw/master/example-images/example-notification.png)

License: MIT
------------
Copyright (c) 2012 Jason Hancock <jsnbyh@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is furnished
to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

Overview:
---------

This plugin, sends out nagios notifications.  In a large setup with distribured
servers, and multiple administrators/teams, this plugin helps in indentifying
the soure nagios server from where the alert was received.  Also it filters out
communication errors; thereby reducing false positives.

Installation
------------

Copy the notify-html-email script from the plugins directory and drop it into 
Nagios' plugins directory (this is usually /usr/lib64/nagios/plugins on a 
64-bit RHEL/CentOS box). Then configure Nagios.

Nagios Configuration
--------------------

This plugin relies on arguments been passed

Create these command objects, you can have them in notification.cfg :

```
define command {
        command_name    inspect-and-service-notify-html
        command_line    /usr/bin/perl
/usr/lib64/nagios/plugins/nagios-html-email \
        --nagioshost http://IP.ADD.RE.SS/nagios \
        --param LOCATION="Bangalore Nagios" \
        --param CENTRAL="http://IP.ADD.RE.SS/mntos/" \
        --param ADMIN="$ADMINEMAIL$" \
        --param NOTIFICATIONTYPE="$NOTIFICATIONTYPE$" \
        --param HOSTNAME="$HOSTNAME$" \
        --param HOSTSTATE="$HOSTSTATE$" \
        --param HOSTADDRESS="$HOSTADDRESS$" \
        --param HOSTGROUPNAMES="$HOSTGROUPNAMES$" \
        --param HOSTNOTES="$HOSTNOTES$" \
        --param HOSTALIAS="$HOSTALIAS$" \
        --param SERVICEDESC="$SERVICEDESC$" \
        --param SERVICEOUTPUT="$SERVICEOUTPUT$" \
        --param SERVICESTATE="$SERVICESTATE$" \
        --param SERVICENOTES="$SERVICENOTES$" \
        --param SERVICEATTEMPT="$SERVICEATTEMPT$" \
        --param SERVICECHECKTYPE="$SERVICECHECKTYPE$" \
        --param SHORTDATETIME="$SHORTDATETIME$" \
        --param CONTACTEMAIL="$CONTACTEMAIL$" \
        --param NOTIFICATIONRECIPIENTS="$NOTIFICATIONRECIPIENTS$" \

}

define command {
        command_name    inspect-and-host-notify-html
        command_line    /usr/bin/perl
/usr/lib64/nagios/plugins/nagios-html-email \
        --nagioshost http://10.230.50.130/nagios \
        --param LOCATION="Alpharetta Nagios" \
        --param CENTRAL="http://10.211.50.160/mntos/" \
        --param ADMIN="$ADMINEMAIL$" \
        --param NOTIFICATIONTYPE="$NOTIFICATIONTYPE$" \
        --param HOSTNAME="$HOSTNAME$" \
        --param HOSTSTATE="$HOSTSTATE$" \
        --param HOSTADDRESS="$HOSTADDRESS$" \
        --param HOSTGROUPNAMES="$HOSTGROUPNAMES$" \
        --param HOSTNOTES="$HOSTNOTES$" \
        --param HOSTALIAS="$HOSTALIAS$" \
        --param HOSTCHECKTYPE="$HOSTCHECKTYPE$" \
        --param SHORTDATETIME="$SHORTDATETIME$" \
        --param CONTACTEMAIL="$CONTACTEMAIL$" \
        --param NOTIFICATIONRECIPIENTS="$NOTIFICATIONRECIPIENTS$" \

}

```

Notice a couple of things with these command objects. First, I had to specify 
the path to the perl binary. This was because Nagios will use the embedded perl
interpreter and I didn't want that to happen. Second, I'm passing few urls
of my nagios instance to the script as arguments 'nagioshost', 'central' . The 
url is only used to build links that get put into the body of the email. This 
is so you can click on the links directly from the email and get taken to your 
Nagios installation.  The argument 'location' is used in the from address of
email, which quickly indicates the location where alert has originated from.

Now that the command object has been configured, you must tell your contacts to
use this new command. I have a generic-contact template that all of my individual
contacts inherit from, thus I only have to update the template to use the new
command:

```
define contact{
    name                            generic-contact 
    register                        0
    ...
    service_notification_commands   inspect-and-service-notify-html
    host_notification_commands      inspect-and-host-notify-html
}

```

Restart Nagios to pick up the changes.

Filtering out false positives:
------------------------------
A few errors, that might not be relavant to recipients except for Nagios admin
can be added as part of the exceptions list in the plugin.  Any alert
containing these keywords will be sent to Nagios admin, instead of the actual
recipient.

```
my @exceptions = (
        'CHECK_NRPE', 
        'check timed out', 
        'PLUGIN TIMEOUT', 
        'Socket timeout', 
        'stdout', 
        'stderr', 
        'out of bounds'
        );
```

Changes from Original:
----------------------
This plugin is forked from https://github.com/jasonhancock/nagios-html-email . It
has been modified to use arguments instead of environments variables.
Enabiling environment varialbes, as per nagios documentation are expensive on 
memory especially on large installations.

A few exceptions keywords have been added. Under certain error conditions the 
notifications meeting criteria will get redirected to nagios admin instead of 
actual recipients.  Mostly communication errors that might be temporary, or 
configuration errors that needs to be fixed by nagios admin.

For use in sites with multi site nagios, URLs and location detailing have been
included.
