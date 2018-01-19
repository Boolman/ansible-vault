Inspired by https://github.com/AvalancheDev/Hashicorp-Vault-Ansible

# Prereq
Required a working consul, with ACL support and a existing token

Create the vault-token in consul
```
cat << EOF > vault
{
  "ID": "generated uuid",
  "Name": "Vault Token",
  "Type": "client",
  "Rules": "
    key \"vault/\" {
        policy = \"write\"
    }
    node \"\" {
        policy = \"write\"
    }
    service \"vault\" {
        policy = \"write\"
    }
    agent \"\"  {
        policy = \"write\"
    }
    session \"\" {
        policy = \"write\"
    }
  "
}
EOF

curl -X PUT --header "X-Consul-Token: $TOKEN" --data @vault http://127.0.0.1:8500/v1/acl/create

``` 

```
{
  "ID": "generated uuid", 
  "Name": "Agent Token",
  "Type": "client",
  "Rules": "node \"\" { policy = \"write\" } service \"\" { policy = \"read\" }"
}
```


Update anonymous acl ( needed for dns ) 

```
{
  "ID":"anonymous",
  "Name":"Anonymous Token",
  "Type":"client",
  "Rules":" node \"\" { policy = \"read\" } service \"\" { policy = \"read\" } "
}
```

# Install

Save the keys from output, 
If you want to unseal during startup, change unseal vars to your situation. 

```
vault_unseal_script: true
vault_unseal_keys: [ '"12314"', '"12313"', '"124124"' ]
```

# Configure

## LDAP
```
vault write auth/ldap/config \
        url="ldap://10.127.21.10,ldap://10.127.21.11" \
        binddn="cn=vault_ldap_user,OU=Service Accounts,dc=int,dc=units,dc=cloud" \
        bindpass="password" \
        userdn="OU=Users,OU=Internal,dc=int,dc=units,dc=cloud" \
        groupdn="OU=Groups,OU=Internal,DC=int,DC=units,DC=cloud" \
        userattr="sAMAccountName" \
        groupfilter="(&(objectClass=group)(member:1.2.840.113556.1.4.1941:={{.UserDN}}))" \
        groupattr="cn" \
        insecure_tls=true \
        starttls=false

```

## Add roles


```
vault write ssh/roles/otp_key_role \
    key_type=otp \
    default_user=ubuntu \
    cidr_list=10.127.0.0/16 \
    allowed_users=""
```


## Add Policies

```
#cat pki.hcl 
path "/intermediate_ca/issue/int_units_cloud" {
    capabilities = [ "update" ]
}
```

```
#cat ssh.hcl 
path "/ssh/creds/otp_key_role" {
    capabilities = [ "update" ]
}
```
vault policy-write ssh ssh.hcl
vault policy-write certificate pki.hcl


## Add policies to LDAP-Groups
vault write auth/ldap/Drift policies=ssh,certificate

