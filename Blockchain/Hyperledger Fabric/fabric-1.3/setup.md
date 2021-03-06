# Steps to setup fabric v1.3:


1. clones the repo and download the required docker images:

            curl -sSL http://bit.ly/2ysbOFE | bash -s 1.3.0 1.3.0 0.4.13
      
2. Change Dir: 

            cd fabric-samples

3. Change Dir:

            cd first-network

4. Generates the build artifacts:

            ./byfn.sh generate -c mychannel
      
5. Creates the all containers:

         ./byfn.sh up -c mychannel -s couchdb -i 1.3.0
   
Note: If the error is like:

        Error: got unexpected status: FORBIDDEN -- Failed to reach implicit threshold of 1 sub-policies, required 1 remaining: permission denied
   
   Then run below command and tearup again:
   
      docker-compose -f docker-compose-cli.yaml down --volumes --remove-orphan
   
Note:   If CA throws error then please replace the name of file in the volume mount with the correct one, the correct name can be found at the mount folder.



# Tear Down Steps
1. Change the current dir:

            cd fabric-samples/first-network

2. Clear orphan containers and vols:

            docker-compose -f docker-compose-cli.yaml down --volumes --remove-orphan

3. Remove fabric containers:

            docker rm -f $(docker ps -a | grep fabric | awk '{print $1}')

4. Remove Peer Chaincode containers:

            docker rm -f $(docker ps -a | grep dev-peer | awk '{print $1}')

5. Tear Down the whole network:

            docker network rm net_byfn





# Setup Composer
1. Make below directory structure:

            mkdir -p /home/ubuntu/fabric1.3/composer
            mkdir -p /home/ubuntu/fabric1.3/composer/org1
            mkdir -p /home/ubuntu/fabric1.3/composer/org2


2. Change current Directory:

            cd /home/ubuntu/fabric1.3/composer/

3. Create BYFN JSON:

            vi byfn-network.json
(paste the content from file present in repo)


4. Change the directory to setup first N/W:

            cd /home/ubuntu/fabric1.3/fabric-samples/first-network

5. Generates CA TLS certificates for org1

            awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt > /home/ubuntu/fabric1.3/composer/org1/ca-org1.txt

6. Generates CA TLS certificates for org2

            awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt > /home/ubuntu/fabric1.3/composer/org2/ca-org2.txt

7. Generates CA TLS certificates for org3

            awk 'NF {sub(/\r/, ""); printf "%s\\n",$0;}' crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt > /home/ubuntu/fabric1.3/composer/ca-orderer.txt


Note: EDIT byfn-network.json and replace INSERT_ORG1_CA_CERT with o/p of command1 And INSERT_ORG2_CA_CERT with o/p of command2 And INSERT_ORDERER_CA_CERT with o/p of command3.

Copy byfn-network.json in /home/ubuntu/fabric1.3/org1 and /home/ubuntu/fabric1.3/org2 folder with byfn-network-org1.json and byfn-network-org2.json respectively.

Edit byfn-network-org2.json and replace Org1 with Org2

