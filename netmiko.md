# Running Netmiko

Lab Objective
The objective of this lab is to introduce Netmiko and learn how to write a network automation script. Netmiko is a Python Toolkit that enables simplied SSH access to common network devices. It is in very common use across the Python for Network Automation landscape- therefore, we definitely want to learn how to use it. In addition we also want to learn how to write original code and to augment a program we already have running.

Before you get started, make sure you have:
- Working virtual switches that can be contacted via SSH
- A \*.csv for a source of input.

In this lab we'll roleplay that we are a company that deploys additional networking gear to support major sports events that we sponsor, such as the U.S. Open (a major golf tournament in late August through September). 

### Procedure

0. Make a directory for us to work in.

    `student@beachhead:/$` `mkdir /home/student/mycode/usopen`

0. Move into our new directory.

    `student@beachhead:/$` `cd /home/student/mycode/usopen`

0. We need an \*.csv file to work with. Create a csv file.

    `student@beachhead:~$` `vim ip_list.csv` 

0. Add the following to ip_list.csv:

    ```
    IP,driver
    sw-01,arista_eos
    sw-02,arista_eos
    ```

0. You may already have this installed, but this program depends on having `pyexcel`, `pyexcel-xls`, and `netmiko` installed with pip.

    `student@beachhead:~/mycode/usopen$` `python3 -m pip install pyexcel pyexcel-xls netmiko`

0. Download the script `usopen.py`

    `student@beachhead:~/mycode/usopen$` `wget https://static.alta3.com/files/napya/usopen.py`

0. Display the code:

    `student@beachhead:~/mycode/usopen$` `cat usopen.py`

0. You code should look like the following:

    ```python
    #!/usr/bin/env python3
    ## USOpen Tournament Switch Checker -- 2018.05.01
    ''' usopen.py
    This script is being designed to provide the following automated tasks:
    - Ping check the router (import os)
    - Login check the router (import netmiko)
    - Determine if interfaces in use are up (import netmiko)
    - Apply new configuration (import netmiko) # not yet built
    
    The IPs and device type should be made available via a csv spreadsheet
    
    '''
    import os
    
    ## pyexcel and pyexcel-xls are required for our program to execute
    # python3 -m pip install --user pyexcel
    # python3 -m pip install --user pyexcel-xls
    import pyexcel
    
    # python3 -m pip install --user netmiko
    from netmiko import ConnectHandler
    
    ## retrieve data set from spreadsheet
    def retv_excel(par):
        d = {}
        # create a record object that is an open spreadsheet
        records = pyexcel.iget_records(file_name=par)
        for record in records:
            # adds a new IP and driver key:value pair to our dictionary
            d.update( { record['IP'] : record['driver'] } )
        return d # return the completed dictionary
    
    ## Ping router - returns True or False
    def ping_router(hostname):
    
        response = os.system("ping -c 1 " + hostname)
        
        #and then check the response...
        if response == 0:
            return True
        else:
            return False


    ## Check interfaces - Issue "show ip init brief"
    def interface_check(dev_type, dev_ip, dev_un, dev_pw):
        try:
            open_connection = ConnectHandler(device_type=dev_type, ip=dev_ip, \
              username=dev_un, password=dev_pw)
            my_command = open_connection.send_command("show ip int brief")       
        except:
            my_command = "** ISSUING COMMAND FAILED **"
        finally:
            return my_command


    ## Login to router - SSH Check with Netmiko class ConnectHandler
    def login_router(dev_type, dev_ip, dev_un, dev_pw):
        try:
            open_connection = ConnectHandler(device_type=dev_type, ip=dev_ip, \
              username=dev_un, password=dev_pw)
            return True        
        except:
            return False

    ## Main function - This is the code that runs our functions
    def main():
    
        ## Determine where *.csv input is
        file_location = str(input("\nWhere is the file location? "))
    
        ## Entry is now a local dictionary containing IP(key):driver(value)
        entry = retv_excel(file_location)
    
        ## Use a loop to check each device for SSH accessibility
        print("\n***** BEGIN SSH CHECKING *****")
        for x in entry.keys():
            if login_router(str(entry[x]), x, "admin", "alta3"):
                print("\n\t**IP: - " + x + " - SSH connectivity UP\n")
            else:
                print("\n\t**IP: - " + x + " - SSH connectivity DOWN\n")
           
        ## Use a loop to check each device for ICMP responses
        print("\n***** BEGIN ICMP CHECKING *****")
        for x in entry.keys():
            if ping_router(x):
                print("\n\t**IP: - " + x + " - responding to ICMP\n")
            else:
                print("\n\t**IP: - " + x + " - NOT responding to ICMP\n")
    
        ## Use a loop to check each device for ICMP responses
        print("\n***** BEGIN SHOW IP INT BRIEF *****")
        for x in entry.keys():
            print("\n" + interface_check(str(entry[x]), x, "admin", "alta3"))
    
    ## Call main()
    main()
    ```

