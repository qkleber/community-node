cron jobs
 - cleanup logs folder
 - cleanup .gradle bloat

daemonize website
daemonize java-tron
separate mail & web server

Add dynamic loading content to website so the equivalent of this command works
tail /root/java-tron/logs/tron.log -F | grep "ERROR" >> /var/www/web/source/templates/homepage.jade &

Create system status webpage for above command
make it a daemon so it respawns after log rotation (currently gives file not found error)

