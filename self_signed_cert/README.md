Role Name
=========

This is a simple role for Cisco IOS XE based devices to instantiate self-signed certificates, distribute certificates to each device, and import the certificates in a predefined crypto PKI trustpoint. 
This automation is developed to help customers workaround Cisco's published field notice, FN - 72105 Ref: https://www.cisco.com/c/en/us/support/docs/field-notices/721/fn72105.html

Requirements
------------

openssl should be installed

Role Variables
--------------

Settable variables which are otherwise defined as defaults (please adjust to your environment)

Location of the folder containing generated self-signed certificates per each device
cert_path: "./certs/"

PEM formatted private key default password (please adjust to your environment)
private_key_password: Cisco123

PKCS12 packaged certificate default password (please adjust to your environment)
pkcs12_password: Cisco123

MAC signing algo, specifically to workaround openssl default SHA256 defect while importing in Cisco IOS trustpoint
pkcs12_mac: sha1 #default is sha256, https://bst.cisco.com/bugsearch/bug/CSCvz41428

Name of Cisco PKI trustpoint to import self-signed cert (please adjust to your environment)
trustpoint_name: TEST


Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

Sample playbook demostrating the use of the above role

- name: Generate self-signed certs for each host
  hosts: 
    - switches
    - routers
  gather_facts: no
  roles:
    - self_signed_cert


License
-------

BSD

Author Information
------------------

Igor Manassypov, Cisco Systems Canada
Solution Engineer
imanassy@cisco.com
