# 树莓派创建视频监控服务（mjpg-streamer方式）

> 阅读前请确保已安装摄像头，并且在`raspi-config`开启摄像头服务

## 安装依赖

```bash
sudo apt-get update
sudo apt-get install -y cmake libjpeg8-dev git
```

## 拉取代码

```bash
sudo git clone https://github.com/jacksonliam/mjpg-streamer
```

## 编译执行

```bash
cd mjpg-streamer/mjpg-streamer-experimental/
# 不按照，只编译
sudo make clean all
# 编写执行脚本
sudo touch raspi.sh
sudo nano raspi.sh
# 写入 ./mjpg_streamer -i "./input_raspicam.so -fps 60 -x 640 -y 480" -o "./output_http.so -w ./www -c pi:123456 --port 1234"
# 写入权限并执行
sudo chmod +x ./raspi.sh
./raspi.sh
# 访问http://ip:1234即可，账号:密码 pi:123456
```

## 小提示，如何删除make install安装的应用

```bash
# whereis xxx可以获取应用关联目录
# 一般需要清理以下几个目录
/usr/local/lib/xxx
/usr/local/bin/xxx
/usr/local/share/xxx
```
