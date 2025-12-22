---
title: Certificate Authentication
date: 2025-12-04 17:03:00 +0100
categories: [intune, conditional_access]
tags: [intune, mfa, conditional_access, "365", azure, microsoft]     # TAG names should always be lowercase
author: mith
description: How to enable phishing resistant mfa with certificates, and example of why it works
toc: true
---

## Context
Seeing that more and more sophisticated phinshing are taking place. The goal if this article is to showcase how to mitigate these attacks. With certificate based MFA it becomes more difficult for the threat actor to steal tokens. This is also a spin off, off the other article about Windows Hello For Business. The difference is that certifiacte based authentication does not only work on windows devices, but for the most part this works on all devices. Currently I have only tested it on Windows and Ubuntu. 

## Setup 

1. Install according to guide: [smallstep](https://smallstep.com/docs/step-cli/installation/#debianubuntu)

2. Debian/ubuntu: 

    ```
    apt-get update && apt-get install -y --no-install-recommends curl vim gpg ca-certificates
    curl -fsSL https://packages.smallstep.com/keys/apt/repo-signing-key.gpg -o /etc/apt/trusted.gpg.d/smallstep.asc && \
        echo 'deb [signed-by=/etc/apt/trusted.gpg.d/smallstep.asc] https://packages.smallstep.com/stable/debian debs main' \
        | tee /etc/apt/sources.list.d/smallstep.list
    apt-get update && apt-get -y install step-cli
```

3. Edit the configuration file found in <step path>/config/defaults.json. For me that is /etc/step/config/defaults.json. The port can be changed in <step path>/config/ca.json


    ```
    {
    "ca-url": "https://ca.internal:<chosen port>",
    "fingerprint": "<fingerprint>",
    "root": "<root ca path>",
    "redirect-url": ""
    }
    ```


4. Basic Crypto Operations Create root and issuer cert

### 365 setup 
1. log into entra.microsoft.com
2. Create an application
3. set the redirect to http://127.0.0.1
4. copy client id

5. edit ca.json: 
```
"authority": {
      "disableIssuedAtCheck": true,
      "provisioners": [
        {
        "type": "OIDC",
        "name": "Microsoft Entra",
        "clientID": "<client id>",
        "configurationEndpoint": "https://login.microsoftonline.com/<tenant_id>/v2.0/.well-known/openid-configuration"

        }
      ],
      "claims": {
      "minTLSCertDuration": "5m",
      "maxTLSCertDuration": "720h",
      "defaultTLSCertDuration": "168h"
      }
      },
```

6. upload certificates: 
7. navigate to entra.microsoft.com home > Certificate authorities > security | Puplic key infrastructure
8. create new PKI and upload intermediate cert (possibly also root?)

9. Authentication methods
10. Configure to your liking
11. Strength can be single or multifactor do some tinkering with Affinity binding. I had some issues with this. 

### Start smallstep 
With the command: 

``` 
step-ca /etc/step/config.json
```

Step will now run. This is verified since the command does not run in background. You should see somthing like serving something on endpoint - cant remember the output

This can be created as a service. But be sure to know the password for the issuer cert. This will be needed to be stored in a file for this is run as a service . 

## Test

```
step ca certificate "<endpoint>" test.crt test.key --provisioner "Microsoft Entra"
```
You should be redirected to microsoft logon page and thereafter a smallstep page, you should see a .crt and .key file in the directory you started to command

This cert needs to be converted to a p12 cert to use in browsers

```
step certificate p12 <name>.p12 <name>.key <name>.crt --no-password --insecure
```
[smallstep p12 command](https://smallstep.com/docs/step-cli/reference/certificate/p12/)

Now the cert can be importet to your favorite browser and used to authenticate with. 

### Autorenewal for Linux
```
#!/bin/bash
# renew.sh - Renew Smallstep certificates past half lifetime, convert to .p12, and log actions

CERT_DIR="$HOME/.certs"
LOG_FILE="$CERT_DIR/renew.log"

# Create log file if it doesn't exist
touch "$LOG_FILE"

# Function to log messages with timestamp
log() {
    local msg="$1"
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $msg" | tee -a "$LOG_FILE"
}

if [ ! -d "$CERT_DIR" ]; then
    log "❌ Certificate directory $CERT_DIR does not exist!"
    exit 1
fi

for cert in "$CERT_DIR"/*.crt; do
    [ -e "$cert" ] || { log "⚠️ No certificates found in $CERT_DIR"; exit 0; }

    log "Checking certificate: $cert"

    base_name=$(basename "$cert" .crt)
    key="$CERT_DIR/$base_name.key"
    p12="$CERT_DIR/$base_name.p12"

    if [ ! -f "$key" ]; then
        log "⚠️ Key file not found for $cert, skipping."
        continue
    fi

    # Get Not Before and Not After dates in seconds
    not_before=$(openssl x509 -in "$cert" -noout -startdate | cut -d= -f2)
    not_after=$(openssl x509 -in "$cert" -noout -enddate | cut -d= -f2)

    start_sec=$(date -d "$not_before" +%s)
    end_sec=$(date -d "$not_after" +%s)
    now_sec=$(date +%s)

    half_life=$(( (end_sec - start_sec) / 2 ))
    half_date=$(( start_sec + half_life ))

    if [ "$now_sec" -ge "$half_date" ]; then
        log "⏳ Certificate has passed half of its validity. Renewing..."
        step ca renew "$cert"

        step certificate p12 --no-password --insecure "$p12" "$cert" "$key"

        if [ $? -eq 0 ]; then
            log "✅ Successfully renewed $cert and converted to $p12"
        else
            log "❌ Failed to convert $cert to PKCS#12"
        fi
    else
        log "✅ Certificate is not past half of its validity. Skipping."
    fi

done

log "🔹 Renewal script completed."
```