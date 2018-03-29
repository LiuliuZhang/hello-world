Development and testing of chaincode in hyperledger fabric

Required software:

- Bash Shell (Steps were created in the GIT Bash Shell under windows 10)

- Hyperledger Fabric Binaries:

curl -sSL https://goo.gl/5ftp2f | bash

- the modified chaincode-docker-devmode directory from the repository


In the chaincode-docker-devmode directory is a Docker Compose file that spans a minimal network (a peer, an orderer, a chaincode container, a CLI container). In addition, she refers to the directory "docutracker", in the chain code is stored in the form of Go source files.

The steps are:

- Starting the network (the blockchain components)
- Compile and install the code in the blockchain
- Actualization of the code in the individual peer nodes (here only one)
- Execute methods in the blockchain


The network is started by

    docker-compose -f docker-compose-simple.yaml up

taking care that all 4 containers are started (check by docker ps).

Change to the chaincode container where the source code will be compiled and executed:

    winpty docker exec -ti chaincode bash

The prefix "winpty" is only needed in Git Bash to get an interactive shell.
The chaincode must now be built and then installed in the chain:

    go build && CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./docutrackerr

The Chaincoce ID name is a freely selectable name, here mycc, followed by a version number. Then comes the executable file with the chaincode (./docutracker).

This should lead to an issue:
2017-11-20 13:24:14.854 UTC [shim] SetupChaincodeLogging -> INFO 001 Chaincode log level not provided; defaulting to: INFO
2017-11-20 13:24:14.854 UTC [shim] SetupChaincodeLogging -> INFO 002 Chaincode (build level: ) starting up ...

This will install the chaincode..

Now you have to switch to the container 'CLI' (Command line Interpreter):

    winpty docker exec -ti cli bash

The prefix "winpty" is a concession to the used Git Bash shell, to be able to work interactively.
be omitted.

The blockchain can now be checked in the CLI container. First, the chaincode
be installed. To do this, go to the directory "/ opt / gopath / src / docutracker". There then:

    peer chaincode install -p . -n mycc -v 0

This installs the chaincode on the peer node named "mycc" and the version number "0".
Then the chain code has to be instantiated:

    peer chaincode instantiate -n mycc -v 0 -c '{"Args":[]}' -C myc

Now the blockchain is ready for operation with its own code.
For test purposes users can now be created:

    ./createUser.sh

This creates 9 example users with the user IDs "user0" to "user8". These can then be queried from the blockchain:
Either per

     ./queryUsers.sh

or directly by:

     peer chaincode invoke -n mycc -c '{"Args":["queryAllUser"]}' -C myc


