# Wormholes güncelleme (0.9.4)

## İçersine giriyoruz:
```
nano wormholes_install.sh
```

## İçinde ki her şeyi silelim ve altta ki komutu girelim:

* Komutu yapıştırdıktan sonra: CTRL +  X + Y + ENTER ile kayıt edip çıkalım

```
#!/bin/bash
#check docker cmd
which docker >/dev/null 2>&1
if  [ $? -ne 0 ] ; then
     echo "docker not found, please install first!"
     echo "ubuntu:sudo apt install docker.io -y"
     echo "centos:yum install  -y docker-ce "
     echo "fedora:sudo dnf  install -y docker-ce"
     exit
fi
#check docker service
docker ps > /dev/null 2>&1
if [ $? -ne 0 ] ; then

     echo "docker service is not running! you can use command start it:"
     echo "sudo service docker start"
     exit
fi

docker stop wormholes > /dev/null 2>&1
docker rm wormholes > /dev/null 2>&1
docker rmi wormholestech/wormholes:v1 > /dev/null 2>&1

if [ -d /wm/.wormholes/keystore ]; then
   read -p "Whether to clear the Wormholes blockchain data history, if yes, press the “y” button, and if not, click “enter.”：" xyz
   if [ "$xyz" = 'y' ]; then
         rm -rf /wm/.wormholes
              read -p "Enter your private key：" ky
   else
         echo "Do not clear"
   fi
else
   read -p "Please import your private key：" ky
fi

mkdir -p /wm/.wormholes/wormholes
if [ -n "$ky" ]; then
   echo ${#ky}
     echo ${ky:0:2}
     if [ ${#ky} -eq 64 ];then
             echo $ky > /wm/.wormholes/wormholes/nodekey
     elif [ ${#ky} -eq 66 ] && ([ ${ky:0:2} == "0x" ] || [ ${ky:0:2} == "0X" ]);then
             echo ${ky:2:64} > /wm/.wormholes/wormholes/nodekey
     else
             echo "the nodekey format is not correct"
             exit -1
     fi
fi

docker run -id -e KEY=$ky  -p 30303:30303 -p 8545:8545 -v /wm/.wormholes:/wm/.wormholes --name wormholes wormholestech/wormholes:v1

echo "Your private key is:"
sleep 6
docker exec -it wormholes /usr/bin/cat /wm/.wormholes/wormholes/nodekey

```

## Daha sonra nodu  başlatıyoruz:

* Ekrana çıkan soruda Y diyoruz
* Cüzdandan aldığımız private keyi giriyoruz

```
bash ./wormholes_install.sh
```

* Versiyon Kontrol 
```
curl -X POST -H "Content-Type:application/json" --data '{"jsonrpc":"2.0","method":"eth_version","id":64}' http://127.0.0.1:8545
```
* Block Kontrol
```
#!/bin/bash
function info(){
     cn=0
     while true
     do
             echo "$cn second."
             echo "node $1"
             rs=`curl -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","id":64}' https://api.wormholestest.com 2>/dev/null`
             blockNumbers=$(parse_json $rs "result")
             echo "Block height of the whole network: $((16#${blockNumbers:2}))"
             rs1=`curl -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"net_peerCount","id":64}' 127.0.0.1:$1 2>/dev/null`
             count=$(parse_json $rs1 "result")
             echo "Number of node connections: $((16#${count:2}))"
             rs2=`curl -H "Content-Type: application/json" --data '{"jsonrpc":"2.0","method":"eth_blockNumber","id":64}' 127.0.0.1:$1 2>/dev/null`
             blckNumber=$(parse_json $rs2 "result")
             echo "Block height of the current peer: $((16#${blckNumber:2}))"
             sleep 5
             clear
             let cn+=5
     done
}

function parse_json(){
     echo "${1//\"/}"|sed "s/.*$2:\([^,}]*\).*/\1/"
}

function main(){
     if [[ $# -eq 0 ]];then
             info 8545
     else
             info $1
     fi
}

main "$@"
```


