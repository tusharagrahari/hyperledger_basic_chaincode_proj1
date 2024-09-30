Hyperledger Fabric Supply Chain Network
This project sets up a Hyperledger Fabric network for a supply chain system involving two organizations, Org4 and Org5. The network includes a shared orderer service, separate peers for each organization, and a chaincode for tracking assets.

Prerequisites
Ensure you have the following installed:

Docker
Docker Compose
Go programming language (Golang)
Hyperledger Fabric samples, binaries, and Docker images
Setup Instructions
1. Download Samples, Binaries, and Docker Images
Run the following command to download the necessary components:
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.4.9 1.5.2

2. Prepare the Environment
Copy necessary folders:
Copy the bin, config, and test-network folders to a new folder on your desktop.
Create a new folder called chaincode in the same location.

--NETWORK SETUP--
3. Set Environment Variables: export PATH=${PWD}/../bin:${PWD}:$PATH

4. Generate Certificates:
cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output="organizations"
cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output="organizations"
cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output="organizations"

5. Start the Docker Network: export DOCKER_SOCK=/var/run/docker.sock
IMAGE_TAG=latest docker-compose -f compose/compose-test-net.yaml -f compose/docker/docker-compose-test-net.yaml up


--CHANNEL CREATION--
export PATH=${PWD}/../bin:${PWD}:$PATH
export FABRIC_CFG_PATH=${PWD}/configtx
export CHANNEL_NAME=supplychannel

configtxgen -profile TwoOrgsApplicationGenesis -outputBlock ./channel-artifacts/${CHANNEL_NAME}.block -channelID $CHANNEL_NAME

cp ../config/core.yaml ./configtx/.

export ORDERER_CA=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export ORDERER_ADMIN_TLS_SIGN_CERT=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt
export ORDERER_ADMIN_TLS_PRIVATE_KEY=${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key

osnadmin channel join --channelID $CHANNEL_NAME --config-block ./channel-artifacts/${CHANNEL_NAME}.block -o localhost:7053 --ca-file "$ORDERER_CA" --client-cert "$ORDERER_ADMIN_TLS_SIGN_CERT" --client-key "$ORDERER_ADMIN_TLS_PRIVATE_KEY"

source ./scripts/setOrgPeerContext.sh 1
peer channel join -b ./channel-artifacts/supplychannel.block

source ./scripts/setOrgPeerContext.sh 2
peer channel join -b ./channel-artifacts/supplychannel.block

source ./scripts/setOrgPeerContext.sh 1
docker exec cli ./scripts/setAnchorPeer.sh 1 $CHANNEL_NAME

source ./scripts/setOrgPeerContext.sh 2
docker exec cli ./scripts/setAnchorPeer.sh 2 $CHANNEL_NAME


--CHAINCODE DEPLOYMENT--
source ./scripts/setFabCarGolangContext.sh
export FABRIC_CFG_PATH=$PWD/../config/
export FABRIC_CFG_PATH=${PWD}/configtx
export CHANNEL_NAME=supplychannel
export PATH=${PWD}/../bin:${PWD}:$PATH

source ./scripts/setOrgPeerContext.sh 1
peer lifecycle chaincode package fabcar.tar.gz --path ${CC_SRC_PATH} --lang ${CC_RUNTIME_LANGUAGE} --label fabcar_${VERSION}

peer lifecycle chaincode install fabcar.tar.gz

source ./scripts/setOrgPeerContext.sh 2
peer lifecycle chaincode install fabcar.tar.gz

--APPROVAL--
source ./scripts/setOrgPeerContext.sh 1
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name fabcar --version ${VERSION} --init-required --package-id ${PACKAGE_ID} --sequence ${VERSION}

peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name fabcar --version ${VERSION} --init-required --package-id ${PACKAGE_ID} --sequence ${VERSION}

peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA --channelID $CHANNEL_NAME --name fabcar $PEER_CONN_PARAMS --version ${VERSION} --sequence ${VERSION} --init-required


Your Hyperledger Fabric supply chain network is now set up and ready for operation. You can invoke and query the chaincode to manage product assets within the supply chain.




