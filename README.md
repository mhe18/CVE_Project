# CVE_Project
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
