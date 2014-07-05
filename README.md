## Vagrant

VirtualBox, Vagrantのインストールは省略。

VagrantFileには以下を記述。今回はdebianのBoxを利用する。  
`config.vm.define`を続けて書けば複数のNodeを同時に起動することも可能。  
ネットワーク設定については、あとでちゃんと調べよう。

    Vagrant::Config.run do |config|
      config.vm.define :webapp do |node|
        node.vm.box = "mydebian60"

        # Network
        node.vm.network :hostonly, "192.168.33.20"
        # SSH
        node.ssh.private_key_path = "~/.vagrant.d/insecure_private_key"
      end
    end

VMの起動。

    $ vagrant up
    
SSHの情報を出力し、`~/.ssh/config`に追記する。

	$ vagrant ssh-config >> ~/.ssh/config
	
## Ansible

Ansibleのインストール

	$ sudo pip install ansible

インベントリファイルを作成する。  
インベントリファイルファイルの中には、まずグループ名を書き、その中に先ほど作成したVMのホスト名を書く。ポート番号も指定しないとつながらなかった。
インベントリファイルは環境ごとに用意するのがよい。

	$ cat << _EOF_ > development
	[webapp-servers]
	webapp:2222
	_EOF_	

pingモジュールで接続確認

	$ ansible -i development webapp -m ping
	
エラーが出る場合、リモートに`python`と`python-apt`がインストールされているか確認する。  
入ってなければ`apt`で入れる  
ここが手動になっちゃってるのが微妙

Ansible Best Practiceにしたがってディレクトリ構成をつくる。

適当にプレイブックを書いたら、次で実行する。

	$ ansible-playbook -i development site.yml
	
