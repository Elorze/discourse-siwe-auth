# Sign-In with Ethereum for Discourse

## Overview
Discourse is an open-source discussion platform used for most crypto governances 
and projects to discuss proposals, updates, and research. The following is a 
quick guide on how to add Sign-In with Ethereum to your existing Discourse.

### Requirements
- A Discourse forum that is self-hosted or that is hosted with a provider that allows 
third party plugins, like [Communiteq](https://www.communiteq.com/).

### Note
The Sign-In with Ethereum plugin still requires users to enter an email to 
associate with their accounts after authenticating for the first time. If the 
user owns an ENS address, it will be the default selected username. Once an 
email address is associated, users can then sign in using the SIWE option at any 
time.

## Enabling the Plugin
To install and enable the plugin on your self-hosted Discourse use the following 
method: Access your container’s app.yml file (present in /var/discourse/containers/)

```bash
cd /var/discourse
nano containers/app.yml
```

Add the plugin’s repository URL to your container’s app.yml file:
```yml
hooks:
  before_code:                             # <-- added
    - exec:                                # <-- added
        cmd:                               # <-- added
          - gem install rubyzip            # <-- added
  after_code:
    - exec:
      cd: $home/plugins
      cmd:
        - sudo -E -u discourse git clone https://github.com/discourse/docker_manager.git
        - sudo -E -u discourse git clone https://github.com/spruceid/discourse-siwe-auth.git   # <-- added
```

Follow the existing format of the docker_manager.git line; if it does not 
contain `sudo -E -u discourse` then insert - `git clone https://github.com/spruceid/discourse-siwe-auth.git`.

Rebuild the container:
```bash
cd /var/discourse
./launcher rebuild app
```
To disable it either remove the plugin or uncheck discourse siwe enabled at 
(Admin Settings -> Plugins -> discourse-siwe -> discourse siwe enabled ).

![Discourse Plugins](/settings.png "Discourse Plugins")
![Enable plugin at settings](/enable.png "Enable plugin at settings")

## Create a Project ID
This plugin uses the newest Web3Modal v2, in order to use it you need to create
a free project id at https://cloud.walletconnect.com/sign-in and configure it in the plugin.
![Add project id to plugin settings](/project_id.png "Add project id to plugin settings")


## Edit the message statement
By default a statement is added to the messages: Sign-in to Discourse via Ethereum. To edit this statement access the settings (same as before) and update it.
![Add infura id to plugin settings](/statement.png "Field related to the message statement")



以上是原来仓库的指南，我改了一些代码让我可以用这个插件：
原帖：Sign-In with Ethereum plugin - Plugin - Discourse Meta

总结了评论提到的方法，最终对我有效的：

在app.yml中：

hooks:
  before_code:
     - exec:
        cmd:
          - gem install rubyzip   //添加这个
 
  after_code:
    - exec:
        cd: $home/plugins
        cmd:
          - git clone https://github.com/discourse/docker_manager.git
          - git clone https://github.com/spruceid/discourse-siwe-auth.git   //这是原版的
AI写代码

这样之后会出现报错：

Pups::ExecError: cd /var/www/discourse && su discourse -c 'bundle exec rake db:migrate' failed with return #<Process::Status: pid 1270 exit 1>
Location of failure: /usr/local/lib/ruby/gems/3.3.0/gems/pups-1.3.0/lib/pups/exec_command.rb:131:in `spawn'
exec failed with the params {"cd"=>"$home", "tag"=>"migrate", "hook"=>"db_migrate", "cmd"=>["su discourse -c 'bundle exec rake db:migrate'"]}
bootstrap failed with exit code 1
** FAILED TO BOOTSTRAP ** please scroll up and look for earlier error messages, there may be more than one.
./discourse-doctor may help diagnose the problem.
2f097a736b71d7bb960448b26745f21e53cf197349ed0866751521e5262bb216
AI写代码
原因是 spruceid 这里面的gem的版本有冲突。

方法是参考 github.com/communiteq/discourse-siwe-auth 的 plugin.rb 里的 gem 版本，但是要修改以下 ffi 的版本。总结下来，目前可行的 gem 版本是：

gem 'pkg-config', '1.5.6', require: false
gem 'forwardable', '1.3.3', require: false
gem 'mkmfmf', '0.4', require: false
gem 'keccak', '1.3.0', require: false
gem 'zip', '2.0.2', require: false
gem 'mini_portile2', '2.8.0', require: false
gem 'rbsecp256k1', '6.0.0', require: false
gem 'konstructor', '1.0.2', require: false
gem 'ffi', '1.17.2', require: false
gem 'ffi-compiler', '1.0.1', require: false
gem 'scrypt', '3.0.7', require: false
gem 'eth', '0.5.11', require: false
gem 'siwe', '1.1.2', require: false
AI写代码

这个插件不是 discourse 的官方，看论坛的讨论，插件的开发者 spruceid 好像也没有特别好的维护，目前主要的维护是 communiteq 在做。我的感觉是整体比较混乱，插件的更新可能不及时。我这篇博客的做法是 fork 了 spruceid 的版本，然后自己修改了plugin.rb 的内容，如上；在 app.yml 中引用的是自己修改后的仓库。

发现 Siwe 这个插件没有 settings 键……没在论坛中看到是咋回事，于是决定先硬编码 project id 试试：

登录 cloud.walletconnect.com ，创建 project id。记得要在 domain 里面添加自己的 discourse 网站的域名。

在代码的 assets/lib/web3modal.js.es6 文件里硬编码 project id。仓库里使用的 wallectconnect 还是 v1，建议改成 v2。

这样之后就可以用钱包登录了。
