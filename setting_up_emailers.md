We’ll gather in this page various configuration tips and tricks for having emails work on various systems.

### Debian
[todo]

### Enterprise Linux
[todo]

### FreeBSD
author Contributed

**Introduction**:

This tutorial specifically shows how to configure a FreeBSD system to use a Gmail account for its sendmail calls. It replaces the default sendmail client with sSMTP.

With that said, the settings used for Gmail can easily be replaced for any other email service, including your own.

 

**Update FreeBSD Ports Tree**:

`portsnap fetch update`

 

**Disable Default Sendmail**:

You can disable Sendmail by adding the following lines to the file /etc/rc.conf:

    sendmail_enable="NO"
    sendmail_submit_enable="NO"
    sendmail_outbound_enable="NO"
    sendmail_msp_queue_enable="NO"

After this you can reboot or you can kill sendmail manually with the following command:

`killall sendmail`

 

**Install sSMTP port**:

`cd /usr/ports/mail/ssmtp/ && make install`

 

**Configure sSMTP**:

Next, you need to create a configuration file for sSMTP:

`ee /usr/local/etc/ssmtp/ssmtp.conf`

Add the following:

    root=<your_email>
    mailhub=smtp.gmail.com:587
    AuthUser=<your_email>
    AuthPass=<your_password>
    UseSTARTTLS=YES

Where:

<your_email> – is a real email on Gmail. Example: test@gmail.com.

<your_password> – is a password for the email account specified for test@gmail.com.

 

**Test sSMTP**:

Create a txt file “message.txt” with the following text:

    To: yourmail@gmail.com
    From: yourmail@gmail.com
    Subject: Testmessage
    This is a test for sending

Where yourmail@gmail.com is your current or second email to get the test email. Run the following command:

`ssmtp -v yourmail@gmail.com < message.txt`

After this you will get a text output confirming your mail was sent. If there were no errors, then you can verify it worked by checking yourmail@gmail.com

If you get errors however, double-check the sSMTP config.

 

**Replace sendmail with sSMTP**:

This makes sSMTP the default mailer for the system (i.e. replaces all calls for sendmail).

`mv /usr/sbin/sendmail /usr/sbin/sendmail.org`

`ln -s /usr/local/sbin/ssmtp /usr/sbin/sendmail`

Make sure that mail is working by using the following command:

`mail -v -s "test subject" yourmail@gmail.com`

 

**To change the name in which e-mail comes from**:

`chpasswd www`

The chpasswd tool uses vi as its text editor. If you don’t know how to use it, you can visit [Colorado State’s reference sheet](http://www.cs.colostate.edu/helpdocs/vi.html).

Once the file is opened, find the user’s full name, it will look something like this:

    Full Name: World Wide Web Owner

Change the full name::

    Pydio From Example.com
 

**On Pydio**:

Use the default sendmail setup and it should work as expected.