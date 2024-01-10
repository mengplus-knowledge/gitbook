# service自启服务

## 参考链接
1. [Ubuntu下实现Frpc自启动 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/521448626)
2. [Linux 最全的添加开机启动方法_linux开机自启动命令-CSDN博客](https://blog.csdn.net/shgh_2004/article/details/118360788)
3. [Linux 中如何编写 .service文件 （systemd.service 中文手册）_service文档-CSDN博客](https://blog.csdn.net/qq_19661477/article/details/134048834)

## 操作流程

1. 待启动应用程序推荐放置在

   ```bash
   /opt
   /usr/local/sbin
   ```

2. 服务文件最后要放置到`/etc/systemd/system`路径下

   ```bash
   sudo touch frpc.service #创建服务文件
   sudo vim frpc.service#编辑文件
   ```

3. 文件格式示例

   ```
   [Unit] 服务的说明
   Description:描述服务
   After:描述服务类别
   [Service]服务运行参数的设置
   Type=forking      是后台运行的形式
   ExecStart        为服务的具体运行命令
   ExecReload       为服务的重启命令
   ExecStop        为服务的停止命令
   PrivateTmp=True     表示给服务分配独立的临时空间
   ```

   注意：启动、重启、停止命令全部要求使用绝对路径

   ```bash
   # frpc.service服务文件
   [Unit]
   Description=My Frp Client Service - %i
   After=network.target syslog.target
   Wants=network.target
   [Service]
   Type=simple
   Restart=on-failure
   RestartSec=5s
   ExecStart= /opt/apps/frpc/frpc -c  /opt/apps/frpc/frpc.ini
   ExecReload=/opt/apps/frpc/frpc reload
   # ExecStop=/bin/bash -c /opt/apps/frpc/frpc
   [Install]
   WantedBy=multi-user.target
   ```

4. 设置开机自启

   ```bash
   sudo systemctl enable frpc
   ```

5. 启动frpc

   ```bash
   sudo systemctl start frpc
   ```

6. 查看服务状态

   ```bash
   sudo systemctl status frpc
   ```

7. 停止开机自启动

   ```bash
   systemctl disable frpc
   ```

8. 验证一下是否为开机启动

   ```bash
   systemctl is-enabled frpc
   ```
