# CVE_Project
## Preconditions for exploitation:

1.Check the user who is executing this script, if it is not tomcat user, the program will be aborted. 
  
  `id | grep -q tomcat`
  
2.Copy a shell to /tmp file, notice right now this shell currently has low privilege.

  `cp $BACKDORSH $BACKDOORPATH`
  
3.Change the owner of ld.so.preload to tomcat7 using the aforementioned vulnerability. Currently, we do not have /etc/ld.so.preload on SEED, so we need to create this file at the same time. 

4.Wait for the tomcat7 server to restart, and after it happens, we can import the geteuid () function we make into **/etc/ld.so/preload**, and run a sudo command.While sudo is called, the geteuid() is executed, and the privilege of the shell is escalated.
  `sudo --help`
  
5.Remove the function, the preload file, and execute the backdoor root shell. 
## Preparations for exploitation:

1.Install Tomcat 7 on the virtual machine

  `sudo apt-get install tomcat7`
  
2.If the first step is failed,set J**DK_DIRS="/usr/lib/jvm/java-8-oracle"** and add the statements “export JAVA_HOME=/usr/lib/jvm/java-8-oracle” and **“export CATALINA_HOME=/usr/share/tomcat7”**. Then reinstall Tomcat 7

3.Set up an initial password for the Tomcat user

  `sudo passwd tomcat7`
  
4.Assign login shell for the Tomcat user

  `sudo usrmod -s /bin/bash tomcat7`
  
5.Establish the Tomcat 7 user shell

  `ssh tomcat7@localhost`


## Launch the attack:
1.Create the exploit scripts attack.sh at /tmp through Tomcat7 user shell and then make the exploit codes executable
  `touch attack.sh`
  
  `chmod +x attack.sh`
  
2.Run the exploit code

  `./attack.sh `
  
3.Restart the Tomcat server to check the attack result.

  `sudo service tomcat7 restart`
  
4.Check the ID of user
  
  `id`
  
Then the attack should be successful and the euid should be 0 which is root.


## Vulnerability:
	# Run the catalina.sh script as a daemon
	set +e
	touch "$CATALINA_PID" "$CATALINA_BASE"/logs/catalina.out
	chown $TOMCAT7_USER "$CATALINA_PID" "$CATALINA_BASE"/logs/catalina.out
	start-stop-daemon --start -b -u "$TOMCAT7_USER" -g "$TOMCAT7_GROUP" \
		-c "$TOMCAT7_USER" -d "$CATALINA_TMPDIR" -p "$CATALINA_PID" \
		-x /bin/bash -- -c "$AUTHBIND_COMMAND $TOMCAT_SH"
	status="$?"
	set +a -e
	return $status
  The code above is key of esclating privilege. As it runs with root privilege, it can do anything we want. Then we need to change the owner of catalina.out to user of tomcat7. After we reboot the Tomcat, we are able to read the file which catalina points to.
  
  Specifically, when the tomcat restart, it will change the owner of the catalina.out to user of Tomcat, and the restart is invoked by the linux init. By taking advantage of Linux init, we can use symlink to change the owener of any file to the user of tomcat. If we symlink catalina.out to any file, we could operate any file in the system with the privilege root which given by the catalina.out. This is typical exploitation of DLL hijack. 
