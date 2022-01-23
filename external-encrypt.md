# External encrypt -> Vault decrypt

## Configure Vault
```
vault secrets enable transit

vault write /transit/keys/mykey \
   exportable=true \
   type=rsa-4096
```

## Export public key
```
vault read /transit/keys/mykey -format=json |\
   jq -r .data.keys.\"1\".public_key >> public_key.pem
```

## Create plaintext
```
echo "hashi4lyf" > message.txt
```

## Confirm OpenSSL version
```
/usr/local/opt/openssl@3/bin/openssl  version
OpenSSL 3.0.0 7 sep 2021 (Library: OpenSSL 3.0.0 7 sep 2021)
```

## External encrypt
```
export ENCODED_CIPHERTEXT=$(/usr/local/opt/openssl@3/bin/openssl \
        pkeyutl \
        -in message.txt \
        -encrypt \
        -pubin \
        -inkey public_key.pem \
        -pkeyopt rsa_padding_mode:oaep \
        -pkeyopt rsa_oaep_md:sha256 \
        -pkeyopt rsa_mgf1_md:sha256 | \
        base64)
```

## Vault decrypt
```
vault write /transit/decrypt/mykey \
   ciphertext="vault:v1:$ENCODED_CIPHERTEXT" -format=json | \
   jq -r  .data.plaintext | \
   base64 -d

hashi4lyf
```

