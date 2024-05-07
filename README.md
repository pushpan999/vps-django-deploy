sudo ufw allow 8000

python manage.py runserver 0.0.0.0:8000

gunicorn --bind 0.0.0.0:8000 myproject.wsgi

* setup gunicorn service
*  cd /etc/systemd/system
=================
sitename.socket
=================

[Unit]
Description=sitename socket

[Socket]
ListenStream=/run/sitename.sock

[Install]
WantedBy=sockets.target

===============
sitename.service
===============

[Unit]
Description=sitename daemon
Requires=sitename.socket
After=network.target

[Service]
User=pushpan
Group=www-data
WorkingDirectory=/home/pushpan/WorkingDirectory
ExecStart=/home/pushpan/miniconda3/envs/EnvName/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/sitename.sock \
          core.wsgi:application

[Install]
WantedBy=default.target

==========================

sudo systemctl start sitename.socket
sudo systemctl enable sitename.socket

sudo systemctl status sitename.socket

file /run/sitename.sock

sudo systemctl status sitename

curl --unix-socket /run/sitename.sock localhost



=============
if you make changes to sitename.service

sudo systemctl daemon-reload
sudo systemctl restart sitename

============

###### Setup nginx
# cd /etc/nginx/sites-available/


====================
sitename.com
====================

server {
    server_name sitename.com www.sitename.com;
    proxy_read_timeout 1800;
    proxy_connect_timeout 1800;
    proxy_send_timeout 1800;
    send_timeout 1800;
    
    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        alias /home/pushpan/workingdirectory/static/;
    }
    location /media/ {
        alias /home/pushpan/workingdirectory/media/;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/sitename.sock;
    }
}

======================

sudo ln -s /etc/nginx/sites-available/sitename.com /etc/nginx/sites-enabled

sudo nginx -t

sudo systemctl restart nginx

sudo ufw delete allow 8000

sudo ufw allow 'Nginx Full'

###### Setup Letsencrypt ssl

sudo certbot --nginx -d sitename.com -d www.sitename.com
