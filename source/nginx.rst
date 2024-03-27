NGINX
=====

Create a new virtual host
-------------------------

Create a new virtual host

.. code-block::
   :caption: /etc/nginx/sites-available/example.com

   server {
      listen 80;
      listen [::]:80;
      server_name example.com;

      return 301 https://$host$request_uri;
   }
   server {
      listen 443 ssl;
      listen [::]:443 ssl;

      if ( $host != "example.com" ) {
         return 444;
      }

      server_name example.com;

      error_log /var/log/nginx/error.log error;
      access_log /var/log/nginx/access.log combined;
      access_log syslog:server=192.168.1.9:12401 graylog_json;

      ssl_certificate /opt/lego/certificates/example.com.pem;
      ssl_certificate_key /opt/lego/certificates/example.com.key;

      add_header Strict-Transport-Security "max-age=31536000; includeSubdomains" always;
      add_header X-Frame-Options "SAMEORIGIN";
      add_header Content-Security-Policy "default-src 'self';";      
      add_header "X-XSS-Protection" "1; mode=block";
      add_header X-Content-Type-Options "nosniff";


      location / {
         proxy_set_header Host $host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-Proto https;
         proxy_set_header X-Forwarded-Host $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_pass http://FORWARD_TO_IP:FOWARD_TO_PORT;
      }
   }


Create the SSL certificates with lego

.. code-block::

   GANDIV5_API_KEY=YOUR_API_KEY lego --dns gandiv5 --email quentin@gireaud.fr -d example.com --pem --path /opt/lego/ run


Create the cron task for renewing the certificates

.. code-block::
   :caption: /var/spool/cron/crontabs/root

   @weekly GANDIV5_API_KEY=YOUR_API_KEY lego --dns gandiv5 --email quentin@gireaud.fr -d example.com --pem --path /opt/lego/ renew && systemctl reload nginx


Reload the nginx service

.. code-block::

   systemctl reload nginx


.. caution:: Dont forget to open corresponding firewall ports and create the DNS records.


Send access logs to syslog server
---------------------------------

First, create the following log_format

.. code-block::
   :caption: /etc/nginx/nginx.conf

   log_format graylog_json escape=json '{ "nginx_timestamp": "$time_iso8601", '
       '"remote_addr": "$remote_addr", '
       '"connection": "$connection", '
       '"connection_requests": $connection_requests, '
       '"pipe": "$pipe", '
       '"body_bytes_sent": $body_bytes_sent, '
       '"request_length": $request_length, '
       '"request_time": $request_time, '
       '"response_status": $status, '
       '"request": "$request", '
       '"request_method": "$request_method", '
       '"host": "$host", '
       '"upstream_cache_status": "$upstream_cache_status", '
       '"upstream_addr": "$upstream_addr", '
       '"http_x_forwarded_for": "$http_x_forwarded_for", '
       '"http_referrer": "$http_referer", '
       '"http_user_agent": "$http_user_agent", '
       '"http_version": "$server_protocol", '
       '"remote_user": "$remote_user", '
       '"http_x_forwarded_proto": "$http_x_forwarded_proto", '
       '"upstream_response_time": "$upstream_response_time", '
       '"nginx_access": true }';


Then, use it in your virtual hosts

.. code-block::
   :caption: /etc/nginx/sites-available/example

   access_log syslog:server=SYSLOG_IP:SYSLOG_PORT graylog_json;



.. tip:: List of available fields `here <https://nginx.org/en/docs/varindex.html>`_.
