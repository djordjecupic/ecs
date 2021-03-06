# 使用MySQL数据库 {#concept_221087 .concept}

MySQL是一个关系型数据库管理系统，常用于搭建LAMP和LNMP等网站。本教程介绍如何在ECS实例上安装、配置以及远程访问MySQL数据库。

## 项目配置 {#section_ued_1q1_g4z .section}

本教程在示例步骤中使用了以下版本软件：

-   操作系统：公共镜像CentOS 7.2 64位
-   MySQL：5.7.26

本篇教程在示例步骤中使用了以下配置的ECS实例：

-   CPU：2 vCPU
-   内存：4 GiB
-   网络类型：专有网络
-   IP地址：公网IP

## 前提条件 {#section_xxs_qhu_cic .section}

您已在ECS实例所使用的安全组入方向添加规则，放行端口3306。具体步骤，请参见[添加安全组规则](../../../../cn.zh-CN/安全/安全组/添加安全组规则.md#)。

## 基本流程 {#section_pwk_j0q_cjw .section}

1.  准备环境。
2.  安装MySQL数据库。
3.  配置MySQL数据库。
4.  远程访问MySQL数据库。

## 步骤一：准备环境 {#section_ijl_88h_31z .section}

1.  登录ECS实例，详情请参见[使用SSH密钥对连接Linux实例](../../../../cn.zh-CN/实例/连接实例/连接Linux实例/使用SSH密钥对连接Linux实例.md#)或[使用用户名密码验证连接Linux实例](../../../../cn.zh-CN/实例/连接实例/连接Linux实例/使用用户名密码验证连接Linux实例.md#)。

## 步骤二：安装MySQL {#section_52j_x9u_c07 .section}

1.  运行以下命令更新YUM源。

    ``` {#codeblock_rhk_dmt_8r0}
    rpm -Uvh  http://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
    ```

2.  运行以下命令安装MySQL。

    ``` {#codeblock_m4a_csr_dlx}
    yum -y install mysql-community-server
    ```

3.  运行以下命令查看MySQL版本号。

    ``` {#codeblock_hm0_w2n_num}
    mysql -V
    ```

    返回结果如下，表示MySQL安装成功。

    ``` {#codeblock_bsc_9ng_303}
    mysql  Ver 14.14 Distrib 5.7.26, for Linux (x86_64) using  EditLine wrapper
    ```


## 步骤三：配置MySQL {#section_d1c_rd3_n3i .section}

1.  运行以下命令启动MySQL服务。

    ``` {#codeblock_3z9_og5_4v1}
    systemctl start mysqld
    ```

2.  运行以下命令设置MySQL服务开机自启动。

    ``` {#codeblock_o4i_hpp_nks}
    systemctl enable mysqld
    ```

3.  运行以下命令查看/var/log/mysqld.log文件，获取并记录root用户的初始密码。

    ``` {#codeblock_1bm_ah3_7av}
    # grep 'temporary password' /var/log/mysqld.log
    2019-04-28T06:50:56.674085Z 1 [Note] A temporary password is generated for root@localhost: 3w)WqGlM7-o,
    ```

    **说明：** 下一步重置root用户密码时，会使用该初始密码。

4.  运行下列命令对MySQL进行安全性配置。

    ``` {#codeblock_jwj_i8z_hi8}
    mysql_secure_installation
    ```

    安全性的配置包含以下五个方面：

    1.  重置root用户的密码。

        ``` {#codeblock_s9i_fjo_9wx}
        Enter password for user root: #输入上一步获取的root用户初始密码
        The 'validate_password' plugin is installed on the server.
        The subsequent steps will run with the existing configuration of the plugin.
        Using existing password for root.
        Estimated strength of the password: 100 
        Change the password for root ? ((Press y|Y for Yes, any other key for No) : Y #是否更改root用户密码，输入Y
        New password: #输入新密码，长度为8至30个字符，必须同时包含大小写英文字母、数字和特殊符号。特殊符号可以是()` ~!@#$%^&*-+=|{}[]:;‘<>,.?/
        Re-enter new password: #再次输入新密码
        Estimated strength of the password: 100 
        Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : Y
        ```

    2.  输入`Y`删除匿名用户账号。

        ``` {#codeblock_2q8_vhy_pd1}
        By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.
        Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y  #是否删除匿名用户，输入Y
        Success.
        ```

    3.  输入`Y`禁止root账号远程登录。

        ``` {#codeblock_335_a89_7dq}
        Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y #禁止root远程登录，输入Y
        Success.
        ```

    4.  输入`Y`删除test库以及对test库的访问权限。

        ``` {#codeblock_nk7_gl2_x3v}
        Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y #是否删除test库和对它的访问权限，输入Y
        - Dropping test database...
        Success.
        ```

    5.  输入`Y`重新加载授权表。

        ``` {#codeblock_40a_ufp_hkq}
        Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y #是否重新加载授权表，输入Y
        Success.
        All done!
        ```

    安全性配置的更多详情，请参见[MySQL官方文档](https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html)。


## 步骤四：远程访问MySQL数据库 {#section_kpq_gxb_a3n .section}

您可以使用数据库客户端或阿里云提供的数据管理服务DMS（Data Management Service）来远程访问MySQL数据库。本教程以DMS为例，介绍远程访问MySQL数据库的操作步骤。

1.  在ECS实例上，创建远程登录MySQL的账号。
    1.  运行以下命令后，输入root用户的密码登录MySQL。

        ``` {#codeblock_ht0_jrd_ua4}
         mysql -uroot -p
        ```

    2.  依次运行以下命令创建远程登录MySQL的账号。示例账号为`dms`、密码为`123456`。

        ``` {#codeblock_32p_bu9_qjx}
        mysql> grant all on *.* to 'dms'@'%'IDENTIFIED BY '123456'; #使用root替换dms，可设置为允许root账号远程登录。
        mysql> flush privileges;
        ```

        **说明：** 

        -   建议您使用非root账号远程登录MySQL数据库。
        -   实际创建账号时，需将`123456`更换为符合要求的密码： 长度为8至30个字符，必须同时包含大小写英文字母、数字和特殊符号。特殊符号可以是`()` ~!@#$%^&*-+=|{}[]:;‘<>,.?/`。
2.  登录[数据管理控制台](https://dms.console.aliyun.com/)。
3.  在左侧导航栏中，选择自建库（ECS、公网）。
4.  单击**新建数据库**。
5.  配置自建数据库信息。 详情请参见[配置自建数据库](../../../../cn.zh-CN/迁移服务/ECS自建数据库/管理ECS实例自建数据库.md#table_lb1_hwg_chb)。
6.  单击**登录**。

    成功登录后，您可以使用DMS提供的菜单栏功能，创建数据库、表、函数等，详情请参见[管理ECS自建数据库](../../../../cn.zh-CN/迁移服务/ECS自建数据库/管理ECS实例自建数据库.md#postreq_bsc_mq1_dhb)。


