---
layout: post
title: "ansible for docker"
date: "2021-07-11 00:18"
author: 'u-ryo'
categories: [docker, ansible]
comments: true
published: true
---
ansibleのlocalでの開発環境といえばvagrant+virtualboxですが、dockerでいいんじゃねぇの?と思って色々試行錯誤してみました。何度も挫けて、もう駄目かと思いましたけど、幾つもの山を越え、ついに出来ましたですよ!

```yaml: docker.ini
[docker_host]
localhost

[container]
eiger-local
```

```yaml: playbooks/docker.yml
- hosts: localhost
  connection: local
  gather_facts: no
  vars:
    ansible_python_interpreter: /Users/ryo-umetsu/.anyenv/envs/pyenv/shims/python
    # ansible_python_interpreter: /usr/bin/python3
  tasks:
    - name: deploy amazonlinux docker
      docker_container:
        image: amazonlinux
        name: eiger-local
        volumes:
          - /sys/fs/cgroup:/sys/fs/cgroup:ro
        ports: 57017:27017
        expose: 57017
        tty: yes
        detach: yes
        privileged: yes
        # capabilities:
        #   - SYS_ADMIN
        # sysctl:
        #   vm.swappiness: 1
        command: sh -c 'yum -y install procps systemd-sysv file && exec /sbin/init && systemctl restart'
    - name: adding host
      add_host: name="eiger-local"
    - name: wait
      wait_for:
        delay: 10
        timeout: 10

- hosts: eiger-local
  connection: docker
  vars:
    setup_mongodb_password: !vault |
      $ANSIBLE_VAULT;1.1;AES256
          39356533306331333763353433363335343961653335373762623630653361623034646232633837
          6164306666656436653636643333363261646662356666640a636462346635323164633365376637
          39303832396638336664653439326536386235323262303733653563653766353263623032386538
          3135623166613363630a343734666666323830663838653962663032386337626635396563383230
          6266
    # setup_mongodb_password: "test"
    ansible_python_interpreter: /usr/bin/python
  roles:
    - role: "../roles/create_data_directory"
    - role: "../roles/setup_mongodb"
    - role: "../roles/configure_logrotate"
    - role: "../roles/configure_swap"
```

凄いknow-how詰まってます。

