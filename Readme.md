### Overview 

- nginx load balancer;
- 2 US servers;
- 1 GB server;
- 1 Server for other public IPs; 

### What to test: 
1. Geo IP load balancing: 
    -   Prerequisits: 
        - Expose your private IP to the network with use of ngrok
        ```
            ngrok http 8086
        ```
        - Download VPN chrome extension

    -   Results: 
        - USA vpn: 
        docker container 'load_balancer' logs:
        ```
        2024-09-11 17:40:58 138.199.9.179 - country_code: US
        ```
        ![usa geoip](./images/usa-geoip.png)
        - UK vpn:
        docker container 'load_balancer' logs:
        ```
        2024-09-11 17:46:37 195.181.165.183 - country_code: GB
        ```
        [gb geoip](./images/uk-geoip.png)
        - Default vpn:
        docker container 'load_balancer' logs:
        ```
        2024-09-11 17:49:02 91.203.164.50 - country_code: UA
        ```
        ![world geoip](./images/world-geoip.png)
2. Active helth check: 
    - Stop one of the nginx server 
    ```
    docker compose stop nginx_world
    ```
    docker container 'load_balancer' logs:
    ```
    2024-09-11 18:02:03 2024/09/11 15:02:03 [error] 7#7: *3348 lua tcp socket connect timed out, when connecting to 172.20.0.5:80, context: ngx.timer
    2024-09-11 18:02:03 2024/09/11 15:02:03 [error] 7#7: *3348 [lua] healthcheck.lua:53: errlog(): healthcheck: failed to connect to 172.20.0.5:80: timeout, context: ngx.timer
    2024-09-11 18:02:09 2024/09/11 15:02:09 [error] 7#7: *3365 lua tcp socket connect timed out, when connecting to 172.20.0.5:80, context: ngx.timer
    2024-09-11 18:02:09 2024/09/11 15:02:09 [error] 7#7: *3365 [lua] healthcheck.lua:53: errlog(): healthcheck: failed to connect to 172.20.0.5:80: timeout, context: ngx.timer
    2024-09-11 18:02:15 2024/09/11 15:02:15 [error] 7#7: *3375 lua tcp socket connect timed out, when connecting to 172.20.0.5:80, context: ngx.timer
    2024-09-11 18:02:15 2024/09/11 15:02:15 [error] 7#7: *3375 [lua] healthcheck.lua:53: errlog(): healthcheck: failed to connect to 172.20.0.5:80: timeout, context: ngx.timer
    2024-09-11 18:02:21 2024/09/11 15:02:21 [error] 7#7: *3385 lua tcp socket connect timed out, when connecting to 172.20.0.5:80, context: ngx.timer
    2024-09-11 18:02:27 2024/09/11 15:02:27 [error] 7#7: *3395 lua tcp socket connect timed out, when connecting to 172.20.0.5:80, context: ngx.timer
    ```
    Based on the logs our load balancer got 3 consequitive error responses from our world_nginx container with static Ip address: 172.20.0.5, and as a result, on next user request, our backup server will automatically handle request, and user won't get 500 error. 
    ![backup response](./images/backup.png)
    - Start up failed server
    ```
    docker compose up nginx_world -d
    ```
    nginx_world docker container logs: 
    ```
    2024-09-11 18:10:34 172.20.0.7 - - [11/Sep/2024:15:10:34 +0000] "GET / HTTP/1.1" 200 236 "-" "-" "-"
    2024-09-11 18:10:39 172.20.0.7 - - [11/Sep/2024:15:10:39 +0000] "GET / HTTP/1.1" 200 236 "-" "-" "-"
    2024-09-11 18:10:44 172.20.0.7 - - [11/Sep/2024:15:10:44 +0000] "GET / HTTP/1.1" 200 236 "-" "-" "-"
    2024-09-11 18:10:49 172.20.0.7 - - [11/Sep/2024:15:10:49 +0000] "GET / HTTP/1.1" 200 236 "-" "-" "-"
    ```
    Based on the logs we've got more then two consequetive 200 response so it means our 172.20.0.5 is in helth status and next user request will be transfered to 'nginx_world' server.
    ```
    2024-09-11 18:17:55 172.20.0.7 - - [11/Sep/2024:15:17:55 +0000] "GET / HTTP/1.1" 200 236 "-" "-" "-"
    ```
    ![world response](./images/world-geoip.png)



