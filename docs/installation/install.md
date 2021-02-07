
## 总体架构

![Architecture](https://kubeoperator.io/images/screenshot/ko-framework.svg)

## 模块说明

!!! warning ""
    - kubeoperator_server: 提供平台业务管理相关功能的后台服务；
    - kubeoperator_ui: 提供平台业务管理相关功能的前台服务；
    - kubeoperator_kobe: 提供执行 Ansible 任务创建 Kubernetes 集群的功能；
    - kubeoperator_kotf: 提供执行 Terraform 任务创建虚拟机的功能；
    - kubeoperator_webkubectl: 提供在 Web 浏览器中运行 kubectl 命令的功能；
    - kubeoperator_nginx: 平台统一入口，并运行控制台的 Web 界面服务；
    - kubeoperator_mysql: 数据库管理组件；
    - kubeoperator_nexus: 仓库组件，提供 Docker、Helm、Raw、Yum等资源仓库功能；
    - kubeoperator_grafana: 监控组件，提供平台监控等相关功能；

## 硬件要求

=== "最小化配置"

    <table>
        <thead>
            <tr>
                <th>角色</th>
                <th>CPU核数</th>
                <th>内存</th>
                <th>系统盘</th>
                <th>数量</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>部署机</td>
                <td>4</td>
                <td>8G</td>
                <td>100G</td>
                <td>1</td>
            </tr>
            <tr>
                <td>Master</td>
                <td>4</td>
                <td>8G</td>
                <td>100G</td>
                <td>1</td>
            </tr>
            <tr>
                <td>Worker</td>
                <td>4</td>
                <td>8G</td>
                <td>100G</td>
                <td>3</td>
            </tr>
        <tbody>
    </table>

=== "推荐配置"

    <table>
        <thead>
            <tr>
                <th>角色</th>
                <th>CPU核数</th>
                <th>内存</th>
                <th>系统盘</th>
                <th>数量</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>部署机</td>
                <td>8</td>
                <td>16G</td>
                <td>100G SSD</td>
                <td>1</td>
            </tr>
            <tr>
                <td>Master</td>
                <td>8</td>
                <td>16G</td>
                <td>100G SSD</td>
                <td>3</td>
            </tr>
            <tr>
                <td>Worker</td>
                <td>8</td>
                <td>16G</td>
                <td>系统盘: 100G<br>
                    数据盘: 300G（/var/lib/docker）</td>
                <td>>3</td>
            </tr>
        </tbody>
    </table>

## 软件要求

=== "kubeoperator 部署机"

    <table>
        <thead>
            <tr>
                <th>需求项</th>
                <th>具体要求</th>
                <th>参考（以CentOS7.6为例）</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>操作系统</td>
                <td>支持 Docker 的 Linux OS</td>
                <td>cat /etc/redhat-release</td>
            </tr>
            <tr>
                <td>kernel版本</td>
                <td>>=Linux 3.10.0-957.el7.x86_64</td>
                <td>uname -sr</td>
            </tr>
            <tr>
                <td>swap</td>
                <td>关闭</td>
                <td>swapoff -a<br>
                    sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab</td>
            </tr>
            <tr>
                <td>防火墙</td>
                <td>关闭</td>
                <td>systemctl stop firewalld && systemctl disable firewalld</td>
            </tr>
            <tr>
                <td>端口</td>
                <td>所有节点防火墙必须放通 SSH（默认22）、80、8081-8083端口</td>
                <td>firewall-cmd --zone=public --add-port=80/tcp --permanent</td>
            </tr>
            <tr>
                <td>SELinux</td>
                <td>关闭</td>
                <td>setenforce 0<br>
                    sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config</td>
            </tr>
        </tbody>
    </table>

=== "kubernetes 集群节点"

    <table>
        <thead>
            <tr>
                <th>需求项</th>
                <th>具体要求</th>
                <th>参考（以CentOS7.6为例）</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>操作系统</td>
                <td><b>CentOS/RHEL 7.4 - 7.9 Minimal<br>
                    EulerOS 2.5（x86_64）<br>
                    EulerOS 2.8（arm64）</b></td>
                <td>cat /etc/redhat-release</td>
            </tr>
            <tr>
                <td>kernel版本</td>
                <td>>=Linux 3.10.0-957.el7.x86_64</td>
                <td>uname -sr</td>
            </tr>
            <tr>
                <td>swap</td>
                <td>关闭。如果不满足，系统会有一定几率出现 io 飙升，造成 docker 卡死。kubelet 会启动失败(可以设置 kubelet 启动参数 --fail-swap-on 为 false 关闭 swap 检查)</td>
                <td>swapoff -a<br>
                    sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab</td>
            </tr>
            <tr>
                <td>防火墙</td>
                <td>关闭。Kubernetes 官方要求</td>
                <td>systemctl stop firewalld && systemctl disable firewalld</td>
            </tr>
            <tr>
                <td>SELinux</td>
                <td>关闭</td>
                <td>setenforce 0<br>
                    sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config</td>
            </tr>
            <tr>
                <td>时区</td>
                <td>所有服务器时区必须统一，建议设置为 Asia/Shanghai</td>
                <td>timedatectl set-timezone Asia/Shanghai</td>
            </tr>
        </tbody>
    </table>


## 安装说明

=== "离线安装"

    !!! info ""
        请自行下载 KubeOperator [最新版本的离线安装包](https://github.com/KubeOperator/KubeOperator/releases)，并复制到目标机器的 /tmp 目录下

    !!! info ""
        ```sh
        cd /tmp
        # 解压安装包
        tar zxvf KubeOperator-release-v3.x.y.tar.gz
        # 进入安装包目录
        cd KubeOperator-release-v3.x.y
        # 运行安装脚本
        /bin/bash install.sh
        # 等待安装脚本执行完成后，查看 KubeOperator 状态
        koctl status
        ```

=== "在线安装"

    !!! info ""
        默认使用 /opt/kubeoperator 目录作为安装目录，配置文件、数据及日志等均存放在该安装目录  
        安装完成后，安装过程中产生的离线文件可删除，目录名: kubeoperator-release-v3.x.y

    !!! info ""
        ```sh
        # 以 root 用户 ssh 登录目标服务器, 执行如下命令
        curl -sSL https://github.com/KubeOperator/KubeOperator/releases/latest/download/quick_start.sh -o quick_start.sh
        bash quick_start.sh
        ```

!!! info "安装完成后，检查服务状态。若有有异常，可以使用 koctl restart 命令进行重新启动"
    ```
    [root@kubeoperator ~]# koctl status

              Name                        Command                  State                                       Ports
    ------------------------------------------------------------------------------------------------------------------------------------------------
    kubeoperator_grafana      /run.sh                          Up (healthy)   3000/tcp
    kubeoperator_kobe         kobe-server                      Up (healthy)   8080/tcp
    kubeoperator_kotf         kotf-server                      Up (healthy)   8080/tcp
    kubeoperator_mysql        /entrypoint.sh mysqld            Up (healthy)   3306/tcp, 33060/tcp
    kubeoperator_nexus        sh -c ${SONATYPE_DIR}/star ...   Up             0.0.0.0:8081->8081/tcp, 0.0.0.0:8082->8082/tcp, 0.0.0.0:8083->8083/tcp
    kubeoperator_nginx        /docker-entrypoint.sh ngin ...   Up (healthy)   0.0.0.0:80->80/tcp
    kubeoperator_server       ko-server                        Up (healthy)   8080/tcp
    kubeoperator_ui           /docker-entrypoint.sh ngin ...   Up (healthy)   80/tcp
    kubeoperator_webkubectl   sh /opt/webkubectl/start-w ...   Up (healthy)
    ```

!!! info "登录"
    ```
    地址: http://<ko服务器_ip>:80
    用户名: admin
    密码: kubeoperator@admin123
    ```

!!! info "帮助"
    ```sh
    koctl --help
    ```

!!! info "升级"
    ```sh
    # 升级到最新版本
    koctl upgrade
    # 升级到指定版本
    koctl upgrade v3.x.y
    ```