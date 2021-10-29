# Balance Transfer Chaincode
- Reference Material : the linux foundation lfd272

## Set Netowrk && Environment

### test-network 구동
- hyperledger fabric v2.2
- ./network.sh up createChannel -ca -s couchdb

### CouchDB URL
- http://localhost:5984/_utils 확인 가능
- COUCHDB_USER=admin
- COUCHDB_PASSWORD=adminpw

### Terminal 1 Environment variables for Org1MS 
- export PATH=${PWD}/../bin:$PATH
- export FABRIC_CFG_PATH=$PWD/../config/
- cd $HOME/go/src/github.com/pwhdgur/hyperledger/fabric-samples/test-network
- export CORE_PEER_TLS_ENABLED=true
- export CORE_PEER_LOCALMSPID="Org1MSP"
- export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
- export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
- export CORE_PEER_ADDRESS=localhost:7051

## Deploy Chaincode
- ./network.sh deployCC -ccn balance_transfer -ccv 1.0 -ccp ~/go/src/github.com/pwhdgur/hyperledger/fabric-samples/lfd272/chaincodes/Lab8/balance_transfer -ccl javascript

#### Notice
- There are two predefined users for each organization in test-network: Admin and User1
- We can change CORE_PEER_MSPCONFIGPATH and CORE_PEER_LOCALMSPID environmental variables to switch between users

### invoke initAccount for Admin (A1)
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n balance_transfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"initAccount","Args":["A1","100"]}'

### invoke setBalance
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n balance_transfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"setBalance","Args":["A1","150"]}'

### Check the A1 account existence and balance value
- peer chaincode query -C mychannel -n balance_transfer -c '{"function":"listAccounts", "Args":[]}'

### Switch to User1 from Org1MSP Admin
- Initialize the User1 U1 account on the user’s behalf.
- Check its existence and balance.

#### export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp

### invoke initAccount for User1 (U1)
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n balance_transfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"initAccount","Args":["U1","200"]}'

### Check the U1 account existence and balance value
- peer chaincode query -C mychannel -n balance_transfer -c '{"function":"listAccounts", "Args":[]}'

### Transfer 100 units from U1 to A1 on behalf of User1 and display the U1 account again.
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n balance_transfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"transfer","Args":["U1","A1", "100"]}'

- peer chaincode query -C mychannel -n balance_transfer -c '{"function":"listAccounts", "Args":[]}'

### Try to transfer 100 units back from A1 to U1, still on behalf of User1
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n balance_transfer --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"transfer","Args":["A1","U1", "100"]}'

- We should receive an error message about an unauthorized access, because only Admin can transfer values from the A1 account.

### Switch back to Admin and check the A1 balance—it should be equal to 250.
- export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
- peer chaincode query -C mychannel -n balance_transfer -c '{"function":"listAccounts", "Args":[]}'

=> [{"balance":250,"id":"A1","owner":"{\"mspid\":\"Org1MSP\",\"id\":\"x509::/C=US/ST=North Carolina/O=Hyperledger/OU=admin/CN=org1admin::/C=US/ST=North Carolina/L=Durham/O=org1.example.com/CN=ca.org1.example.com\"}"}]

## Balance Transfer Chaincode Code View

### initAccount(ctx, id, balance)
const account = {
    id: id,
    owner: this._getTxCreatorUID(ctx),
    balance: accountBalance
}

- await this._accountExists(ctx, account.id)

async _accountExists(ctx, id) {
    const compositeKey = ctx.stub.createCompositeKey(accountObjType, [id]);
    const accountBytes = await ctx.stub.getState(compositeKey);
        eturn accountBytes && accountBytes.length > 0;
}

- await this._putAccount(ctx, account);

async _putAccount(ctx, account) {
    const compositeKey = ctx.stub.createCompositeKey(accountObjType, [account.id]);
    await ctx.stub.putState(compositeKey, Buffer.from(JSON.stringify(account)));
}

### async setBalance(ctx, id, newBalance)
- await this._getAccount(ctx, id);

async _getAccount(ctx, id) {
    const compositeKey = ctx.stub.createCompositeKey(accountObjType, [id]);

    const accountBytes = await ctx.stub.getState(compositeKey);
    if (!accountBytes || accountBytes.length === 0) {
        throw new Error(`the account ${id} does not exist`);
    }

    return JSON.parse(accountBytes.toString());
}

- const txCreator = this._getTxCreatorUID(ctx);

_getTxCreatorUID(ctx) {
    return JSON.stringify({
        mspid: ctx.clientIdentity.getMSPID(),
        id: ctx.clientIdentity.getID()
    });
}

- await this._putAccount(ctx, account);

async _putAccount(ctx, account) {
    const compositeKey = ctx.stub.createCompositeKey(accountObjType, [account.id]);
    await ctx.stub.putState(compositeKey, Buffer.from(JSON.stringify(account)));
}

### async listAccounts(ctx)
- const txCreator = this._getTxCreatorUID(ctx);

- const iteratorPromise = ctx.stub.getStateByPartialCompositeKey(accountObjType, []);

for await (const res of iteratorPromise) {
    const account = JSON.parse(res.value.toString());
    if (account.owner === txCreator) {
        results.push(account);
    }
}

### async transfer(ctx, idFrom, idTo, amount)
- let accountFrom = await this._getAccount(ctx, idFrom);

- const txCreator = this._getTxCreatorUID(ctx);

- let accountTo = await this._getAccount(ctx, idTo);

- accountFrom.balance -= amountToTransfer
- accountTo.balance += amountToTransfer

- await this._putAccount(ctx, accountFrom);
- await this._putAccount(ctx, accountTo);

## Docker Log
- docker logs dev-peer0.org1.example.com-reports_1.0-3e5c30afbe220a0c5d91e6c9fe58a8e40938bd1c59176dcf4e3e728f13ba3e27 -f

## Clear Up
- ./network.sh down