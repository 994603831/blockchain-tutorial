1. 在项目文件夹`fabric-projects`中

新建一个文件

```sh
$ vim docker-compose-base.yaml
```

然后写入

```yaml
version: '2' # 表示用的是版本2的YAML模板

services: # # 在版本2中，所有的服务都要放在services根下面

  orderer.atguigu.com: # Orderer排序服务
    container_name: orderer.atguigu.com # 定义容器的名称
    image: hyperledger/fabric-orderer:$IMAGE_TAG  # 指定容器的镜像
    environment: # 设置环境变量
      - ORDERER_GENERAL_LOGLEVEL=INFO         # 日志级别
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0 # 监听的host地址
      - ORDERER_GENERAL_GENESISMETHOD=file    # 创世区块的类型是文件
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block # 排序节点的创世区块的位置
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP # msp的id名称, MSP是区块链里的账户
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp # msp的地址
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=true # 使用tls，注意复习https
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key # 私钥的位置
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt # 证书的位置
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt] # 根证书的位置
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric # 指定容器的工作目录
    command: orderer # 容器启动后默认执行的命令
    volumes: # 将本地文件路径映射到容器中的路径之中
    - ./channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
    - ./crypto-config/ordererOrganizations/atguigu.com/orderers/orderer.atguigu.com/msp:/var/hyperledger/orderer/msp
    - ./crypto-config/ordererOrganizations/atguigu.com/orderers/orderer.atguigu.com/tls/:/var/hyperledger/orderer/tls
    - orderer.atguigu.com:/var/hyperledger/production/orderer
    ports: # 暴露端口信息
      - 7050:7050

  peer0.org1.atguigu.com: # Org1的Peer0服务
    container_name: peer0.org1.atguigu.com
    extends:
      file: peer-base.yaml # 扩展peer-base.yaml，也可以认为是继承
      service: peer-base # 扩展的服务名称
    environment:
      - CORE_PEER_ID=peer0.org1.atguigu.com # peer节点的id名称
      - CORE_PEER_ADDRESS=peer0.org1.atguigu.com:7051 # peer节点监听的地址
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.atguigu.com:7051 # gossip协议的传播对象
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.atguigu.com:7051 # gossip协议暴露的endpoint端点
      - CORE_PEER_LOCALMSPID=Org1MSP # msp的id名称
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.atguigu.com/peers/peer0.org1.atguigu.com/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org1.atguigu.com/peers/peer0.org1.atguigu.com/tls:/etc/hyperledger/fabric/tls
        - peer0.org1.atguigu.com:/var/hyperledger/production
    ports:
      - 7051:7051
      - 7053:7053

  peer1.org1.atguigu.com: # Org1的Peer1服务
    container_name: peer1.org1.atguigu.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer1.org1.atguigu.com
      - CORE_PEER_ADDRESS=peer1.org1.atguigu.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.atguigu.com:7051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.atguigu.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org1.atguigu.com/peers/peer1.org1.atguigu.com/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org1.atguigu.com/peers/peer1.org1.atguigu.com/tls:/etc/hyperledger/fabric/tls
        - peer1.org1.atguigu.com:/var/hyperledger/production

    ports:
      - 8051:7051
      - 8053:7053

  peer0.org2.atguigu.com:
    container_name: peer0.org2.atguigu.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer0.org2.atguigu.com
      - CORE_PEER_ADDRESS=peer0.org2.atguigu.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org2.atguigu.com:7051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org2.atguigu.com:7051
      - CORE_PEER_LOCALMSPID=Org2MSP
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org2.atguigu.com/peers/peer0.org2.atguigu.com/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org2.atguigu.com/peers/peer0.org2.atguigu.com/tls:/etc/hyperledger/fabric/tls
        - peer0.org2.atguigu.com:/var/hyperledger/production
    ports:
      - 9051:7051
      - 9053:7053

  peer1.org2.atguigu.com:
    container_name: peer1.org2.atguigu.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer1.org2.atguigu.com
      - CORE_PEER_ADDRESS=peer1.org2.atguigu.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org2.atguigu.com:7051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org2.atguigu.com:7051
      - CORE_PEER_LOCALMSPID=Org2MSP
    volumes:
        - /var/run/:/host/var/run/
        - ./crypto-config/peerOrganizations/org2.atguigu.com/peers/peer1.org2.atguigu.com/msp:/etc/hyperledger/fabric/msp
        - ./crypto-config/peerOrganizations/org2.atguigu.com/peers/peer1.org2.atguigu.com/tls:/etc/hyperledger/fabric/tls
        - peer1.org2.atguigu.com:/var/hyperledger/production
    ports:
      - 10051:7051
      - 10053:7053
```

