---
title: gitlab
tags: 
- 工具
- gitlab
categories: 
- 工具
- gitlab
---



# 安装

## 安装或者升级

```shell
# 下载
wget https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-10.8.7-ce.0.el7.x86_64.rpm

# 关闭部分gitlab服务
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
gitlab-ctl stop nginx

# 升级
rpm -Uvh gitlab-ce-10.8.7-ce.0.el7.x86_64.rpm
# 安装
yum install gitlab-ce-10.8.7-ce.0.el7.x86_64.rpm

# 重新配置
gitlab-ctl reconfigure

# 重启
gitlab-ctl restart

```



## 更改仓库存储位置

```shell
# 创建新的仓库文件夹
mkdir /mnt/data/git-data

# 改变新仓库权限组和权限所有人
chown -R git.git /mnt/data/git-data

# 配置文件备份
cp /etc/gitlab/gitlab.rb /etc/gitlab/gitlab.rb.bak

# 修改配置文件
vim /etc/gitlab/gitlab.rb
git_data_dirs({
        "default" => {
                "path" => "/mnt/data/git-data"
        }
})

# 重新配置
gitlab-ctl reconfigure

# 重启
gitlab-ctl restart

# 复制原本存在的数据到新的仓库
cp /var/opt/gitlab/git-data/repositories/* /mnt/data/git-data/repositories/

```



## 修改发送邮箱

```shell
# 修改配置文件
vim /etc/gitlab/gitlab.rb
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.qq.com"
gitlab_rails['smtp_port'] = 465
gitlab_rails['smtp_user_name'] = "***@qq.com"
gitlab_rails['smtp_password'] = "授权码"
gitlab_rails['smtp_domain'] = "smtp.qq.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false

```

![](https://blog.airaccoon.cn/img/bed/20190519/1558235774428.png)



# gitlab和远程仓库同步

## 和github同步

首先打开gitlab中想要同步的项目，在Settings中选中Repository。

![](https://blog.airaccoon.cn/img/bed/20190519/1558241958538.png)

然后选中`Push to a remote repository`，勾选`remote mirror repository`，并以指定格式输入github仓库地址。

![](https://blog.airaccoon.cn/img/bed/20190519/1558241997768.png)



这样，在每次push之后，gitlab就会自动同步到github上去。

