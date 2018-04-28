cron jobs
 - cleanup logs folder
 - cleanup .gradle bloat

daemonize website

Fix java-tron daemon to exit cleanly

separate mail & web server

Add dynamic loading content to website so the equivalent of this command works
tail /home/jasonneely/java-tron/logs/tron.log -F | grep "ERROR" >> /var/www/web/source/templates/homepage.jade &

Create system status webpage for above command
make it a daemon so it respawns after log rotation (currently gives file not found error)

Firewall or proxy to deny http, etc., requests to port 50051 / 18888