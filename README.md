# Vault access control example

An example to help the team work with Vault

# Install

Install [Docker and docker-compose](https://docs.docker.com/compose/install/)

Install a vault cli tool:
```
sudo apt-get install -y software-properties-common curl gnupg2
sudo curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update 
sudo apt-get install -y vault
```

# Step 1: Spin up the vault container
```
docker-compose up -d
```
# Step 2: Configure Vault with a policy, role, secret and access control
```
source source_me.txt 
vault status
vault policy write nginx nginx.hcl
vault policy write couchdb couchdb.hcl
vault secrets enable -version=2 kv
vault kv put kv/nginx app=nginx username=nginx password=sup4s3cr3t
vault kv put kv/couchdb app=couchdb username=couchdb password=fakepassword
vault auth enable approle
vault write auth/approle/role/nginx token_policies="nginx"
vault write auth/approle/role/couchdb token_policies="couchdb"
vault read -format=json auth/approle/role/nginx/role-id | jq -r '.data.role_id' > nginx_role_secret_id
vault write -format=json -f auth/approle/role/nginx/secret-id | jq -r '.data.secret_id' >> nginx_role_secret_id
```

# Step 3: On new terminal access secret with role
```
export VAULT_ADDR=http://localhost:8200
vault write auth/approle/login role_id=$(sed -n '1p' nginx_role_secret_id) secret_id=$(sed -n '2p' nginx_role_secret_id)
vault login
```

# Step 4: Read secrets from vault
```
vault kv get kv/nginx
```