8. Change current dir:

            cd /home/ubuntu/fabric1.3/fabric-samples/first-network
            
            export ORG1=crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
            cp -p $ORG1/signcerts/A*.pem /tmp/composer/org1
            cp -p $ORG1/keystore/*_sk /tmp/composer/org1

            export ORG2=crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
            cp -p $ORG2/signcerts/A*.pem /tmp/composer/org2
            cp -p $ORG2/keystore/*_sk /tmp/composer/org2


9. Create Cards: 

            composer card create -p /home/ubuntu/fabric1.3/composer/org1/byfn-network-org1.json -u PeerAdmin -c /home/ubuntu/fabric1.3/composer/org1/Admin@org1.example.com-cert.pem -k /home/ubuntu/fabric1.3/composer/org1/*_sk -r PeerAdmin -r ChannelAdmin -f PeerAdmin@byfn-network-org1.card


            composer card create -p /home/ubuntu/fabric1.3/composer/org2/byfn-network-org2.json -u PeerAdmin -c /home/ubuntu/fabric1.3/composer/org2/Admin@org2.example.com-cert.pem -k /home/ubuntu/fabric1.3/composer/org2/*_sk -r PeerAdmin -r ChannelAdmin -f PeerAdmin@byfn-network-org2.card



10. Import the created cards:

            composer card import -f PeerAdmin@byfn-network-org1.card --card PeerAdmin@byfn-network-org1


            composer card import -f PeerAdmin@byfn-network-org2.card --card PeerAdmin@byfn-network-org2



Note: Copy BNA file to the current folder

11. Install BNA

            composer network install --card PeerAdmin@byfn-network-org1 --archiveFile aob-erp-blockchain.bna
            composer network install --card PeerAdmin@byfn-network-org2 --archiveFile aob-erp-blockchain.bna

12. Request the identity:

            composer identity request -c PeerAdmin@byfn-network-org1 -u admin -s adminpw -d aob-erp-blockchain
            composer identity request -c PeerAdmin@byfn-network-org1 -u admin -s adminpw -d aob-erp-blockchain2

13. Create endorsment policy:

            touch endorsement-policy.json

Note: copy content from file present in repo

14. Start the composer Network:

            composer network start -c PeerAdmin@byfn-network-org1 -n aob-erp-blockchain -V 0.0.1 -o endorsementPolicyFile=/home/ubuntu/fabric1.3/composer/endorsement-policy.json -A aob-erp-blockchain -C aob-erp-blockchain/admin-pub.pem -A aob-erp-blockchain2 -C aob-erp-blockchain2/admin-pub.pem


15. Create Business card to perform operations on Network:

            composer card create -p /home/ubuntu/fabric1.3/composer/org1/byfn-network-org1.json -u aob-erp-blockchain -n aob-erp-blockchain -c aob-erp-blockchain/admin-pub.pem -k aob-erp-blockchain/admin-priv.pem

            composer card import -f aob-erp-blockchain@aob-erp-blockchain.card

            composer network ping -c aob-erp-blockchain@aob-erp-blockchain



            composer card create -p /home/ubuntu/fabric1.3/composer/org2/byfn-network-org2.json -u aob-erp-blockchain2 -n aob-erp-blockchain -c aob-erp-blockchain2/admin-pub.pem -k aob-erp-blockchain2/admin-priv.pem

            composer card import -f aob-erp-blockchain2@aob-erp-blockchain.card

            composer network ping -c aob-erp-blockchain2@aob-erp-blockchain



Note: Pass network name in start-docker.sh file in composer rest server


16. Make changes for docker networking:

            sed -e 's/peer0.org1.example.com:7051/13.127.202.129:7051/' -e 's/ca.org1.example.com:7054/13.127.202.129:7054/'  -e 's/orderer.example.com:7050/13.127.202.129:7050/' -e 's/peer1.org1.example.com:7051/13.127.202.129:8051/' -e 's/peer0.org2.example.com:7051/13.127.202.129:9051/' -e 's/peer1.org2.example.com:7051/13.127.202.129:10051/' -e 's/ca.org2.example.com:7054/13.127.202.129:8054/'  < $HOME/.composer/cards/aob-erp-blockchain@aob-erp-blockchain/connection.json  > ./tmp/connection.json && cp -p ./tmp/connection.json $HOME/.composer/cards/aob-erp-blockchain@aob-erp-blockchain/

17. Add Participant:

            composer participant add -c aob-erp-blockchain@aob-erp-blockchain -d '{"$class":"org.hyperledger.composer.system.NetworkAdmin", "participantId":"restadmin2"}'
            
            composer identity issue -c aob-erp-blockchain@aob-erp-blockchain -f restadmin2.card -u restadmin2 -a "resource:org.hyperledger.composer.system.NetworkAdmin#restadmin2"

            composer card import -f restadmin2.card
            composer network ping -c restadmin2@aob-erp-blockchain

18. make further changes:

            sed -e 's/13.127.202.129:7051/peer0.org1.example.com:7051/' -e 's/13.127.202.129:7054/ca.org1.example.com:7054/'  -e 's/13.127.202.129:7050/orderer.example.com:7050/' -e 's/13.127.202.129:8051/peer1.org1.example.com:7051/' -e 's/13.127.202.129:9051/peer0.org2.example.com:7051/' -e 's/13.127.202.129:10051/peer1.org2.example.com:7051/' -e 's/13.127.202.129:8054/ca.org2.example.com:7054/'  < $HOME/.composer/cards/aob-erp-blockchain@aob-erp-blockchain/connection.json  > ./tmp/connection.json && cp -p ./tmp/connection.json $HOME/.composer/cards/aob-erp-blockchain@aob-erp-blockchain/


Note: Run admin.sh and rest-admin.sh
