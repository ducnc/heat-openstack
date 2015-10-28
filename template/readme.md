0. Requirement:
===============

- OS: Windows 10
- virtualenv
- Install heat client

1. Setup environmet
===================

```sh
venv\Scripts\activate
cd template
demo-openrc.bat
```

2. Deploy Stack
===============

2.1. Single VM
--------------

This template will deploy a VM with Apache and Mysql DB

`heat stack-create -f vm-web-db-1node.yaml -P key=demo_key single-vm`

2.2. Multi VM
--------------

This template will deploy 2 VM with Apache and Mysql DB on each node, using a private network for database and floating a IP for Web Server 

- Without create 2 Private Networks

`heat stack-create -f vm-web-db-2node-v1.yaml -P key=demo_key single-vm`

- Create 2 new Private Networks

`heat stack-create -f vm-web-db-2node-v2.yaml -P key=demo_key single-vm`

or:

`heat stack-create -f vm-web-db-2node-v3.yaml -P key=demo_key single-vm`
