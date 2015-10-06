Heat xây dựng sẵn một số phương thức được sử dụng trong template để thực hiện một hành động cụ thể nào đó chẳng hạn như lấy giá trị của resource,...
Dưới đây là các hàm được sử dụng trong một template, tuy nhiên tùy thuộc vào phiên bản của Heat mà các hàm này được hỗ trợ hay không
http://docs.openstack.org/developer/heat/template_guide/hot_spec.html#hot-spec-intrinsic-functions

1. get_attr
===========

Sử dụng trong output để lấy ra các giá trị (thông tin) mong muốn của máy ảo và trả lại cho người dùng

```sh
get_attr:
  - <resource name>
  - <attribute name>
  - <key/index 1> (optional)
  - <key/index 2> (optional)
  - ...
```

`resource name`: The resource name for which the attribute needs to be resolved. The resource name must exist in the resources section of the template.

`attribute name`: The attribute name to be resolved. If the attribute returns a complex data structure such as a list or a map, then subsequent keys or indexes can be specified. These additional parameters are used to navigate the data structure to return the desired value.

Ví dụ:
------

```sh
outputs:
  instance_ip:
    description: IP address of the deployed compute instance
    value: { get_attr: [my_instance, first_address] }
  instance_private_ip:
    description: Private IP address of the deployed compute instance
   value: { get_attr: [my_instance, networks, private, 0] }
```

Với đầu vào như sau, khi thực hiện hàm `get_attr` sẽ trả ra kết quả

`networks: {"public": ["2001:0db8:0000:0000:0000:ff00:0042:8329", "1.2.3.4"], "private": ["10.0.0.1"]}`

=> get_attr: 10.0.0.1


2. get_file
==========

Trả về nội dung của một file trong template. Thường được sử dụng như một phương pháp file inclusion cho các script hoặc các file cấu hình.

`get_file: <content key>`

`content key:` là một URL cố đinh chứ không thể là một biến lấy từ get_param

Ví dụ:
-----

```sh
resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      # general properties ...
      user_data:
        get_file: my_instance_user_data.sh
  my_other_instance:
    type: OS::Nova::Server
    properties:
      # general properties ...
      user_data:
        get_file: http://example.com/my_other_instance_user_data.sh
```

3. get_param
============

Sử dụng để lấy giá trị của param truyền vào template
	 
```sh
get_param:
 - <parameter name>
 - <key/index 1> (optional)
 - <key/index 2> (optional)
 - ...
```

`parameter name`: The parameter name to be resolved. If the parameters returns a complex data structure such as a list or a map, then subsequent keys or indexes can be specified. These additional parameters are used to navigate the data structure to return the desired value.

Ví dụ:
------

```sh
parameters:
   instance_type:
    type: string
    label: Instance Type
    description: Instance type to be used.
  server_data:
    type: json

resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      flavor: { get_param: instance_type}
      metadata: { get_param: [ server_data, metadata ] }
      key_name: { get_param: [ server_data, keys, 0 ] }
```

4. get_resource
==============

references another resource within the same template

`get_resource: <resource ID>`


5. list_join
===========

Join các phần tử của 1 list (string) thành 1 chuỗi cùng với delimeter

```sh
list_join:
- <delimiter>
- <list to join>
```

Ví dụ: `list_join: [', ', ['one', 'two', 'and three']]`

kết quả: one, two, and three.

6. digest
==========

dùng để hash một chuỗi thành số

```sh
digest:
  - <value>
  - <algorithm>
```

`algorithm`: The digest algorithm. Valid algorithms are the ones provided natively by hashlib (md5, sha1, sha224, sha256, sha384, and sha512) or any one provided by OpenSSL.

`value`: The value to digest. This function will resolve to the corresponding hash of the value.

Ví dụ: `pwd_hash: { digest: ['sha512', { get_param: raw_password }] }`


7. repeat
========

sử dụng như các câu lệnh lặp trong các ngôn ngữ lập trình, repeat cho phép duyệt qua 1 hoặc nhiều list và thay thế các phần tử của list trong template 

```sh
repeat:
  template:
    <template>
  for_each:
    <var>: <list>
```

`template`: định nghĩa ra nội dung được tạo qua mỗi lần lặp

`for_each`: <var> là biến sẽ được gán với từng giá trị của <list>  đầu vào và sử dụng trong template, <list> là list đầu vàoVí dụ:

Ví dụ:
-------

```sh
parameters:
  ports:
    type: comma_delimited_list
    label: ports
    default: "80,443,8080"
  protocols:
    type: comma_delimited_list
    label: protocols
    default: "tcp,udp"

resources:
  security_group:
    type: OS::Neutron::SecurityGroup
    properties:
      name: web_server_security_group
      rules:
        repeat:
          for_each:
            %port%: { get_param: ports }
            %protocol%: { get_param: protocols }
          template:
            protocol: %protocol%
            port_range_min: %port%
```

8. resource_facade
===========

retrieves data in a parent provider template

9. srt_replace
==============

thay thế một chuỗi bằng 1 chuỗi khác, có thể sử dụng trong outputs để trả về cho user hoặc sử dụng trong resource

```sh
str_replace:
  template: <template string>
  params: <parameter mappings>
```

`template`: chuỗi sẽ bị thay đổi

`params`: các chuỗi thay thế

Ví dụ:
-----

```sh
outputs:
  Login_URL:
    description: The URL to log into the deployed application
    value:
      str_replace:
        template: http://host/MyApplication
        params:
          host: { get_attr: [ my_instance, first_address ] }
          
resources:
  my_instance:
    type: OS::Nova::Server
    properties:
      # general properties ...
      user_data:
        str_replace:
          template: |
            #!/bin/bash
            echo "Hello world"
            echo "Setting MySQL root password"
            mysqladmin -u root password $db_rootpassword
            # do more things ...
          params:
            $db_rootpassword: { get_param: DBRootPassword }
```

10. str_split
==========

chia một chuỗi thành một list dựa vào delimeter

```
str_split:
  - ','
  - string,to,split
```

Ví dụ: `str_split: [',', 'string,to,split']``

=> ['string', 'to', 'split']

`str_split: [',', 'string,to,split', 0]`

=> 'string'