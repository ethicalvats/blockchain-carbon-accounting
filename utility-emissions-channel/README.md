# utility-emissions-channel

# Setting up the app

## Set up AWS credentials

1. Create a new file in utility-emissions-channel/chaincode/node/lib called aws-config.js

2. Fill it out with your credentials for AWS based on the fields requested in aws-config.js.template.

## Set up S3

### Running Locally

1. Install serverless:

```bash
npm install -g serverless
```

2. cd blockchain-carbon-accounting/utility-emissions-channel/typescript_app

```bash
serverless s3 start
```

3. Be sure that S3_LOCAL is set to true in aws-config.js

### Running Remotely

1. Set up an S3 bucket named blockchain-carbon-accounting on your AWS account

2. Set S3_LOCAL to false in aws-config.js

## Set up the dynamodb

### Running Locally

1. From the blockchain-carbon-accounting/utility-emissions-channel/docker-compose-setup directory, start dynamodb:

```bash
./scripts/startDynamo.sh
```

2. Seed the db, see eGrid Loader documentation in utility-emissions-channel/egrid-data-loader/README.md

### Running remotely

1. Set AWS_ENDPOINT to the endpoint of your remote dynamodb

2. Seed the db, see eGrid Loader documentation in utility-emissions-channel/egrid-data-loader/README.md

## Get the blockchain network up and running

1. Install Prerequisites (https://hyperledger-fabric.readthedocs.io/en/release-2.2/prereqs.html)
2. Setup AWS credentials in `chaincode/node/lib/aws-config.js`::
   exports.AWS_ACCESS_KEY_ID = 'your_access_key';
   exports.AWS_SECRET_ACCESS_KEY = 'your_secret_key';
3. cd to `docker-compose-setup`
4. Start network: Run `./network.sh up createChannel`
5. Deploy and invoke `emissionscontract` chaincode (JS): Run `./network.sh deployCC`
6. (optional) Start Hyperledger Explorer (http://localhost:8080, username: exploreradmin, pw: exploreradminpw): Run `./network.sh startBlockchainExplorer`

##### Play with the chaincode and have a look at the blockchain-explorer.

1. Invoke chaincode with peer binaries

```shell
# Record emission to utilityemissionchannel
./scripts/invokeChaincode.sh '{"function":"'recordEmissions'","Args":["11208","MyCompany","2018-06-01","2018-06-30","150","KWH"]}' 1 2 3

# Query emission data
./scripts/invokeChaincode.sh '{"function":"'getEmissionsData'","Args":["11208","MyCompany","2018-06-01","2018-06-30"]}' 1
```

2. Invoke chaincode with nodeJS
   2.1. cd to `docker-compose-setup/application`
   2.2. Install node modules: RUN `npm i`
   2.3. Create wallet: Run `node invokeChaincodeManually/createWallet.js`
   2.2. Invoke chaincode:

```shell
# Invoke chaincode
node invokeChaincodeManually/invoke.js

# Should print similar output
Wallet path: /Users/robinklemens/Documents/GitHub/blockchain-carbon-accounting/utility-emissions-channel/docker-compose-setup/application/wallet
{"class":"org.hyperledger.blockchain-carbon-accounting.emissionsrecord","key":"\"MediumUtility\":\"MyCompany\":\"2020-06-01\":\"2020-06-30\"","currentState":null,"emissionsAmount":150,"emissionsUom":"TONS","fromDate":"2020-06-01","partyId":"MyCompany","thruDate":"2020-06-30","utilityId":"MediumUtility"}
```

## Start Express server (REST API)

1. cd to `utility-emissions-channel/typescript_app`
2. Install node modules: RUN `npm i`
3. Start express sever: Run `sh start.sh`
4. Go to `http://localhost:9000/api-docs/` to access swagger file from where you can interact with the blockchain.
5. Register org admin of org auditor1, auditor2, and/or auditor (e.g. { "orgName": "auditor1"})
6. Register and enroll user: First register org admin of step 5. Then register user with userId, orgName, and affiliation. (e.g. { "userID": "User8", "orgName": "auitor1", "affiliation": "auditor1.department1"} )
7. Interact with the `emissionscontract` chaincode.
   Note: As of 09/30/2020 the REST API a static, and doesn't include a proper error handling.

## Update emissioncontact Chaincode

1. cd to `docker-compose-setup`
2. Update Chaincode:
   Run `./network.sh deployCC -ccv 'VERSION' -ccs 'SEQUENCE'`
   e.g. update chaincode `emissionscontract` to version 2: `./network.sh deployCC -ccv 2.0 -ccs 2`
3. Check help, if further infomation is needed. Run: `./network.sh -h`

## Stop the blockchain network and remove container

1. cd to `docker-compose-setup`
2. Run `./network.sh down`

#### Stop blockchain explorer

1. cd to `docker-compose-setup`
2. Run `./network.sh stopBlockchainExplorer`

## Troubleshooting

If any error in `Get the blockchain network up and running` please run the commands of `Stop the blockchain network and remove container` and retry starting the network. If you still run into errors open an issue with the error logs, please.

# Deployment

See docs under utility-emissions-channel/docker-compose-setup/DEPLOY.md
