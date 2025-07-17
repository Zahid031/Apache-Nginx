default location is /var/log/nginx/   access.log error.log

we can custom log to the directory with 

  location /userdata {
                access_log /var/log/nginx/access_user.log;
                return 200 "User data is .....";
        }

tail -f /var/log/nginx/*

we can off log for specific one like 
access_log off


parent child worker proceses
ps -aux --forest | grep nginx
