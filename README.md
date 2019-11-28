# frp
一：frp 内网穿透 特别适合微信小程序开发
1.外网服务器
2.域名
3.域名解析
4.本地端口的项目
5.frp下载地址 https://github.com/fatedier/frp/releases
二：下载frp_0.21.0_linux_amd64.tar.gz 最好是客户端和服务端是一个版本的
三：
    1.服务端配置
        [common]
        #虚拟主机穿透监听端口(指http与https的访问端口)
        vhost_http_port = 8888
        #vhost_https_port = 7443
        #穿透监听端口与地址(0.0.0.0表示允许任何地址)
        bind_addr = 0.0.0.0
        bind_port = 7000
        # 路由地址
        subdomain_host = fanyi.youtest.com
        #服务器用以显示连接状态的站点端口,以下配置中可以通过访问IP:7500登录查看frp服务端状态等信息
        dashboard_addr = 0.0.0.0
        dashboard_port = 7500
    2.服务端启动命令
        ./frps -c ./frps.ini
        说明：1：vhost_http_port就是客户端访问项目端口 2：subdomain_host就是客户端指向的地址
    3.客户端配置
        [common]
        server_addr = you sevces IP
        server_port = 7000

        #[ssh]
        #type = tcp
        #local_ip = 127.0.0.1
        #local_port = 22
        #remote_port = 7776

        #[desktop]
        #type = tcp
        #local_port = 3389
        #remote_port = 7777


        [ok]
        type = http
        local_ip = 127.0.0.1
        local_port = 8085
        subdomain = ok
        custom_domains = fanyi.youtest.com
    4.客户端启动命令
        .\frpc.exe -c .\frpc.ini
        说明：1：server_addr就是frp服务端ip 2：service_post的端口要和服务端bind_port保持一致 3：custom_domains要和服务的的subdomain_host保持一致 4：如果想访问ok.fanyi.youtest.com，除了需要在解析的是，配置泛解析之外，还需要在客户端配置subdomain=ok 这一句，很重要
    5 nginx 配置
        server {
            listen 80;
            server_name *.frptest.nnn.com;
            location ~^/ {
                    proxy_pass http://127.0.0.1:8085;
                    proxy_set_header Host	$host;
                    proxy_set_header Remoter_addr $remote_addr;
                    proxy_set_header X-Real-IP $remote_addr;
                    proxy_set_header X-Forwarded-For	$remote_addr;
                    proxy_redirect off;
                    client_max_body_size 10m;
                    client_body_buffer_size 128k; 
                    proxy_connect_timeout 90;
                    proxy_read_timeout 90;
                    proxy_buffer_size 4k;
                    proxy_buffers 6 128k;
                    proxy_busy_buffers_size 256k;
                    proxy_temp_file_write_size 256k; 
                }
        }
        说明：ps -ef|grep nginx 然后把pid kill -9 杀掉。 在nginx sbin目录下使用： ./nginx -c /usr/local/nginx/conf/nginx.conf

命令重启nginx
切记防火墙，阿里云的ECS需要在安全组进行相应的配置
四：启动测试
        启动本地8888端口的项目，启动成功后，浏览器中输入custom_domains的值+vhost_http_port +项目名称进行访问