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
