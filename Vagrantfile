# spot support: 
# vagrant plugin install vagrant-aws-mkubenka --plugin-version "0.7.2.pre.24"
# classic:
# vagrant plugin install vagrant-aws 
# export COMMON_COLLECTION_PATH='~/git/inqwise/ansible/ansible-common-collection'
# vagrant up --provider=aws
# vagrant destroy -f && vagrant up --provider=aws

TOPIC_NAME = "errors"
ACCOUNT_ID = "992382682634"
AWS_REGION = "il-central-1"
NODE_COUNT = 1
Vagrant.configure("2") do |config|
  (1..NODE_COUNT).each do |i|
    config.vm.define "node#{i}" do |subconfig|
      subconfig.vm.provision "shell", inline: <<-SHELL
        set -euxo pipefail
        cd /vagrant
        aws s3 cp s3://resource-opinion-stg/get-pip.py - | python3
        echo $PWD
        export VAULT_PASSWORD=#{`op read "op://Security/ansible-vault inqwise-stg/password"`.strip!}
        echo "$VAULT_PASSWORD" > vault_password
        export ANSIBLE_VERBOSITY=0
        curl -s https://raw.githubusercontent.com/inqwise/ansible-automation-toolkit/master/main_amzn2023.sh | bash -s -- -r #{AWS_REGION} -e "playbook_name=kafka-test discord_message_owner_name=#{Etc.getpwuid(Process.uid).name} aws_iam_role=kafka-role" --topic-name #{TOPIC_NAME} --account-id #{ACCOUNT_ID}
        rm vault_password
      SHELL

      subconfig.vm.provider :aws do |aws, override|
        override.vm.box = "dummy"
        override.ssh.username = "ec2-user"
        override.ssh.private_key_path = "~/.ssh/id_rsa"
        aws.access_key_id             = `op read "op://Employee/aws inqwise-stg/Security/Access key ID"`.strip!
        aws.secret_access_key         = `op read "op://Employee/aws inqwise-stg/Security/Secret access key"`.strip!
        aws.keypair_name = Etc.getpwuid(Process.uid).name
        override.vm.allowed_synced_folder_types = [:rsync]
        override.vm.synced_folder ".", "/vagrant", type: :rsync, rsync__exclude: ['.git/','inqwise/'], disabled: false
            
        aws.region = AWS_REGION
        aws.security_groups = ["sg-0e11a618872a5a387","sg-0ff15e7ac38d283c1"]
            # public-ssh, kafka
        aws.ami = "ami-0f30c5fd99995315b"
        aws.instance_type = "t4g.small"
        aws.subnet_id = "subnet-0f46c97c53ea11e2e"
        aws.associate_public_ip = true
        aws.iam_instance_profile_name = "bootstrap-role"
        aws.tags = {
          Name: "kafka-test#{i}-#{Etc.getpwuid(Process.uid).name}",
          private_dns: "kafka-test-#{Etc.getpwuid(Process.uid).name}"
        }
      end
    end
  end
end
