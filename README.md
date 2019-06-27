## Dell iDrac fan Management Daemon, Rx20

***

This is a daemon to set the fans in a 12G server to a manual speed when the CPU and ambient temperatures are low, and back to automatic when the load and temperature increases.  

Unlike running from cron, this can sample the temperatures in intervals as low as a second, which can result in faster response time when the temperature rises.  

I have no idea if this script will work properly on other generations of servers.  I know sometimes the IPMI values can change between different generations. 

### Prequisites

* PHP7+, specifically the CLI binary. 
* ipmitool

On Debian/Ubuntu:

    sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
    echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
    sudo apt install php7.3-cli

ipmitool:

    sudo apt install ipmitool

On CentOS/Fedora/RHEL:

https://blog.remirepo.net/post/2018/12/10/Install-PHP-7.3-on-CentOS-RHEL-or-Fedora

ipmitool:

    yum install OpenIPMI ipmitool


***

### Configuration


Put the script in /usr/local/bin (or wherever), and the conf file in /etc.  Both are configurable via the systemd unit and conf file. 

Look in the conf file for options and what needs to be configured.  

Run `ipmitool sdr` or `ipmitool -I lanplus -H idrac_ip -U username -P password sdr`. Identify these keys, and drop into the conf file.

In my output, the important keys are `Temp` (these are CPU temps on my r420), `Fan1a, 2b, etc`, and `Inlet Temp`.

    Fan1A            | 6480 RPM          | ok
    Fan1B            | 6240 RPM          | ok
    Fan2A            | 6480 RPM          | ok
    Fan2B            | 6000 RPM          | ok
    Fan3A            | 6360 RPM          | ok
    Fan3B            | 6000 RPM          | ok
    Fan4A            | 6480 RPM          | ok
    Fan4B            | 6000 RPM          | ok
    Fan5A            | 6480 RPM          | ok
    Fan5B            | 6000 RPM          | ok
    Fan6A            | 6360 RPM          | ok
    Fan6B            | 6120 RPM          | ok
    Inlet Temp       | 26 degrees C      | ok
    Temp             | 44 degrees C      | ok
    Temp             | 40 degrees C      | ok


Add the systemd unit to `/etc/systemd/system`, reload units `sudo systemctl daemon-reload`, `sudo systemctl enable`, `sudo systemctl start`.  If running IPMItool over the network, you can configure the unit to run as a user other than root.  

I tried to make it somewhat resilient to bad configuration options, but ultimately I wrote this in about an hour.  It will be easy to break.  Caveat Emptor. 

