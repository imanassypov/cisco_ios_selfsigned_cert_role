
# This is a simple role definition and a sample playbook leveraging the role to generate self-signed certificates, distribute them to IOS-XE devices, and importing to a specified crypto PKI trustpoint.
This playbook can be used to simplify a workaround to a Cisco published field notice, https://www.cisco.com/c/en/us/support/docs/field-notices/721/fn72105.html

During the execution the playbook leverages openssl to create self-signed certs for each of the devices, and bundling the resultant .crt file per each device into a pkcs12 formatted .pfx file. PFX bundle gets transferred to each target IOS-XE device bootflash, a new trustpoint gets created and the .pfx bundled certificate gets imported into the newly defined trustpoint. 

As a validation step, status of the imported certificate is validated to ensure self-signed certificate import was successful

## Environment Variables

To run this project, you will need to add the following environment variables to your .env file

`CISCO_USERNAME` is username for SSH login to the scoped IOS-XE devices

`CISCO_PASSWORD` is the password to authenticate SSH session

`CISCO_ENABLE` is the enable password for privileged commands in IOS-XE prompt

## Caveats
OpenSSL by default leverages stronger SHA256 MAC signature when generating both the certificate, as well as the PKCS12 bundle.
In certain IOS-XE versions, this may result in the following error message when attempting to import the SHA256 signed certificate bundle

```
*Mar  4 19:43:48.332: CRYPTO_PKI: status = 0x760(E_DIGEST_ALG_NOT_SUPPORTED : message digest algorithms not supported): Imported PKCS12 file failure 
*Mar  4 19:43:48.333: %PKI-3-PKCS12_IMPORT_FAILURE: PKCS #12 import failed for trustpoint: TEST. Reason: Failed to import pkcs12 context
```

To workaround this issue, I have explicitely configured SHA1 to be used for the purposes of instantiating the certs (ref https://bst.cisco.com/bugsearch/bug/CSCvz41428)


## Sample playbook execution

ansible-playbook -i hosts.ini self-signed-certs.yml   


## Sample playbook run output

```
ansible-playbook -i hosts.ini self-signed-certs.yml                                    

PLAY [Generate self-signed certs for each host] ******************************************************************************************************************************************************

TASK [self_signed_cert : Cert path creation] *********************************************************************************************************************************************************
[WARNING]: ansible-pylibssh not installed, falling back to paramiko
[WARNING]: ansible-pylibssh not installed, falling back to paramiko
[WARNING]: ansible-pylibssh not installed, falling back to paramiko
[WARNING]: ansible-pylibssh not installed, falling back to paramiko
ok: [c9200.example.com]
ok: [c8kv-01.example.com]
ok: [c8kv-02.example.com]
ok: [c8kv-03.example.com]

TASK [self_signed_cert : Generate the private key for self-signed cert] ******************************************************************************************************************************
changed: [c8kv-03.example.com]
changed: [c9200.example.com]
changed: [c8kv-01.example.com]
changed: [c8kv-02.example.com]

TASK [self_signed_cert : Generate a Self-Signed OpenSSL Certificate] *********************************************************************************************************************************
changed: [c8kv-02.example.com]
changed: [c9200.example.com]
changed: [c8kv-01.example.com]
changed: [c8kv-03.example.com]

TASK [self_signed_cert : Convert the certificate and private key to PFX format with SHA-1] ***********************************************************************************************************
changed: [c8kv-01.example.com]
changed: [c9200.example.com]
changed: [c8kv-03.example.com]
changed: [c8kv-02.example.com]

TASK [self_signed_cert : Copy pfx cert to device] ****************************************************************************************************************************************************
ok: [c8kv-02.example.com]
ok: [c8kv-01.example.com]
ok: [c8kv-03.example.com]
ok: [c9200.example.com]

TASK [self_signed_cert : Create trustpoint "TEST"] ***************************************************************************************************************************************************
ok: [c8kv-02.example.com]
ok: [c8kv-01.example.com]
ok: [c8kv-03.example.com]
ok: [c9200.example.com]

TASK [self_signed_cert : Import PKCS12 pfx into trustpoint] ******************************************************************************************************************************************
ok: [c8kv-03.example.com]
ok: [c8kv-02.example.com]
ok: [c9200.example.com]
ok: [c8kv-01.example.com]

TASK [self_signed_cert : Trustpoint status collection] ***********************************************************************************************************************************************
ok: [c9200.example.com]
ok: [c8kv-02.example.com]
ok: [c8kv-01.example.com]
ok: [c8kv-03.example.com]

TASK [self_signed_cert : Asserting trustpoint status] ************************************************************************************************************************************************
ok: [c8kv-03.example.com] => {
    "changed": false,
    "msg": "Passed: Issuing CA certificate configured"
}
ok: [c8kv-01.example.com] => {
    "changed": false,
    "msg": "Passed: Issuing CA certificate configured"
}
ok: [c8kv-02.example.com] => {
    "changed": false,
    "msg": "Passed: Issuing CA certificate configured"
}
ok: [c9200.example.com] => {
    "changed": false,
    "msg": "Passed: Issuing CA certificate configured"
}

PLAY RECAP *******************************************************************************************************************************************************************************************
c8kv-01.example.com        : ok=9    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
c8kv-02.example.com        : ok=9    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
c8kv-03.example.com        : ok=9    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
c9200.example.com          : ok=9    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## Sample trustpoint output resulting from importing self-signed pfx certificate bundle
```
C9200L-1#show crypto pki trustpoints TEST status
Trustpoint TEST:
  Issuing CA certificate configured:
    Subject Name:
     cn=c9200.example.com
    Fingerprint MD5: 5ECAF77E 82DECF83 DFEE5CC5 FE6A0E58 
    Fingerprint SHA1: E09655FB 63A8F216 F50C1F42 9263665A 563075BD 
  Router General Purpose certificate configured:
    Subject Name:
     cn=c9200.example.com
    Fingerprint MD5: 5ECAF77E 82DECF83 DFEE5CC5 FE6A0E58 
    Fingerprint SHA1: E09655FB 63A8F216 F50C1F42 9263665A 563075BD 
  State:
    Keys generated ............. Yes (General Purpose, non-exportable)
    Issuing CA authenticated ....... Yes
    Certificate request(s) ..... Yes

```