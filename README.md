Inspired by https://github.com/AvalancheDev/Hashicorp-Vault-Ansible

# Prereq
Required a working consul, with ACL support and a existing token

Create the vault-token in consul
```
cat << EOF > vault
{
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

# Install

Save the keys from output, 
If you want to unseal during startup, change unseal vars to your situation. 

```
vault_unseal_script: true
vault_unseal_keys: [ '"12314"', '"12313"', '"124124"' ]
```
