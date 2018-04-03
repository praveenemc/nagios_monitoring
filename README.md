# Monitoring network bandwidth of Virtual Machines hosted in XenServer using Nagios

Nagios Server [Server node which monitors client machines]
**********************************************************

 * OS: Ubuntu 14.04 Desktop.
 * IP: 172.27.3.77
 * Hostname: infics [Ensure you have set proper hostname to the server]

*STEP 1:*

    sudo apt-get update -y

    sudo apt install nagios3 nagios-nrpe-plugin vnstat -y

It prompts for postfix - mail configuraitons. I selected No Configuration.
You will be asked to enter a password for the nagiosadmin user.

You can access the nagios UI in webbrowser using http://172.27.3.77/nagios3
It prompts for user/password ==> nagiosadmin / praveen@123

*STEP 2:*

Rename localhost to hostname of your server

    sudo mv /etc/nagios3/conf.d/localhost_nagios2.cfg \
    /etc/nagios3/conf.d/infics.cfg

NOTE: Here infics is my nagios server hostname so i renamed localhost_nagios2.cfg to infics.cfg file and properly replaced the hostname as shown below:
![alt tag](https://github.com/npraveen35/Xen_API/blob/nagios/infics_cfg_initial.JPG)

*STEP 3:* Replace localhost with * in the /etc/nagios3/conf.d/hostgroups_nagios2.cfg file at line 14,21, 28.

*STEP 4:* Place the script check_bandwidth in /usr/lib/nagios/plugins/ folder

*STEP 5:* You should store the following text by creating a file in /etc/nagios-plugins/config/bandwidth.cfg
![alt tag](https://github.com/npraveen35/Xen_API/blob/nagios/bandwidth_cfg.JPG)

*STEP 6:* Define the CHECK-BANDWIDTH service as show below:

praveen@infics:/etc/nagios3/conf.d$ cat infics.cfg

    # A simple configuration file for monitoring the local host
    # This can serve as an example for configuring other servers;
    # Custom services specific to this host are added here, but services
    # defined in nagios2-common_services.cfg may also apply.
    #
    define host{
            use                     generic-host            ; Name of host template to use
            host_name               infics
            alias                   infics
            address                 172.27.3.77
            }
    # Define a service to check the disk space of the root partition
    # on the local machine.  Warning if < 20% free, critical if
    # < 10% free space on partition.

    define service{
            use                             generic-service         ; Name of service template to use
            host_name                       infics
            service_description             Disk Space
            check_command                   check_all_disks!20%!10%
            }

    # Define a service to check the number of currently logged in
    # users on the local machine.  Warning if > 20 users, critical
    # if > 50 users.
    define service{
            use                             generic-service         ; Name of service template to use
            host_name                       infics
            service_description             Current Users
            check_command                   check_users!20!50
            }

    # Define a service to check the number of currently running procs
    # on the local machine.  Warning if > 250 processes, critical if
    # > 400 processes.
    define service{
            use                             generic-service         ; Name of service template to use
            host_name                       infics
            service_description             Total Processes
            check_command                   check_procs!250!400
            }

    # Define a service to check the load on the local machine.
    define service{
            use                             generic-service         ; Name of service template to use
            host_name                       infics
            service_description             Current Load
            check_command                   check_load!5.0!4.0!3.0!10.0!6.0!4.0
            }
    # PRAVEEN: Define a service to monitor network bandwidth
    # check_bandwidth bw_warning bw_critical pkt_warning pkt_critical
    define service{
            use                     generic-service
            host_name               infics
            service_description     CHECK-BANDWIDTH
            check_command           check_bandwidth!100!500!100!500
            }

*STEP 7:* Restart the nagios daemon to enable the new configuration:

     sudo service nagios3 restart

*STEP 8:* Access through your browser.
![alt tag](https://github.com/npraveen35/Xen_API/blob/nagios/nagios_server.JPG)

To Add Client Nodes to be monitored by Nagios Server:
=====================================================

Create a host configuration file for client node. Run the below commands on Nagios Server host.

*STEP 9:*

    sudo cp /etc/nagios3/conf.d/infics.cfg \
    /etc/nagios3/conf.d/infics-praveen-odl.cfg

*STEP 10:*
Remember to replace infics-praveen-odl with the hostname of your client machine to be monitored.
Here infics-praveen-odl is my client node's hostname.

Next, edit /etc/nagios3/conf.d/infics-praveen-odl.cfg to replace the hostname properly.

*STEP 11:* Restart the nagios daemon to enable the new configuration:

    sudo service nagios3 restart

Nagios Client Machines [host to be monitored]
==============================================
 * OS: Ubuntu 14.04 Desktop.
 * IP:172.27.3.176
 * Hostname: infics-praveen-odl

*STEP 12:*

    sudo apt-get update -y

    sudo apt-get dist-upgrade -y

*STEP 13:* Reboot the node and install the below packages:

    sudo apt install nagios-nrpe-server vnstat -y 

*STEP 14:* Place the shell script check_bandwidth to path /usr/lib/nagios/plugins/

*STEP 15:* Then edit /etc/nagios/nrpe.cfg 

    allowed_hosts=172.27.3.77   # This is my Nagios Server IP

*STEP 16:* Add below in the command definition area of npre.cfg file:

    command[check_bandwidth]=/usr/lib/nagios/plugins/check_bandwidth 100 500 100 500

It resembles as below:

![alt tag](https://github.com/npraveen35/Xen_API/blob/nagios/command_definition_nrpe_cfg.JPG)

*STEP 17:* Finally, restart nagios-nrpe-server on your client machine:

    sudo service nagios-nrpe-server restart

*STEP 18:* After that, check in your browser: [Ignore the CRITICAL message as status for Disk Space which is not my priority as of now]
![alt tag](https://github.com/npraveen35/Xen_API/blob/nagios/services.JPG)

*STEP 19:* To test the bandwidth usage, i started downloading a cloud image on both server & client node as shown below:

![alt tag](https://github.com/npraveen35/Xen_API/blob/nagios/wget_testing_cloud_image.JPG)

*STEP 20:* After approximately 4-5 mins in the browser i was able to observe the CRITICAL status for CHECK-BANDWIDTH as shown below:
![alt tag](https://github.com/npraveen35/Xen_API/blob/nagios/Nagios_and_client_services_critical.JPG)



    YET TO COME: Configuring Email Notifications - CRITICAL / SMS Messages - WARNING