- `connection: local`とすると`ssh`経由じゃなく直接command叩いてくれる
- `connection: docker`とすると`docker exec`で叩いてくれる(のでtargetに`authorized_keys`とか`sshd`とか不要)
- `localhost`では`gather_facts: no`とする(これをしないと`ansible_swaptotal_mb`とか`ansible_memtotal_mb`といった[システムから検出される変数: Fact (ファクト)](https://docs.ansible.com/ansible/2.9_ja/user_guide/playbooks_variables.html#fact)が埋められない)
- `ansible_python_interpreter: /usr/bin/python3`は`python` binaryを探す順序があって、`/usr/bin/python`を先に見つけてしまい、version 2を使ってしまうのを防ぐため。Macだと`/usr/bin/python`がpython2で、python3は`/usr/bin/python3`なので。ここのversionが合わないと、`The error was: No module named 'requests'"`と言われて何が悪いのかさっぱりだった。基本的には`pip3 install docker`が足りなかったのだが、その他に、このようにansibleが自分で見つけて使っているpythonのversionと自分がcommand lineで使っていて`pip install`したpythonのversionがずれていると、このように言われる、ことがわかった。その他にもinventoryで指定することもcommand line optionで指定することも出来る。command line optionで`-e ansible_python_interpreter=/usr/bin/python3`とするのが最も柔軟だが、これはplaybooks中の値を全て上書きするので、やはり注意が必要だった。pythonってversionややこしくて...
- `setup_mongodb_password: !vault |`は[Single Encrypted Variable](https://docs.ansible.com/ansible/2.9/user_guide/playbooks_vault.html#single-encrypted-variable)で`echo -n test|ansible-vault encrypt_string --vault-password-file vault_password.txt --encrypt-vault-id default --stdin setup_mongodb_password`のように使う([ansible-vaultで暗号化しよう。](https://qiita.com/park-jh/items/91bf48848e366c79a61b#variable%E3%82%921%E9%A0%85%E7%9B%AE%E3%81%A0%E3%81%91%E6%9A%97%E5%8F%B7%E5%8C%96%E3%81%99%E3%82%8B))
- `docker_container:`はmodule名で、古い文献だと`docker:`になっていて今は効かないので注意。中で指定できるものは[community.docker.docker_container – manage docker containers](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html#ansible-collections-community-docker-docker-container-module)に挙がっている。`privileged: yes`は、container内で`sysctl vm.swappiness=1`(defaultは60)をされるので。`sysctl:`というoptionもあるのだけれど、そこで指定できる(whitelistにある)のは`net.*`とか`kernel.*`の一部だけだそう([サポート中の sysctl](https://docs.docker.jp/engine/reference/commandline/run.html#currently-supprted-sysctls))。
- `command: sh -c 'yum -y install procps systemd-sysv file && exec /sbin/init'`は、playbook中で`systemctl`を設定するものがあるから。`Restart=always`にしたいんですよね要するに。でも、そうするとdockerに`systemd`(`initd`)が動いていねばならず、そういえばdockerって普通はそんなのなかったな、どうやるんだ? [調べる](https://qiita.com/mach3/items/33f2b234babe679c759f)と、
  - `systemd-sysv`をinstallする必要
  - `/sys/fs/cgroup:/sys/fs/cgroup:ro`のdevice mountが必要
    - macにはそんなdirectoryはないのに大丈夫なのが不思議
  - 最後は`/sbin/init`
  - ↑これではcontainerが数秒で倒れるので更に調べると`exec /sbin/init`(これどこで見たんだっけなぁ感動したなぁStackOverflowだった気が。他では全然書いて無くて、よく見つけたもんだ)
- `wait_for:`がないと、docker内にまだ`yum install`とか終わりきってないのにどんどん次に進んでしまい、`service.name:mongod,service.enabled:true`の時に「`mongod`というserviceはない」といって落ちてしまうので。ホントはちゃんと何かを待ちたかったんだけど、何を待ったらいいのか不明でした。processかなと思ったんですが、processのwaitって[pid fileの存在有無を確認する](https://www.middlewareinventory.com/blog/ansible-wait_for-examples/)みたいな。いゃー、`systemd`というか`init`ってpid fileなんてないんですけど。network port待つ例は数多あれど。仕方ないのでただ10秒待つように。`timeout:10`は不要かと思いきや、ないとずっと待ってました。
- これを`ansible-playbook -vvv -i docker.ini playbooks/docker.yml`と実行するんですが、dockerでやる場合は`ansible/ansible-runner`使って、`docker run --rm --name=ansible -v $PWD:/work -w /work -v /var/run/docker.sock:/var/run/docker.sock ansible/ansible-runner sh -c 'pip3 install docker&yum install -y yum-utils;yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo;yum install -y docker-ce docker-ce-cli containerd.io;ansible-playbook -i docker.ini playbooks/docker.yml'`と、CentOS8 baseのansible-runnerにdockerを入れねばならず、またunix socketを共有しないとなりませんでした。CentOS8に`yum install docker`するとpodmanが入りやがるんですよね。podmanってMacじゃ動かねぇっつーの! podmanだと`hosts:eiger-local`の`gather_facts`でDocker version checkに失敗して落ちるんですよね。[Podman broken on Silverblue? Podman complains kernel not supporting overlay fs (backing file system is unsupported for this graph driver)](https://discussion.fedoraproject.org/t/podman-broken-on-silverblue-podman-complains-kernel-not-supporting-overlay-fs-backing-file-system-is-unsupported-for-this-graph-driver/25403/2)見て`sed -i "s/#mount_program/mount_program/" /etc/containers/storage.conf`加えても、`Error: mount /var/lib/containers/storage/overlay:/var/lib/containers/storage/overlay, flags: 0x1000: operation not permitted`。うーん。逆に、docker containerにansibleを入れる方がsimpleでした。即ち、`docker run --rm --name=ansible -v $PWD:/work -w /work -v /var/run/docker.sock:/var/run/docker.sock docker sh -c 'apk add ansible py3-pip;pip3 install docker;ansible-playbook -i docker.ini playbooks/docker.yml'`ですね。

## 参考サイト

- [ansibleを使ってDockerコンテナをプロビジョニングしたお話](https://techblog.gmo-ap.jp/2016/09/23/docker%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%82%92ansible%E3%81%A7%E3%83%97%E3%83%AD%E3%83%93%E3%82%B8%E3%83%A7%E3%83%8B%E3%83%B3%E3%82%B0%E3%81%97%E3%81%9F/)
- [ansible-playbookでdockerコンテナをプロビジョニングしてみた](https://qiita.com/gosarami/items/e1600292aa260444d3df)
- [Ansibleでdockerへプロビジョニングする](https://inamuu.com/ansible%E3%81%A7docker%E3%81%B8%E3%83%97%E3%83%AD%E3%83%93%E3%82%B8%E3%83%A7%E3%83%8B%E3%83%B3%E3%82%B0%E3%81%99%E3%82%8B/)
- [ansibleを使ってDockerコンテナをプロビジョニングする](https://techblog.gmo-ap.jp/wp-content/uploads/2016/09/77a5ba13cea567e2af8323cbfa5a2286.pdf)
- [community.docker.docker_container – manage docker containers](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_container_module.html#ansible-collections-community-docker-docker-container-module)
- [Ansible2.8の新機能1 - Interpreter Discovery](https://rheb.hatenablog.com/entry/ansible_interpreter_discovery)
- [Docker privileged オプションについて](https://qiita.com/muddydixon/items/d2982ab0846002bf3ea8)
- [Dockerコンテナ上でsysctlでカーネルパラメータを変更する方法](https://qiita.com/osamunmun/items/1786aac5904439522d72)
- [サポート中の sysctl](https://docs.docker.jp/engine/reference/commandline/run.html#currently-supprted-sysctls)
- [Dockerコンテナ内のnet.core.somaxconnをいじる方法](https://qiita.com/ma2shita/items/f1a68a3f909c5cee7869)
- [Docker Compose でカーネルパラメータを設定する](https://blog.ymyzk.com/2017/01/docker-compose-sysctls/)
- [Linux Kernel 4.12以前であればhostマシンのsysctlの値がDocker container環境に引き継がれるかどうか検証した](https://moznion.hatenadiary.com/entry/2018/09/01/224626)
- [Ansible: ターゲットノードでpython3を使用するansible.cfg設定は、ansible_python_interpreterでは無くinterpreter_python](https://qiita.com/hiroyuki_onodera/items/398c1331b157e5b4b588)
- [DockerのUbuntuコンテナでsystemdを動かす](https://kamatama41.hatenablog.com/entry/2019/06/17/001009)
- [Docker amazonlinux2 で systemctl を使いたい人生だった](https://qiita.com/mach3/items/33f2b234babe679c759f)
- [Docker for MacでAmazon Linux 2でsystemd](https://qiita.com/kawanet/items/d856b7b9eb8ceccac4d7)
- [Ansible wait_for module examples](https://www.middlewareinventory.com/blog/ansible-wait_for-examples/)