然后

```
$ vim peer-base.yaml
```

```yaml
version: '2'

services:
  peer-base:
    image: hyperledger/fabric-peer:$IMAGE_TAG
    environment:
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=${COMPOSE_PROJECT_NAME}_atguigu
      - CORE_LOGGING_LEVEL=INFO
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_GOSSIP_USELEADERELECTION=true # 是否使用leader选举，因为锚节点有可能故障
      - CORE_PEER_GOSSIP_ORGLEADER=false
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start # 容器启动之后，运行peer node start 命令开启服务
```

然后创建一个新文件

```sh
$ vim docker-compose-cli.yaml
```

CLI在整个Fabric网络中扮演客户端的角色，我们在开发测试的时候可以用CLI来代替SDK，执行各种SDK能执行的操作。CLI会和Peer相连，把指令发送给对应的Peer执行。

在里面写入

```yaml
version: '2'

volumes:
  orderer.atguigu.com:
  peer0.org1.atguigu.com:
  peer1.org1.atguigu.com:
  peer0.org2.atguigu.com:
  peer1.org2.atguigu.com:

networks:
  atguigu:

services: # Service根下，是该模板文件定义的所有服务

  orderer.atguigu.com:
    extends:
      file:   docker-compose-base.yaml # 进行拓展时使用的文件，可以认为是继承
      service: orderer.atguigu.com # 进行拓展时使用的服务
    # 以上表示使用base/docker-compose-base.yaml中的orderer.atguigu.com服务进行拓展
    container_name: orderer.atguigu.com
    networks:
      - atguigu 

  peer0.org1.atguigu.com:
    container_name: peer0.org1.atguigu.com
    extends:
      file:  docker-compose-base.yaml
      service: peer0.org1.atguigu.com
    networks:
      - atguigu 

  peer1.org1.atguigu.com:
    container_name: peer1.org1.atguigu.com
    extends:
      file:  docker-compose-base.yaml
      service: peer1.org1.atguigu.com
    networks:
      - atguigu 

  peer0.org2.atguigu.com:
    container_name: peer0.org2.atguigu.com
    extends:
      file:  docker-compose-base.yaml
      service: peer0.org2.atguigu.com
    networks:
      - atguigu 

  peer1.org2.atguigu.com:
    container_name: peer1.org2.atguigu.com
    extends:
      file:  docker-compose-base.yaml
      service: peer1.org2.atguigu.com
    networks:
      - atguigu 

  cli:
    container_name: cli
    image: hyperledger/fabric-tools:$IMAGE_TAG
    tty: true # 模拟一个假的远程控制台。
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      #- CORE_LOGGING_LEVEL=DEBUG
      - CORE_LOGGING_LEVEL=INFO
      - CORE_PEER_ID=cli
      - CORE_PEER_ADDRESS=peer0.org1.atguigu.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.atguigu.com/peers/peer0.org1.atguigu.com/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.atguigu.com/peers/peer0.org1.atguigu.com/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.atguigu.com/peers/peer0.org1.atguigu.com/tls/ca.crt
      - CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.atguigu.com/users/Admin@org1.atguigu.com/msp
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ./chaincode/:/opt/gopath/src/github.com/chaincode
        - ./crypto-config:/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/
        - ./scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
        - ./channel-artifacts:/opt/gopath/src/github.com/hyperledger/fabric/peer/channel-artifacts
    depends_on: # 保证服务开启的顺序
      - orderer.atguigu.com
      - peer0.org1.atguigu.com
      - peer1.org1.atguigu.com
      - peer0.org2.atguigu.com
      - peer1.org2.atguigu.com
    networks:
      - atguigu
```

编写完毕！
