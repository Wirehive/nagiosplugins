Wirehive Nagios Plugins
=======================
This repository is a set of Nagios plugins that have been either built from scratch or improved by Wirehive. At Wirehive we make heavy use of the Nagios Core product in the core of our monitoring platform. This by no means represents the entirety of the plugins we use, however we will release mroe as the become tidy and fit for consumption :)


check_memcached
---------------
Sets and retrieves a value from a memcached instance to test both availability and responsiveness. Requires netcat to be installed.

```
define command{
        command_name check_memcached
        command_line $USER1$/check_memcached -H $HOSTADDRESS$
}
```


check_domain
------------
Check the expiration on a domain name. We use this as the check_command with a special host template specifically for domains.

```
define command{
        command_name check_domain
        command_line $USER1$/check_domain -d $HOSTADDRESS$
}
```
```
define host{
        name                            domain
        use                             generic-host
        check_period                    24x7
        check_interval                  10   #Check every 10 minutes
        retry_interval                  1
        max_check_attempts              3
        check_command                   check_domain
        notification_period             24x7
        notification_interval           240
        notification_options            d,r
        contact_groups                  admins
        register                        0
}
```

send_android
------------
This is used in conjunction with the Notify My Android app (http://www.notifymyandroid.com/). You will need a developer key from them. Example given of use on a contact.
```
define command{
        command_name notify-host-by-android
        command_line $USER1$/send_android -d $ARG1$ -f "Wirehive Monitoring" -H "$HOSTNAME$" -S $HOSTSTATE$ -p 2
}

define command{
        command_name notify-service-by-android
        command_line $USER1$/send_android -d $ARG1$ -f "Wirehive Monitoring" -s $SERVICESTATE$ -H "$HOSTNAME$" -D $SERVICEDESC$ -p 2
}
```
```
define contact{
        contact_name                    simonANDROID
        use                             generic-contact
        alias                           Simon Green NMA
        service_notification_options    c
        host_notification_options       d,u
        service_notification_commands   notify-service-by-android!USERS-NMA-KEY
        host_notification_commands      notify-host-by-android!USERS-NMA-KEY
}
```

send_irc
--------
Used in conjunction with irccat (https://github.com/Wirehive/irccat) to send a message to an IRC channel. We add on the unusual flapping and maintenance window notifications to the contact as well.
```
define command{
        command_name    notify-host-by-irc
        command_line    $USER1$/send_irc -H "$HOSTNAME$" -a "$HOSTALIAS$" -S "$HOSTSTATE$" -O "$HOSTOUTPUT$" -t "$NOTIFICATIONTYPE$"
}

define command{
        command_name    notify-service-by-irc
        command_line    $USER1$/send_irc -s "$SERVICESTATE$" -H "$HOSTNAME$" -a "$HOSTALIAS$" -S "$HOSTSTATE$" -D "$SERVICEDESC$" -O "$SERVICEOUTPUT$" -t "$NOTIFICATIONTYPE$"
}
```
```
define contact{
        contact_name                    irc
        alias                           IRC
        service_notification_period     24x7
        host_notification_period        24x7
        service_notification_options    w,u,c,r,f
        host_notification_options       d,u,r,s,f
        service_notification_commands   notify-service-by-irc
        host_notification_commands      notify-host-by-irc
}
```

send_prowl
----------
Sends a push notification via the iOS app Prowl (http://www.prowlapp.com/) for notifying Apple iDevices
```
define command{
        command_name notify-host-by-prowl
        command_line $USER1$/send_prowl -d $ARG1$ -f "Monitoring" -H "$HOSTNAME$" -S $HOSTSTATE$ -P YourProviderKey -p 2
}

define command{
        command_name notify-service-by-prowl
        command_line $USER1$/send_prowl -d $ARG1$ -f "Monitoring" -s $SERVICESTATE$ -H "$HOSTNAME$" -D $SERVICEDESC$ -P YourProviderKey -p 2
}
```
```
define contact{
        contact_name                    simonProwl
        use                             prowl-contact
        alias                           Simon Green Prowl
        service_notification_options    c
        host_notification_options       d,u
        service_notification_commands   notify-service-by-prowl!YourRecipientKey
        host_notification_commands      notify-host-by-prowl!YourRecipientKey
}
```
send_twilio_sms
---------------
Sends an SMS via Twilio. Mobile number should be entered with country code, eg +44123412345678
```
define contact{
        contact_name                    simonSMS
        use                             sms-contact
        alias                           Simon Green SMS
        service_notification_options    c
        host_notification_options       d,u
        service_notification_commands   notify-service-by-sms!+441234123456
        host_notification_commands      notify-host-by-sms!+441234123456
}
```
```
define command{
        command_name notify-host-by-sms
        command_line $USER1$/send_twilio_sms -d $ARG1$ -H $HOSTNAME$ -S $HOSTSTATE$ -O "$HOSTOUTPUT$"
}
define command{
        command_name notify-service-by-sms
        command_line $USER1$/send_twilio_sms -d $ARG1$ -s $SERVICESTATE$ -H "$HOSTNAME$" -D $SERVICEDESC$ -O "$SERVICEOUTPUT$"
}
```

send_twilio_phone
-----------------
Sends a text to speech notification to a PSTN phone number via Twilio.

1. Create a `/phonealerts` web route on your Nagios server for Twilio to hit with a callback.

    ```sh
    $ mkdir /var/www/phonealerts
    ```

2. Update the base settings in the `send_twilio_phone` script. Set `CALLBACKURL` to the new `phonealerts` route on your Nagios server.

    ```sh
    CALLBACKURL="http://nagios.acme.com/phonealerts"
    ```

3. Set `CALLBACKPATH` to the DocumentRoot location of your `phonealerts` route.

    ```sh
    CALLBACKPATH="/var/www/phonealerts"
    ```

4. Set the Twilio Account SID and Auth Token found in your [account settings](https://www.twilio.com/user/account/settings).

    ```sh
    TWIMLSID="TWILIO-ACCOUNT-SID"
    TWIMLTOKEN="TWILIO-AUTH-TOKEN"
    ```

```
define contact{
        contact_name                    simonPhone
        use                             phone-contact
        alias                           Simon Green Phone
        service_notification_options    c
        host_notification_options       d,u
        service_notification_commands   notify-service-by-phone!+441234123456
        host_notification_commands      notify-host-by-phone!+441234123456
}

define command{
        command_name notify-host-by-phone
        command_line $USER1$/send_twilio_phone -d $ARG1$ -H $HOSTNAME$ -S $HOSTSTATE$ -O "$HOSTOUTPUT$"
}

define command{
        command_name notify-service-by-phone
        command_line $USER1$/send_twilio_phone -d $ARG1$ -s $SERVICESTATE$ -H "$HOSTNAME$" -D $SERVICEDESC$ -O "$SERVICEOUTPUT$"
}

```

Check SNMP time in UTC
-----------------
Checks remote server time in UTC and compares to local system time in UTC
```
define command{
        command_name snmp_remote_time_utc
        command_line $USER1$/snmp_remote_time_utc -H $HOSTADDRESS$ -C $ARG1$ -w $ARG2$ -c $ARG3$
}

```


