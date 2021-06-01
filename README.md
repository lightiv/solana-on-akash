# solana-on-akash

### This repo is for setting up a Solana, v1.6.10, node deployed to Akash Network's DeCloud.  I am using a non-native method of accessing the node via SSL to run the Solana Setup script I created.  I believe this provides the greates flexibility and insight in to your deployment.  You also know exactly what is in your deployment.  

### Step One - Setup The Environment

``` 
Setup Environment:

AKASH_NODE=http://135.181.60.250:26657  
AKASH_CHAIN_ID=akashnet-2  
AKASH_ACCOUNT_ADDRESS=<YOUR_FUNDING_ADDRESS>  
AKASH_KEYRING_BACKEND=os  
AKASH_KEY_NAME=<YOUR_FUNDING_ADDRESS_KEY_NAME>  
``` 

### Step Two - The deploy.yml file.  I have already created this for you.  ** You will need to personalize to your information.  Any thing with <ALL_CAPS>

### Step Three - Create a certificate for <YOUR_FUNDING_ADDRESS>. You only do this once and it is stored in your ~/.akash directory.  You can copy it to a new machine if needed.

```  
akash tx cert create client --chain-id $AKASH_CHAIN_ID --keyring-backend $AKASH_KEYRING_BACKEND \ 
    --from $AKASH_KEY_NAME --node $AKASH_NODE --fees 5000uakt  
```

### Step Four - Deploy Your Request

```  
akash tx deployment create deploy.yml --from $AKASH_KEY_NAME --node $AKASH_NODE \
    --chain-id $AKASH_CHAIN_ID --fees 5000uakt -y  
```  

### Step Five - Set The Following Variables

``` 
AKASH_DSEQ=<FROM_THE_OUTPUT_OF_STEP_FOUR>    
AKASH_OSEQ=1  
AKASH_GSEQ=1  
```  

### Step Six - View Bids

```  
akash query market bid list --owner $AKASH_ACCOUNT_ADDRESS --node $AKASH_NODE --dseq $AKASH_DSEQ -o text  
```  

### Step Seven - Select A Provider

Set the folloiwng environment setting  

```  
AKASH_PROVIDER=<PROVIDER_ADDRESS_SELECTED_FROM_STEP_SIX>  
```  

### Step Eight - Create Your Lease 

```  
akash tx market lease create --from $AKASH_KEY_NAME --fees 5000uakt  --owner $AKASH_ACCOUNT_ADDRESS --dseq $AKASH_DSEQ --gseq $AKASH_GSEQ --oseq $AKASH_OSEQ --provider $AKASH_PROVIDER --chain-id $AKASH_CHAIN_ID --node $AKASH_NODE  
```  

### Step Nine - View Lease

This is to make sure everything has worked.  
```  
akash query market lease list --owner $AKASH_ACCOUNT_ADDRESS --node $AKASH_NODE --dseq $AKASH_DSEQ -o text  
```  

### Step Ten - Upload Manifest

This fives the Provider the necessary information to deploy Solana.  There will be NO out put from this command.
```  
akash provider send-manifest deploy.yml --node $AKASH_NODE --dseq $AKASH_DSEQ --provider $AKASH_PROVIDER --home ~/.akash --from $AKASH_KEY_NAME  
```  

### Step Eleven - View Access urls  

If these are listed you have successfully deploy Solana

```  
akash provider lease-status --node $AKASH_NODE --home ~/.akash --dseq $AKASH_DSEQ --from $AKASH_KEY_NAME --provider $AKASH_PROVIDER  
```  

### Step Twelve - Correct And The setup-solana.sh file

Here you will need the connection randomly assigned port for the SSH.  You will get that from above or by running:  
```  
akash provider lease-status --node $AKASH_NODE --home ~/.akash --dseq $AKASH_DSEQ --from $AKASH_KEY_NAME --provider $AKASH_PROVIDER
```

Connect to your Solana instance:  
```  
ssh -p <SSH PORT> -i ~/.ssh/<YOUR SSH PRIVATE KEY> <USERNAME>@<YOUR VPS PUBLIC IP ADDRESS>  
```  
Once connected continue with the following:

```  
cd /root  
  
cat setu* > setup-solana.sh  
  
nano setup-solana.sh  
```  
Paste the following into the above nano window.  

```  
echo "Install Curl"  
echo y|apt install curl  
  
echo "Install Solana's install Tool"  
sh -c "$(curl -sSfL https://release.solana.com/v1.6.10/install)"  
export PATH="/root/.local/share/solana/install/active_release/bin:$PATH"  
solana --version  
solana cluster-version  
echo "The above two version should match!"  
  
echo "Create a File System wallet.  Not recommends for storing large amounts of SOL!  Use paper or Leger based wallets"  
mkdir ~/<YOUR_NAME>-solana-wallet  
solana-keygen new --outfile ~/<YOUR_NAME>-solana-wallet/<YOUR_NAME>.json > ~/mnumonic.txt  
  
echo "Your public key/address is saved to ~/mnumonic.txt."  
solana-keygen pubkey ~/<YOUR_NAME>-solana-wallet/<YOUR_NAME>.json  
  
echo "Verifying you hold the private key for your address"  
solana-keygen verify pubkey ~/<YOUR_NAME>-solana-wallet/<YOUR_NAME>.json  
  
echo "Configure and Connect to the Devnet Cluster"  
solana config set --url http://api.devnet.solana.com  
solana transaction-count  
solana-sys-tuner --user $(whoami) > sys-tuner.log 2>&1 &  
solana-gossip spy --entrypoint entrypoint.devnet.solana.com:8001  
```  

### Step Thirteen - Start The Solana Install  

Type the following:  
```  
. setup-solana.sh  
```
The script will run.  When you are asked to enter a keyring password, ** DO NOT MESS UP. ** The script will continue and your will have to open another terminal window to manually create the keys.  

Do not work, just run the following:  
```  
echo "Create a File System wallet.  Not recommends for storing large amounts of SOL!  Use paper or Leger based wallets"  
mkdir ~/<YOUR_NAME>-solana-wallet  
solana-keygen new --outfile ~/<YOUR_NAME>-solana-wallet/<YOUR_NAME>.json > ~/mnumonic.txt  
  
echo "Your public key/address is saved to ~/mnumonic.txt."  
solana-keygen pubkey ~/<YOUR_NAME>-solana-wallet/<YOUR_NAME>.json 
```  
  
# If everything went well your are running a Solana node.

![SkyNet | Validators](http://paullovette.com/wp-content/uploads/2021/06/solana-on-akash.jpg)
