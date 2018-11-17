# RasLIRC Troubleshooting

### Logs
  * If the Ansible playbook was ran, it configures LIRC to log to /var/log/lircd .  It will log all errors and connections.  If you're having problems, keep an SSH window open to the Pi and run "tail -f /var/log/lircd".  This will keep the log scrolling on the screen with whatever pops up in it.
---
### LIRC IR Sending Checks 
  * `irsend list "" ""` will list all IR databases.
  * `irsend list remote ""` will list all codes for the database named "remote".  Replace "remote" with whatever database is listed that you want to look into.
  * `irsend list remote key` will confirm that LIRC can pull out the one key.  Replace "key" with whatever command is in the right column of the database you want to check.
  * To test remotely (ie, from another machine), run `irsend -a 1.2.3.4:8765 list "" ""`.  Replace "1.2.3.4" with whatever the IP address of the target device is.  8765 is the port configured for LIRC to listen on, based on the Ansible playbook.  If you've changed the port manually, then change it in this command.