0. Run the script.

    `student@beachhead:~/mycode/usopen$` `python3 usopen.py`
    
0. When prompted, type the location of the csv file: `ip_list.csv`

0. The result should look similar to below.

    ![Output produced by usopen.py](https://static.alta3.com/images/python/us_open.png)
    
0. So now we have a working Netmiko script. Let's write a new stand alone function for our program that also uses Netmiko. Our goal is to write a function that we can import into `usopen.py` and then use it within our `main()` function to extend our program.

0. Let's start by describing the new functionality we'd like to build. How about a function that reads config data, line-by-line, from a text document? This will allow users to config the switches with a configuration file they provide. Start by listing out all of the new code we need to write.
    - At the end of the script, user is asked if they want to provide a new configuration file? **YES/NO**
    - If **NO** the script ends.
    - If **YES** the user should be asked where the new configuration file is.
    - If **YES** the user should be asked what IP address they'd like to bootstrap-- the driver should be grabbed if that IP exists.
    - If **YES** the IP address, driver and file location should be passed as an argument to the function `bootstrapper`.
    - If **YES** the new function `bootstrapper` should apply the new running configuration.
    - If **YES** the new function `bootstrapper` should return TRUE when complete.
    - IF **YES** the new function `bootstrapper` should return FALSE if fails.
    - Print to the user if the new configuration application was successful or failed.
    - For simplicity's sake, assume the same static UN/PW we've used so far.

0. Now we have a very clear picture of what we're trying to do. We have some tweaks to our main function and know more or less exactly what the function `bootstrapper` needs to do. In fact, it sounds like we now have a **TWO** person *team* project. Someone could work on `bootstrapper` while someone else made the appropriate tweaks to *main*. Unfortunately, you're going to be doing all the work solo.

0. Let's write `bootstrapper.py`.

    `student@beachhead:~/mycode/usopen$` `vim /home/student/mycode/usopen/bootstrapper.py`

0. The objective should be clear from our description above so you can write it yourself. Or, use the code below. If you use the code below, please review the comments.

    ```python
    from netmiko import ConnectHandler
    ## Define our new function called bootstrapper and the expected arugments (all five)
    def bootstrapper(dev_type, dev_ip, dev_un, dev_pw, config):
        try:
            config_file = open(config, 'r') # open the file object described by config argument
            config_lines = config_file.read().splitlines() # create a list of the file lines without \n
            config_file.close() # close the file object
        
            open_connection = ConnectHandler(device_type=dev_type, ip=dev_ip, \
              username=dev_un, password=dev_pw)
            open_connection.enable() # this sets the connection in enable mode
            # pass the config to the send_config_set() method
            output = open_connection.send_config_set(config_lines)
            print(output) # print the config to the screen # output to the screen
            open_connection.send_command_expect('write memory') # write the memory (okay if this gets done twice)
            open_connection.disconnect() # close the open connection
       
            return True # Everything worked! - "Return TRUE when complete"
        except:
            return False # Something failed during the configuration process - "Return FALSE if fails"
    ```

0. Save and exit.

0. Make the file executable.

    `student@beachhead:~/mycode/usopen$` `chmod u+x bootstrapper.py`

0. Now we need to make some tweaks to `usopen.py` so that our new function is imported and then called.

    `student@beachhead:~/mycode/usopen$` `vim usopen.py`

0. First add the line import bootstrapper. Then add the lines below the comment, `## Determine if new config should be applied`. Add these new lines to your script.

    ```python
    #!/usr/bin/env python3
    ## USOpen Tournament Switch Checker -- 2018.05.01
    ''' usopen.py
    This script is being designed to provide the following automated tasks:
    - Ping check the router (import os)
    - Login check the router (import netmiko)
    - Determine if interfaces in use are up (import netmiko)
    - Apply new configuration (import netmiko)
    
    The IPs and device type should be made available via a spreadsheet
    
    '''
    import os
    
    ## ADD THE LINE BELOW THIS COMMENT
    import bootstrapper
    
    ## pyexcel and pyexce-xls are required for our program to execute
    # python3 -m pip install --user pyexcel
    # python3 -m pip install --user pyexcel-xls
    import pyexcel
    
    # python3 -m pip install --user netmiko
    from netmiko import ConnectHandler
    
    ## retrieve data set from spreadsheet
    def retv_excel(par):
        d = {}
        records = pyexcel.iget_records(file_name=par) # create a record object that is an open spreadsheet
        for record in records:
             # adds a new IP and driver key:value pair to our dictionary
            d.update( { record['IP'] : record['driver'] } )
        return d # return the completed dictionary
    
    ## Ping router - returns True or False
    def ping_router(hostname):
    
        response = os.system("ping -c 1 " + hostname)
        
        #and then check the response...
        if response == 0:
            return True
        else:
            return False
    
    ## Check interfaces - Issue "show ip init brief"
    def interface_check(dev_type, dev_ip, dev_un, dev_pw):
        try:
            open_connection = ConnectHandler(device_type=dev_type, ip=dev_ip, \
              username=dev_un, password=dev_pw)
            my_command = open_connection.send_command("show ip int brief")       
        except:
            my_command = "** ISSUING COMMAND FAILED **"
        finally:
            return my_command
    
    
    ## Login to router - SSH Check with Netmiko class ConnectHandler
    def login_router(dev_type, dev_ip, dev_un, dev_pw):
        try:
            open_connection = ConnectHandler(device_type=dev_type, ip=dev_ip, \
              username=dev_un, password=dev_pw)
            return True        
        except:
            return False

    ## Main function - This is the code that runs our functions
    def main():
    
        ## Determine where *.csv input is
        file_location = str(input("\nWhere is the file location? "))
    
        ## Entry is now a local dictionary containing IP(key):driver(value)
        entry = retv_excel(file_location)
    
        ## Use a loop to check each device for SSH accessibility
        print("\n***** BEGIN SSH CHECKING *****")
        for x in entry.keys():
            if login_router(str(entry[x]), x, "admin", "alta3"):
                print("\n\t**IP: - " + x + " - SSH connectivity UP\n")
            else:
                print("\n\t**IP: - " + x + " - SSH connectivity DOWN\n")
    
        ## Use a loop to check each device for ICMP responses
        print("\n***** BEGIN ICMP CHECKING *****")
        for x in entry.keys():
            if ping_router(x):
                print("\n\t**IP: - " + x + " - responding to ICMP\n")
            else:
                print("\n\t**IP: - " + x + " - NOT responding to ICMP\n")
    
        ## Use a loop to check each device for ICMP responses
        print("\n***** BEGIN SHOW IP INT BRIEF *****")
        for x in entry.keys():
            print("\n" + interface_check(str(entry[x]), x, "admin", "alta3"))
            
        ## Determine if new config should be applied && if so apply new config
        print("\n***** NEW BOOTSTRAPPING CHECK *****")
        ynchk = input("\nWould you like to apply a new configuration? y/N ")
        if (ynchk.lower() == "y") or (ynchk.lower() == "yes"):  # if user input yes or y
            conf_loc = str(input("\nWhere is the location of the new config file? "))
            conf_ip = str(input("\nWhat is the IP address of the device to be configured? "))
            
            if bootstrapper.bootstrapper(entry[conf_ip], conf_ip, "admin", "alta3", conf_loc):
                print("\nNew configuration applied!")
            else:
                print("\nProblem in applying new configuration!")
    
    ## Call main()
    main()
    ```

0. Save and exit.

0. Run our augmented program, `usopen.py`. Indicate `ip_list.csv`. We don't yet have a config to apply, but indicate `Yes` (we have a new config to apply) and apply the file location `dragons.here`. Supply the IP address `sw-01`. We can test our ability to catch errors.

0. So our error was caught. No dragon files exist. Download the following a *basic switch 1 config*.

    `student@beachhead:~/mycode/usopen$` `wget http://static.alta3.com/images/python/sw1-clean.conf -O sw-01-clean.conf`

0. Re-run our augmented program, `usopen.py`. Indicate `ip_list.csv`. This time indicate `y` and apply the file location `sw-01-clean.conf`. Supply the IP address `sw-01`.

0. You should get an indication `New switch configuration applied!`. If not, find your error.

0. You can try this lab again if you'd like and this time target switch2 (`sw-02`). If you run the script against switch 2, **be careful not to overwrite switch1 config with switch2 config** or vice-versa!

    `student@beachhead:~/mycode/usopen$` `wget http://static.alta3.com/images/python/sw2-clean.conf -O sw-02-clean.conf`


0. That's it for this lab, but what an introduction to Netmiko and writing original Python network automation scripts! Hopefully it's clear that automating networks with Python can be done via stare-and-compare, but ideally it requires a strong Python skill set. Don't hesitate to try writing your own function and calling it from the `usopen.py` script! If you need help, ask the instructor on ways to get started.

0. If you're tracking your code in GitHub, issue the following commands:
    - `cd ~/mycode`
    - `git add *`
    - `git commit -m "your commit message"`
    - `git push origin main`

<br><br><div align="center">


![Alta3 Logo](https://static.alta3.com/images/Alta3-logo_large.png)

</div>
