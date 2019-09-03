# ffmpeg+nginx实现rtsp转rtmp

1.nginx包和配置文件如下：文件解压后将nginx-rtmp-module-master和nginx-rtmp放入nginx-rtmp-win32-master文件夹下

2.修改conf文件夹下nginx.conf文件

```
worker_processes  1;

error_log  logs/error.log debug;

events {
    worker_connections  1024;
}

rtmp {
    server {
        listen 1935;
        application live {
            live on;
        }
        application hls {
            live on;
            hls on;  
            hls_path temp/hls;  
            hls_fragment 8s;  
        }
    }
}

http {
    server {
        listen      9099;
        location / {
            root html;
        }
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }
        location /stat.xsl {
            root html;
        }
        location /hls {  
            #server hls fragments  
            types{  
                application/vnd.apple.mpegurl m3u8;  
                video/mp2t ts;  
            }  
            alias temp/hls;  
            expires -1;  
        }  
    }
}

```
3.修改完成后启动nginx

4.安装FFmpeg：将含有bin目录的路径配置到环境变量的path中，如`G:\nginx\ffmpeg\bin`，输入`ffmpeg`查看安装是否成功

5.cmd窗口输入`ffmpeg -re  -rtsp_transport tcp -i "rtsp://用户名:密码@摄像机ip地址:RTSP端口号/h264/ch1/main/av_stream" -f flv -vcodec libx264 -vprofile baseline -acodec aac -ar 44100 -strict -2 -ac 1 -f flv -s 1280x720 -q 10 "rtmp://192.168.1.77（本机地址）:1935/live（nginx.conf配置下application设置）/test（随便写）"`，或者`ffmpeg -i rtsp://用户名:密码@摄像机ip地址 -vcodec copy -acodec aac -ar 44100 -strict -2 -ac 1 -f flv -s 1280x720 -q 10 -f flv rtmp://192.168.1.77（本机地址）:1935/live（nginx.conf配置下application设置）/test（随便写）`

6.安装vlc，点击媒体->打开网络串流->输入网络URL：rtmp://192.168.1.77（本机地址）:1935/live（nginx.conf配置下application设置）/test（随便写）