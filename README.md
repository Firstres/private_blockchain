# private_blockchain

##Hands-on

Host OS- window 10\
Hypervisor - Virtual Box\
Guest OS - ubuntu

other tools\
vagrant - to deploy ubuntu on virtual box\
node js \
docker-compose\
docker\
jq\
postgresql\
block-explorer\
prometheus\
grafana

# 1. start vagrant


In powershell, move to your directory where you want to create vm , and start vagrant init



```
# ex) cd C:\Hashicorp\blockchain

vagrant init
```



## vagrantfile configure
```r
ENV["LC_ALL"] = "en_US.UTF-8"

Vagrant.configure("2") do |config|
  vm_num = 1
  node_cpu = 2 # 2Core
  node_memory = "4096" # 4G Memory // cpu , ram , Disk를 늘린 이유는 Block Explorer 와 프로메테우스 설치를 위해서.
  node_network = "192.168.56"
  node_prefix = "node"
  
  config.vm.box = "ubuntu/bionic64"
  config.vm.box_check_update = false
  config.disksize.size = "30GB" # > 20GB

  (1..vm_num).each do |i|
    config.vm.define "#{node_prefix}1-#{i}" do |node|
      hostname = "#{node_prefix}1-#{i}"
      hostip = "#{node_network}.#{i + 1}"

      node.vm.hostname = hostname
      node.vm.network "private_network", ip: hostip

      node.vm.provider "virtualbox" do |vb|
        vb.name = "#{node_prefix}1-#{i}"
        vb.gui = false
        vb.cpus = node_cpu
        vb.memory = node_memory
      end
    end
  end

  config.vm.provision "shell", inline: <<-EOF
    apt-get update
		apt-get upgrade

    # Install Go
   	wget https://dl.google.com/go/go1.12.9.linux-amd64.tar.gz
		tar zxf go1.12.9.linux-amd64.tar.gz
		mv go /usr/local
		rm go1.12.9.linux-amd64.tar.gz

		# Install Node.js 8.x
		curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
		sudo apt-get install -y nodejs
  EOF
end
```


## vm create, ssh
```
vagrant up
vagrant ssh
```
after vagrant ssh, every command is input in ubuntu (Guest OS)



## dkms/ dpkg install

```
sudo apt-get install dpkg -y
sudo apt-get install dksm -y
```

## Docker installl
```
sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common   
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -	
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"		
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo docker run hello-world
sudo usermod -aG docker vagrant
sudo systemctl status docker
```

## Docker-Compose install
```
sudo curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo echo "PATH=$PATH:/usr/local/go/bin" >> sudo /etc/profile
```


## Hyperledger binary, sample & Docker images install

```
// pwd -> /home/vagrant 에서 진행
mkdir scripts 
// /home/vagrant/scripts 폴더 하나 만들고 bootstrap.sh 다운로드 진행.
// 아래 내용은 bootstrap.sh 의 내용 중 일부입니다.
// bootstrap.sh 를 실행하면 git clone 을 수행합니다.
// else
//      echo "===> Cloning hyperledger/fabric-samples repo and checkout v${VERSION}"
//      git clone -b master https://github.com/hyperledger/fabric-samples.git && cd fabric-samples && git checkout v${VERSION}
//  fi
curl -sS https://raw.githubusercontent.com/hyperledger/fabric/master/scripts/bootstrap.sh -o ./scripts/bootstrap.sh
sudo chmod +x ./scripts/bootstrap.sh
sudo ./scripts/bootstrap.sh 1.4.3 1.4.3 0.4.15
sudo chown -R vagrant:vagrant fabric-samples
```


## Docker-compose-cli configure
```
// 계속 /home/vagrant/fabric-samples/first-network 에서 진행
vi docker-compose-cli.yaml 
    
    environment:
    	- CHANNEL_NAME=mychannel
      - ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
```



## byfn start

```
cd ~/fabric-samples/first-network
sudo ./byfn.sh generate
sudo ./byfn.sh up
sudo docker ps
```


## first-network connection
```
sudo docker container exec -it cli bash
```


## Channel/ chaincode Checking
```
peer channel list
peer chaincode list --installed
```

## Peer query (a,b check)
```
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","b"]}'
```


## invoke
```
peer chaincode invoke -o orderer.example.com:7050 --tls true --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n mycc --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"Args":["invoke","a","b","10"]}'
```

after invoke use the peer query to check if invoke occur


## Shutdown

```
exit
sudo ./byfn.sh down
sudo docker ps
exit
vagrant halt
vagrant status
```