Result should be in both cases:

    2017-11-20 13:38:35.855 UTC [msp] GetLocalMSP -> DEBU 001 Returning existing local MSP
    2017-11-20 13:38:35.855 UTC [msp] GetDefaultSigningIdentity -> DEBU 002 Obtaining default signing identity
    2017-11-20 13:38:35.855 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 003 Using default escc
    2017-11-20 13:38:35.855 UTC [chaincodeCmd] checkChaincodeCmdParams -> INFO 004 Using default vscc
    2017-11-20 13:38:35.856 UTC [msp/identity] Sign -> DEBU 005 Sign: plaintext: 0AA9080A6108031A0C08DBB5CBD00510...1A0E0A0C7175657279416C6C55736572
    2017-11-20 13:38:35.856 UTC [msp/identity] Sign -> DEBU 006 Sign: digest: 610728C8652C1F08E9BD62F6CDB729BCB66F55F1473B99E009C27899332757CF
    Query Result: [{"Key":"user0", "Record":{"FirstName":"Chad","LastName":"Palaniuk","SecurityLevel":0}},{"Key":"user1", "Record":{"FirstName":"Wolfgang","LastName":"Petry","SecurityLevel":0}},{"Key":"user2",
    "Record":{"FirstName":"Dude","LastName":"Lebowski","SecurityLevel":0}},{"Key":"user3", "Record":{"FirstName":"Samuel","LastName":"Jackson","SecurityLevel":1}},{"Key":"user4", "Record":{"FirstName":"Wolf","L
    astName":"Gang","SecurityLevel":1}},{"Key":"user5", "Record":{"FirstName":"Jim","LastName":"Pansen","SecurityLevel":1}},{"Key":"user6", "Record":{"FirstName":"Bratislav","LastName":"Methulski","SecurityLeve
    l":2}},{"Key":"user7", "Record":{"FirstName":"Arvo","LastName":"Paert","SecurityLevel":2}},{"Key":"user8", "Record":{"FirstName":"Towa","LastName":"Tei","SecurityLevel":2}}]
    2017-11-20 13:38:35.908 UTC [main] main -> INFO 007 Exiting.....

Attachment of a document:

    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC1", "Pananama Papers", "1", "user0", "0"]}' -C myc

Creates a document with the ID "DOC1" with the title "Panama Papers" in the version "1" with the owner "user0" and the
Security level "0" (readable by all). Call should result:

    ...
    2017-11-20 13:45:18.208 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 0a7 ESCC invoke result: version:1 response:<status:200 message:"OK" > payload:"\n \220\215&\342\026\3208\317\017\\]\324aQ8\326\004 }
    k\226\000h\270B\353c\301\235\366\353\360\022\252\001\n\227\001\022\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\001\022\177\n\004mycc\022w\n\013\n\005user0\022\002\010\002\032h\n\004DOC1\032`{\"Title\":\
    "Pananama Papers\",\"Version\":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user0\",\"SecurityLevel\":0}\032\003\010\310\001\"\t\022\004mycc\032\0010" endorsement:<endorser:"\n\007DEFAULT\022\232\007-----BEGIN
    -----\nMIICizCCAjKgAwIBAgIUBEVwsSx0TmqdbzNwleNBBzoIT0wwCgYIKoZIzj0EAwIw\nfzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh\nbiBGcmFuY2lzY28xHzAdBgNVBAoTFkludGVybmV0IFdpZGdldHMsIEluYy4xDDAK\nB
    gNVBAsTA1dXVzEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMTExMTcwNzAw\nWhcNMTcxMTExMTcwNzAwWjBjMQswCQYDVQQGEwJVUzEXMBUGA1UECBMOTm9ydGgg\nQ2Fyb2xpbmExEDAOBgNVBAcTB1JhbGVpZ2gxGzAZBgNVBAoTEkh5cGVybGVkZ2Vy\nIEZhYnJpY
    zEMMAoGA1UECxMDQ09QMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nHBuKsAO43hs4JGpFfiGMkB/xsILTsOvmN2WmwpsPHZNL6w8HWe3xCPQtdG/XJJvZ\n+C756KEsUBM3yw5PTfku8qOBpzCBpDAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYw\nFAYIKwYBBQUHAwEGC
    CsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFOFC\ndcUZ4es3ltiCgAVDoyLfVpPIMB8GA1UdIwQYMBaAFBdnQj2qnoI/xMUdn1vDmdG1\nnEgQMCUGA1UdEQQeMByCCm15aG9zdC5jb22CDnd3dy5teWhvc3QuY29tMAoGCCqG\nSM49BAMCA0cAMEQCIDf9Hbl4x
    n3z4EwNKmilM9lX2Fq4jWpAaRVB97OmVEeyAiAk\naXzB/jnlU39B7Wws9BIr9c8mSOEPF6VY1uGP+dKV0g==\n-----END -----\n" signature:"0E\002!\000\272B\355O\304^A\342E0\341z0\023AkG\254\226\335\344\223\270<\264Q\212\223\372\2
    17\035\033\002 &u\332/H]\250\270! \331hm<\263:5\210\224\234\273\254\344h\217\177_1\244\000*r" >
    2017-11-20 13:45:18.208 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0a8 Chaincode invoke successful. result: status:200
    2017-11-20 13:45:18.208 UTC [main] main -> INFO 0a9 Exiting.....

Attachment of further example documents by:

    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC1", "Pananama Papers", "1", "user0", "0"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC2", "WikiLeaks", "1", "user1", "0"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC3", "BKA", "1", "user2", "1"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC4", "Deutsche Bundesbank", "1", "user3", "1"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC5", "Schoenhofer", "1", "user4", "1"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC6", "Bitcoin", "1", "user5", "1"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC7", "UBS", "1", "user6", "2"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC8", "Whitehouse", "1", "user7", "2"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC9", "RSA", "1", "user8", "3"]}' -C myc
    peer chaincode invoke -n mycc -c '{"Args":["createDocument", "DOC10", "SPON", "1", "user8", "3"]}' -C myc

Check if documents have been created:

   peer chaincode invoke -n mycc -c '{"Args":["queryAllDocs"]}' -C myc

Should be a listing of all documents:

    ...
    2017-11-20 13:51:59.450 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 0a7 ESCC invoke result: version:1 response:<status:200 message:"OK" payload:"[{\"Key\":\"DOC1\", \"Record\":{\"Title\":\"Pananama Pa
    pers\",\"Version\":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user0\",\"SecurityLevel\":0}},{\"Key\":\"DOC2\", \"Record\":{\"Title\":\"WikiLeaks\",\"Version\":1,\"Owner\":\"user1\",\"CurrentOwner\":\"user1\",
    \"SecurityLevel\":0}},{\"Key\":\"DOC7\", \"Record\":{\"Title\":\"UBS\",\"Version\":1,\"Owner\":\"user6\",\"CurrentOwner\":\"user6\",\"SecurityLevel\":2}},{\"Key\":\"DOC9\", \"Record\":{\"Title\":\"RSA\",\"V
    ersion\":1,\"Owner\":\"user7\",\"CurrentOwner\":\"user7\",\"SecurityLevel\":3}}]" > payload:"\n \233NF\236U\367{\203\317\254\\Z\230\317\003a\276\217\214J\371\222\010\362S\261r[y*\207\256\022\307\004\nf\022\
    024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\001\022N\n\004mycc\022F\022D\n\004DOC0\022\010DOC99999\030\001\"0\n\n\n\004DOC1\022\002\010\007\n\n\n\004DOC2\022\002\010\010\n\n\n\004DOC7\022\002\010\n\n\n\
    n\004DOC9\022\002\010\t\032\321\003\010\310\001\032\313\003[{\"Key\":\"DOC1\", \"Record\":{\"Title\":\"Pananama Papers\",\"Version\":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user0\",\"SecurityLevel\":0}},{\
    "Key\":\"DOC2\", \"Record\":{\"Title\":\"WikiLeaks\",\"Version\":1,\"Owner\":\"user1\",\"CurrentOwner\":\"user1\",\"SecurityLevel\":0}},{\"Key\":\"DOC7\", \"Record\":{\"Title\":\"UBS\",\"Version\":1,\"Owner
    \":\"user6\",\"CurrentOwner\":\"user6\",\"SecurityLevel\":2}},{\"Key\":\"DOC9\", \"Record\":{\"Title\":\"RSA\",\"Version\":1,\"Owner\":\"user7\",\"CurrentOwner\":\"user7\",\"SecurityLevel\":3}}]\"\t\022\004
    mycc\032\0010" endorsement:<endorser:"\n\007DEFAULT\022\232\007-----BEGIN -----\nMIICizCCAjKgAwIBAgIUBEVwsSx0TmqdbzNwleNBBzoIT0wwCgYIKoZIzj0EAwIw\nfzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAc
    TDVNh\nbiBGcmFuY2lzY28xHzAdBgNVBAoTFkludGVybmV0IFdpZGdldHMsIEluYy4xDDAK\nBgNVBAsTA1dXVzEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMTExMTcwNzAw\nWhcNMTcxMTExMTcwNzAwWjBjMQswCQYDVQQGEwJVUzEXMBUGA1UECBMOTm9ydGgg\nQ
    2Fyb2xpbmExEDAOBgNVBAcTB1JhbGVpZ2gxGzAZBgNVBAoTEkh5cGVybGVkZ2Vy\nIEZhYnJpYzEMMAoGA1UECxMDQ09QMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nHBuKsAO43hs4JGpFfiGMkB/xsILTsOvmN2WmwpsPHZNL6w8HWe3xCPQtdG/XJJvZ\n+C756KEsU
    BM3yw5PTfku8qOBpzCBpDAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYw\nFAYIKwYBBQUHAwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFOFC\ndcUZ4es3ltiCgAVDoyLfVpPIMB8GA1UdIwQYMBaAFBdnQj2qnoI/xMUdn1vDmdG1\nnEgQMCUGA1UdEQQeM
    ByCCm15aG9zdC5jb22CDnd3dy5teWhvc3QuY29tMAoGCCqG\nSM49BAMCA0cAMEQCIDf9Hbl4xn3z4EwNKmilM9lX2Fq4jWpAaRVB97OmVEeyAiAk\naXzB/jnlU39B7Wws9BIr9c8mSOEPF6VY1uGP+dKV0g==\n-----END -----\n" signature:"0E\002!\000\274\
    365qa:-i\017\027\273\3622hE\343\307\347\000g\362J\2444\233t=\227d\346\003^\233\002 ,\336e\023\002\352\003\201\033\276]\236U\337\304\230\356\002D\317\2405\345%\311\252s\364\320\247\025/" >
    2017-11-20 13:51:59.450 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0a8 Chaincode invoke successful. result: status:200 payload:"[{\"Key\":\"DOC1\", \"Record\":{\"Title\":\"Pananama Papers\",\"Version
    \":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user0\",\"SecurityLevel\":0}},{\"Key\":\"DOC2\", \"Record\":{\"Title\":\"WikiLeaks\",\"Version\":1,\"Owner\":\"user1\",\"CurrentOwner\":\"user1\",\"SecurityLevel\
    ":0}},{\"Key\":\"DOC7\", \"Record\":{\"Title\":\"UBS\",\"Version\":1,\"Owner\":\"user6\",\"CurrentOwner\":\"user6\",\"SecurityLevel\":2}},{\"Key\":\"DOC9\", \"Record\":{\"Title\":\"RSA\",\"Version\":1,\"Own
    er\":\"user7\",\"CurrentOwner\":\"user7\",\"SecurityLevel\":3}}]"
    2017-11-20 13:51:59.451 UTC [main] main -> INFO 0a9 Exiting.....

Here are some demonstrations of the implemented methods. In general, these are called by the command "peer chaincode invoke"
executed. Queries are generally made with the command "peer chaincode query".

The documents are in their original state, i. all documents are with your owners and can be borrowed,
provided the lender has a security level equal to or greater than that of the document to be lent.


Loan document "DOC0":

    peer chaincode invoke -n mycc -c '{"Args":["lendDocument", "DOC1", "user2"]}' -C myc

Generates the following output:

    ...
    2017-11-20 15:53:48.278 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 0a7 ESCC invoke result: version:1 response:<status:200 message:"OK" > payload:"\n \243\206\221\316c\234=\370\252\304\320b\241,/}\330
    \006\262\201~D\255\362\204\263b)>o\177\017\022\270\001\n\245\001\022\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\001\022\214\001\n\004mycc\022\203\001\n\n\n\004DOC1\022\002\010\007\n\013\n\005user2\022\
    002\010\003\032h\n\004DOC1\032`{\"Title\":\"Pananama Papers\",\"Version\":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user2\",\"SecurityLevel\":0}\032\003\010\310\001\"\t\022\004mycc\032\0010" endorsement:<end
    orser:"\n\007DEFAULT\022\232\007-----BEGIN -----\nMIICizCCAjKgAwIBAgIUBEVwsSx0TmqdbzNwleNBBzoIT0wwCgYIKoZIzj0EAwIw\nfzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh\nbiBGcmFuY2lzY28xHzAdBgNV
    BAoTFkludGVybmV0IFdpZGdldHMsIEluYy4xDDAK\nBgNVBAsTA1dXVzEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMTExMTcwNzAw\nWhcNMTcxMTExMTcwNzAwWjBjMQswCQYDVQQGEwJVUzEXMBUGA1UECBMOTm9ydGgg\nQ2Fyb2xpbmExEDAOBgNVBAcTB1JhbGVp
    Z2gxGzAZBgNVBAoTEkh5cGVybGVkZ2Vy\nIEZhYnJpYzEMMAoGA1UECxMDQ09QMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nHBuKsAO43hs4JGpFfiGMkB/xsILTsOvmN2WmwpsPHZNL6w8HWe3xCPQtdG/XJJvZ\n+C756KEsUBM3yw5PTfku8qOBpzCBpDAOBgNVHQ8B
    Af8EBAMCBaAwHQYDVR0lBBYw\nFAYIKwYBBQUHAwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFOFC\ndcUZ4es3ltiCgAVDoyLfVpPIMB8GA1UdIwQYMBaAFBdnQj2qnoI/xMUdn1vDmdG1\nnEgQMCUGA1UdEQQeMByCCm15aG9zdC5jb22CDnd3dy5teWhv
    c3QuY29tMAoGCCqG\nSM49BAMCA0cAMEQCIDf9Hbl4xn3z4EwNKmilM9lX2Fq4jWpAaRVB97OmVEeyAiAk\naXzB/jnlU39B7Wws9BIr9c8mSOEPF6VY1uGP+dKV0g==\n-----END -----\n" signature:"0E\002!\000\201\357t\201\265\214w\322\364z\374\
    230\373\244\0001\257\325\330\014\364S\216mNT\276\375\261}\005\233\002 ?\334\302\364k\tTi\235\001?D\274\243\277\244,\371\276\033\311\237\250\"\rZ\313\346\211\374\260\010" >
    2017-11-20 15:53:48.279 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0a8 Chaincode invoke successful. result: status:200
    2017-11-20 15:53:48.279 UTC [main] main -> INFO 0a9 Exiting.....

Query status of documents:

    peer chaincode invoke -n mycc -c '{"Args":["queryAllDocs"]}' -C myc

Result: "DOC1" now has CurrentOwner "user2":

    2017-11-20 15:55:11.960 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 0a7 ESCC invoke result: version:1 response:<status:200 message:"OK" payload:"[{\"Key\":\"DOC1\", \"Record\":{\"Title\":\"Pananama Pa
    pers\",\"Version\":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user2\",\"SecurityLevel\":0}},{\"Key\":\"DOC2\", \"Record\":{\"Title\":\"WikiLeaks\",\"Version\":1,\"Owner\":\"user1\",\"CurrentOwner\":\"user1\",
    \"SecurityLevel\":0}},{\"Key\":\"DOC7\", \"Record\":{\"Title\":\"UBS\",\"Version\":1,\"Owner\":\"user6\",\"CurrentOwner\":\"user6\",\"SecurityLevel\":2}},{\"Key\":\"DOC9\", \"Record\":{\"Title\":\"RSA\",\"V
    ersion\":1,\"Owner\":\"user7\",\"CurrentOwner\":\"user7\",\"SecurityLevel\":3}}]" > payload:"\n \225{\237\215\316\236\303\260[K{\3039@\027\0260^\373#[\277\230\000\006,\354\320\315\332*\316\022\307\004\nf\02
    2\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\001\022N\n\004mycc\022F\022D\n\004DOC0\022\010DOC99999\030\001\"0\n\n\n\004DOC1\022\002\010\017\n\n\n\004DOC2\022\002\010\010\n\n\n\004DOC7\022\002\010\n\n\
    n\n\004DOC9\022\002\010\t\032\321\003\010\310\001\032\313\003[{\"Key\":\"DOC1\", \"Record\":{\"Title\":\"Pananama Papers\",\"Version\":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user2\",\"SecurityLevel\":0}},
    {\"Key\":\"DOC2\", \"Record\":{\"Title\":\"WikiLeaks\",\"Version\":1,\"Owner\":\"user1\",\"CurrentOwner\":\"user1\",\"SecurityLevel\":0}},{\"Key\":\"DOC7\", \"Record\":{\"Title\":\"UBS\",\"Version\":1,\"Own
    er\":\"user6\",\"CurrentOwner\":\"user6\",\"SecurityLevel\":2}},{\"Key\":\"DOC9\", \"Record\":{\"Title\":\"RSA\",\"Version\":1,\"Owner\":\"user7\",\"CurrentOwner\":\"user7\",\"SecurityLevel\":3}}]\"\t\022\0
    04mycc\032\0010" endorsement:<endorser:"\n\007DEFAULT\022\232\007-----BEGIN -----\nMIICizCCAjKgAwIBAgIUBEVwsSx0TmqdbzNwleNBBzoIT0wwCgYIKoZIzj0EAwIw\nfzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVB
    AcTDVNh\nbiBGcmFuY2lzY28xHzAdBgNVBAoTFkludGVybmV0IFdpZGdldHMsIEluYy4xDDAK\nBgNVBAsTA1dXVzEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMTExMTcwNzAw\nWhcNMTcxMTExMTcwNzAwWjBjMQswCQYDVQQGEwJVUzEXMBUGA1UECBMOTm9ydGgg\
    nQ2Fyb2xpbmExEDAOBgNVBAcTB1JhbGVpZ2gxGzAZBgNVBAoTEkh5cGVybGVkZ2Vy\nIEZhYnJpYzEMMAoGA1UECxMDQ09QMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nHBuKsAO43hs4JGpFfiGMkB/xsILTsOvmN2WmwpsPHZNL6w8HWe3xCPQtdG/XJJvZ\n+C756KE
    sUBM3yw5PTfku8qOBpzCBpDAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYw\nFAYIKwYBBQUHAwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFOFC\ndcUZ4es3ltiCgAVDoyLfVpPIMB8GA1UdIwQYMBaAFBdnQj2qnoI/xMUdn1vDmdG1\nnEgQMCUGA1UdEQQ
    eMByCCm15aG9zdC5jb22CDnd3dy5teWhvc3QuY29tMAoGCCqG\nSM49BAMCA0cAMEQCIDf9Hbl4xn3z4EwNKmilM9lX2Fq4jWpAaRVB97OmVEeyAiAk\naXzB/jnlU39B7Wws9BIr9c8mSOEPF6VY1uGP+dKV0g==\n-----END -----\n" signature:"0D\002 \n\314j
    \261d\364\177A\330\204\2323\216\236\311x\356\027\026\034\332\332\207\333\323\370\020\227u\244\217\260\002 ~\363%\357\242%\247\211\271d\033\353\262\3166\034\337\217N)*D#\311\327\362\214\277j\240\rM" >
    2017-11-20 15:55:11.960 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0a8 Chaincode invoke successful. result: status:200 payload:"[{\"Key\":\"DOC1\", \"Record\":{\"Title\":\"Pananama Papers\",\"Version
    \":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user2\",\"SecurityLevel\":0}},{\"Key\":\"DOC2\", \"Record\":{\"Title\":\"WikiLeaks\",\"Version\":1,\"Owner\":\"user1\",\"CurrentOwner\":\"user1\",\"SecurityLevel\
    ":0}},{\"Key\":\"DOC7\", \"Record\":{\"Title\":\"UBS\",\"Version\":1,\"Owner\":\"user6\",\"CurrentOwner\":\"user6\",\"SecurityLevel\":2}},{\"Key\":\"DOC9\", \"Record\":{\"Title\":\"RSA\",\"Version\":1,\"Own
    er\":\"user7\",\"CurrentOwner\":\"user7\",\"SecurityLevel\":3}}]"
    2017-11-20 15:55:11.961 UTC [main] main -> INFO 0a9 Exiting.....


Returned document by a wrong user "user3" who did not borrow the document:

     peer chaincode invoke -n mycc -c '{"Args":["returnDocument", "DOC1", "user3"]}' -C myc

Output:

    ...
    2017-11-20 15:57:16.425 UTC [msp/identity] Sign -> DEBU 0a4 Sign: digest: 1045A3FADF57447A4339028151D9B576100FB8A64CE428AB9AC1CD26CCAD863A
    Error: Error endorsing invoke: rpc error: code = Unknown desc = chaincode error (status: 500, message: WARNING: someone else brought the document back!) - <nil>
    ...

Return document by correct user "user2":

     peer chaincode invoke -n mycc -c '{"Args":["returnDocument", "DOC1", "user2"]}' -C myc

Output (Owner and CurrentOwner are again "user0"):

    ...
    2017-11-20 15:58:38.272 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 0a7 ESCC invoke result: version:1 response:<status:200 message:"OK" > payload:"\n Bh%\322\253&\024?LR\310\274\353u\345\200\334\263\0
    27\355W\375\006\274\315l\003\304\025{>\327\022\251\001\n\226\001\022\024\n\004lscc\022\014\n\n\n\004mycc\022\002\010\001\022~\n\004mycc\022v\n\n\n\004DOC1\022\002\010\017\032h\n\004DOC1\032`{\"Title\":\"Pan
    anama Papers\",\"Version\":1,\"Owner\":\"user0\",\"CurrentOwner\":\"user0\",\"SecurityLevel\":0}\032\003\010\310\001\"\t\022\004mycc\032\0010" endorsement:<endorser:"\n\007DEFAULT\022\232\007-----BEGIN ----
    -\nMIICizCCAjKgAwIBAgIUBEVwsSx0TmqdbzNwleNBBzoIT0wwCgYIKoZIzj0EAwIw\nfzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh\nbiBGcmFuY2lzY28xHzAdBgNVBAoTFkludGVybmV0IFdpZGdldHMsIEluYy4xDDAK\nBgNVB
    AsTA1dXVzEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMTExMTcwNzAw\nWhcNMTcxMTExMTcwNzAwWjBjMQswCQYDVQQGEwJVUzEXMBUGA1UECBMOTm9ydGgg\nQ2Fyb2xpbmExEDAOBgNVBAcTB1JhbGVpZ2gxGzAZBgNVBAoTEkh5cGVybGVkZ2Vy\nIEZhYnJpYzEMM
    AoGA1UECxMDQ09QMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nHBuKsAO43hs4JGpFfiGMkB/xsILTsOvmN2WmwpsPHZNL6w8HWe3xCPQtdG/XJJvZ\n+C756KEsUBM3yw5PTfku8qOBpzCBpDAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0lBBYw\nFAYIKwYBBQUHAwEGCCsGA
    QUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFOFC\ndcUZ4es3ltiCgAVDoyLfVpPIMB8GA1UdIwQYMBaAFBdnQj2qnoI/xMUdn1vDmdG1\nnEgQMCUGA1UdEQQeMByCCm15aG9zdC5jb22CDnd3dy5teWhvc3QuY29tMAoGCCqG\nSM49BAMCA0cAMEQCIDf9Hbl4xn3z4
    EwNKmilM9lX2Fq4jWpAaRVB97OmVEeyAiAk\naXzB/jnlU39B7Wws9BIr9c8mSOEPF6VY1uGP+dKV0g==\n-----END -----\n" signature:"0D\002 J\227nC:\333\220\223\211;\363\371H\027\376R\033\206p@R\216\255\237d\235k\371L\350\320\0
    14\002 o\222\310\373z\254y{\227\273\327xk%\254\352\006\013\030#\013om\355\220\314\026\220\302\333ik" >
    2017-11-20 15:58:38.273 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0a8 Chaincode invoke successful. result: status:200
    2017-11-20 15:58:38.275 UTC [main] main -> INFO 0a9 Exiting.....

Loan document "DOC7" with Security Level "2" by "user0" with SecurityLevel "0" (forbidden):

    peer chaincode invoke -n mycc -c '{"Args":["lendDocument", "DOC7", "user0"]}' -C myc

Output:

    ...
    2017-11-20 16:01:47.658 UTC [msp/identity] Sign -> DEBU 0a4 Sign: digest: 8A031FC2514CB9928C77BE6259B1439982AF422C5C58DFCB4A8F3F3FC372438F
    Error: Error endorsing invoke: rpc error: code = Unknown desc = chaincode error (status: 500, message: Security Levels are not compatible.) - <nil>
    ...


Query of all users:

    peer chaincode invoke -n mycc -c '{"Args":["queryAllUser"]}' -C myc

Output:

    ...
    2017-11-20 16:05:06.548 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> DEBU 0a7 ESCC invoke result: version:1 response:<status:200 message:"OK" payload:"[{\"Key\":\"user0\", \"Record\":{\"FirstName\":\"Chad\"
    ,\"LastName\":\"Palaniuk\",\"SecurityLevel\":0}},{\"Key\":\"user1\", \"Record\":{\"FirstName\":\"Wolfgang\",\"LastName\":\"Petry\",\"SecurityLevel\":0}},{\"Key\":\"user2\", \"Record\":{\"FirstName\":\"Dude\
    ",\"LastName\":\"Lebowski\",\"SecurityLevel\":0}},{\"Key\":\"user3\", \"Record\":{\"FirstName\":\"Samuel\",\"LastName\":\"Jackson\",\"SecurityLevel\":1}},{\"Key\":\"user4\", \"Record\":{\"FirstName\":\"Wolf
    \",\"LastName\":\"Gang\",\"SecurityLevel\":1}},{\"Key\":\"user5\", \"Record\":{\"FirstName\":\"Jim\",\"LastName\":\"Pansen\",\"SecurityLevel\":1}},{\"Key\":\"user6\", \"Record\":{\"FirstName\":\"Bratislav\"
    ,\"LastName\":\"Methulski\",\"SecurityLevel\":2}},{\"Key\":\"user7\", \"Record\":{\"FirstName\":\"Arvo\",\"LastName\":\"Paert\",\"SecurityLevel\":2}},{\"Key\":\"user8\", \"Record\":{\"FirstName\":\"Towa\",\
    "LastName\":\"Tei\",\"SecurityLevel\":2}}]" > payload:"\n 1\245\243\\1\017\207\224|\244V\344\002\336A\326S+\t\2670C]\360\210\330\215\304T\222\302J\022\330\007\n\270\001\022\024\n\004lscc\022\014\n\n\n\004my
    cc\022\002\010\001\022\237\001\n\004mycc\022\226\001\022\223\001\n\005user0\022\tuser99999\030\001\"}\n\013\n\005user0\022\002\010\002\n\r\n\005user1\022\004\010\002\020\001\n\013\n\005user2\022\002\010\003
    \n\r\n\005user3\022\004\010\003\020\001\n\013\n\005user4\022\002\010\004\n\r\n\005user5\022\004\010\004\020\001\n\013\n\005user6\022\002\010\005\n\r\n\005user7\022\004\010\005\020\001\n\013\n\005user8\022\0
    02\010\006\032\217\006\010\310\001\032\211\006[{\"Key\":\"user0\", \"Record\":{\"FirstName\":\"Chad\",\"LastName\":\"Palaniuk\",\"SecurityLevel\":0}},{\"Key\":\"user1\", \"Record\":{\"FirstName\":\"Wolfgang
    \",\"LastName\":\"Petry\",\"SecurityLevel\":0}},{\"Key\":\"user2\", \"Record\":{\"FirstName\":\"Dude\",\"LastName\":\"Lebowski\",\"SecurityLevel\":0}},{\"Key\":\"user3\", \"Record\":{\"FirstName\":\"Samuel\
    ",\"LastName\":\"Jackson\",\"SecurityLevel\":1}},{\"Key\":\"user4\", \"Record\":{\"FirstName\":\"Wolf\",\"LastName\":\"Gang\",\"SecurityLevel\":1}},{\"Key\":\"user5\", \"Record\":{\"FirstName\":\"Jim\",\"La
    stName\":\"Pansen\",\"SecurityLevel\":1}},{\"Key\":\"user6\", \"Record\":{\"FirstName\":\"Bratislav\",\"LastName\":\"Methulski\",\"SecurityLevel\":2}},{\"Key\":\"user7\", \"Record\":{\"FirstName\":\"Arvo\",
    \"LastName\":\"Paert\",\"SecurityLevel\":2}},{\"Key\":\"user8\", \"Record\":{\"FirstName\":\"Towa\",\"LastName\":\"Tei\",\"SecurityLevel\":2}}]\"\t\022\004mycc\032\0010" endorsement:<endorser:"\n\007DEFAULT
    \022\232\007-----BEGIN -----\nMIICizCCAjKgAwIBAgIUBEVwsSx0TmqdbzNwleNBBzoIT0wwCgYIKoZIzj0EAwIw\nfzELMAkGA1UEBhMCVVMxEzARBgNVBAgTCkNhbGlmb3JuaWExFjAUBgNVBAcTDVNh\nbiBGcmFuY2lzY28xHzAdBgNVBAoTFkludGVybmV0IFdp
    ZGdldHMsIEluYy4xDDAK\nBgNVBAsTA1dXVzEUMBIGA1UEAxMLZXhhbXBsZS5jb20wHhcNMTYxMTExMTcwNzAw\nWhcNMTcxMTExMTcwNzAwWjBjMQswCQYDVQQGEwJVUzEXMBUGA1UECBMOTm9ydGgg\nQ2Fyb2xpbmExEDAOBgNVBAcTB1JhbGVpZ2gxGzAZBgNVBAoTEkh5
    cGVybGVkZ2Vy\nIEZhYnJpYzEMMAoGA1UECxMDQ09QMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE\nHBuKsAO43hs4JGpFfiGMkB/xsILTsOvmN2WmwpsPHZNL6w8HWe3xCPQtdG/XJJvZ\n+C756KEsUBM3yw5PTfku8qOBpzCBpDAOBgNVHQ8BAf8EBAMCBaAwHQYDVR0l
    BBYw\nFAYIKwYBBQUHAwEGCCsGAQUFBwMCMAwGA1UdEwEB/wQCMAAwHQYDVR0OBBYEFOFC\ndcUZ4es3ltiCgAVDoyLfVpPIMB8GA1UdIwQYMBaAFBdnQj2qnoI/xMUdn1vDmdG1\nnEgQMCUGA1UdEQQeMByCCm15aG9zdC5jb22CDnd3dy5teWhvc3QuY29tMAoGCCqG\nSM
    49BAMCA0cAMEQCIDf9Hbl4xn3z4EwNKmilM9lX2Fq4jWpAaRVB97OmVEeyAiAk\naXzB/jnlU39B7Wws9BIr9c8mSOEPF6VY1uGP+dKV0g==\n-----END -----\n" signature:"0E\002!\000\376*\307n-q5\273\326\342\327-A,\346\342B\231\033~k\365\
    324\341\313\206J\317%`\317\316\002 K\330\260\347\\\017`@\234\277yh\274o(I[\302h\307\351\224\033\3424\237\030}\"\232?\223" >
    2017-11-20 16:05:06.549 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 0a8 Chaincode invoke successful. result: status:200 payload:"[{\"Key\":\"user0\", \"Record\":{\"FirstName\":\"Chad\",\"LastName\":\"
    Palaniuk\",\"SecurityLevel\":0}},{\"Key\":\"user1\", \"Record\":{\"FirstName\":\"Wolfgang\",\"LastName\":\"Petry\",\"SecurityLevel\":0}},{\"Key\":\"user2\", \"Record\":{\"FirstName\":\"Dude\",\"LastName\":\
    "Lebowski\",\"SecurityLevel\":0}},{\"Key\":\"user3\", \"Record\":{\"FirstName\":\"Samuel\",\"LastName\":\"Jackson\",\"SecurityLevel\":1}},{\"Key\":\"user4\", \"Record\":{\"FirstName\":\"Wolf\",\"LastName\":
    \"Gang\",\"SecurityLevel\":1}},{\"Key\":\"user5\", \"Record\":{\"FirstName\":\"Jim\",\"LastName\":\"Pansen\",\"SecurityLevel\":1}},{\"Key\":\"user6\", \"Record\":{\"FirstName\":\"Bratislav\",\"LastName\":\"
    Methulski\",\"SecurityLevel\":2}},{\"Key\":\"user7\", \"Record\":{\"FirstName\":\"Arvo\",\"LastName\":\"Paert\",\"SecurityLevel\":2}},{\"Key\":\"user8\", \"Record\":{\"FirstName\":\"Towa\",\"LastName\":\"Te
    i\",\"SecurityLevel\":2}}]"
    2017-11-20 16:05:06.549 UTC [main] main -> INFO 0a9 Exiting.....


