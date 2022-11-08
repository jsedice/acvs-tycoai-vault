# Vault access control example

An example to help the team work with Vault

# Install

```
#Install [Docker](https://docs.docker.com/compose/install/) and docker-compose

apt-get install -y software-properties-common curl gnupg2
curl -fsSL https://apt.releases.hashicorp.com/gpg | apt-key add -
apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
apt-get update && apt-get install -y vault
```

# Step 1: Spin up the vault container
```
docker-compose up -d
```
# Step 2: Configure Vault with a policy, role, secret and access control
```
source source_me.txt vault status
vault policy write nginx nginx.hcl
vault secrets enable -version=2 kv
vault kv put kv/nginx app=nginx username=nginx password=sup4s3cr3t
vault auth enable approle
vault write auth/approle/role/nginx token_policies="nginx"
vault read -format=json auth/approle/role/nginx/role-id | jq -r '.data.role_id' > role_secret_id
vault write -format=json -f auth/approle/role/nginx/secret-id | jq -r '.data.secret_id' >> role_secret_id
```

# Step 3: On new terminal access secret with role
```
export VAULT_ADDR=http://localhost:8200
vault write auth/approle/login role_id=$(sed -n '1p' role_secret_id) secret_id=$(sed -n '2p' role_secret_id)
vault login
vault kv get kv/nginx
```
