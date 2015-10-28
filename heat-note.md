http://docs.openstack.org/developer/heat/template_guide/hot_spec.html

*Heat* là sự phối hợp của các thành phần để triển khai OPS
Cơ bản nhất heat quản lý về hạ tầng (infrac) nhưng nó cũng hỗ trợ các phần mềm quản lý cấu hình
Heat cung cấp khả năng cho phép người dùng tự định nghĩa ứng dụng của mình thông qua các template

Template của Heat gọi là HOT(Heat Orchestration Template), HOT có thể được viết dưới dạng YAML hoặc JSON, tuy nhiên YAML là định dạng được sử dụng phổ biến hơn do tính đơn giản của nó

Các thành phần của HOT được chia thành các section như sau:

```sh
-  heat_template_version
#phiên bản của HOT

- description
#chú thích cho HOT ( optional )

- parameter_groups
#input param group ( optional )

- parameters
#Phần này cho phép xác định các thông số đầu vào mà phải được cung cấp khi sử dụng template.phần này là tùy chọn và có thể bỏ qua nếu như không yêu cầu tham số đầu vào

-  resources
#định nghĩa các resource được sử dụng trong template, cần ít nhất 1 resource trong template, nó chính là sản phầm và mục đích của template.
VD: Khi viết một template để tạo ra các Router, dải mạng, VM thì lúc này cần định nghĩa resources là Router, VM, dải mạng

- outputs
#cho phép xác định các thông số đầu ra có sẵn cho người dùng một khi mẫu được khởi tạo, đây là phần tùy chọn và có thể được bỏ qua khi không có giá trị sản lượng được yêu cầu.
```

1. heat_template_version
=======

Mỗi version sẽ hỗ trợ một số tính năng nhất định, truy cập http://docs.openstack.org/developer/heat/template_guide/hot_spec.html#heat-template-version để biết các tính năng này

2. parameter_groups
=======

- định nghĩa dưới dạng list với mỗi group chứa một list các params

- các list được sử dụng để biểu thị thứ tự mong muốn của các param

- mỗi param chỉ nên nằm trong một group nhất định

Cú pháp:
--------

```
parameter_groups:
- label: <human-readable label of parameter group>
  description: <description of the parameter group>
  parameters:
  - <param name>
  - <param name>
```
  
`label`: nhãn mà người dùng có thể đọc được liên hệ đến nhóm của các param

`description`: mô tả 

`parameters`: một list của các param trong nhóm này

`param name`: tên của các param reong parameters section

3. parameters section
=========

Sử dụng để định nghĩa các tham số thay đổi với mỗi khi triển khai template ví dụ như username hoặc password, image,...
Mỗi param được đặt trong một khối với tên của param bắt đầu và theo sau là một loạt các thuộc tính của param

Cú pháp:
--------

```sh
parameters:
  <param name>:
    type: <string | number | json | comma_delimited_list | boolean>
    label: <human-readable name of the parameter>
    description: <description of the parameter>
    default: <default value for parameter>
    hidden: <true | false>
    constraints:
      <parameter constraints>
```

`param name`: tên của param

`type`: loại của param là 1 trong số string, number, comma_delimited_list, json and boolean (cần phải có)

`label`: A human readable name for the parameter. This attribute is optional.

`description`: A human readable description for the parameter. This attribute is optional.

`default`: giá trị mặc định được sử dụng nếu người dùng không truyền giá trị vào. Đây là giá trị tùy chọn

`hidden`: Được dùng để hẩn thông tin của param khi ra show template. Giá trị này là tùy chọn và mặc định nó có giá trị là False.

`constraints`: Đây là điều kiện ràng buộc của param. Khi khai báo param thì nó sẽ check xem thông số của param có trong hệ thống không. Đây là trường tùy chọn http://docs.openstack.org/developer/heat/template_guide/hot_spec.html#parameter-constraints


4. resource section
========

Xác định nguồn lực thực tế làm nên 1 stack được deploy từ template hay chính là sản phẩm của template đó.

```sh
resources:
  <resource ID>:
    type: <resource type>
    properties:
      <property name>: <property value>
    metadata:
      <resource specific metadata>
    depends_on: <resource ID or list of ID>
    update_policy: <update policy>
    deletion_policy: <deletion policy>
```

`resource ID`: ID của Resource ( tên của Resource ) và giá trị này là duy nhất trong template

`type`: Loại Resource, ví dụ như: OS::Nova::Server or OS::Neutron::Port. Giá trị này bắt buộc phải có trong Resource và tùy theo Resouce mà cần chỉ ra loại. VD như Resouce là VM thì cần định nghĩa loại là OS::Nova::Server

`properties`: A list of resource-specific properties. The property value can be provided in place, or via a function (see Intrinsic functions). This section is optional.

`metadata`: Resource-specific metadata. This section is optional.

`depends_on`: Dependencies of the resource on one or more resources of the template. See Resource dependencies for details. This attribute is optional.

`update_policy`: Update policy for the resource, in the form of a nested dictionary. Whether update policies are supported and what the exact semantics are depends on the type of the current resource. This attribute is optional.

`deletion_policy`: Deletion policy for the resource. Which type of deletion policy is supported depends on the type of the current resource. This attribute is optional.


5. Output Section
============

Định nghĩa ra các params trả về cho người dùng sau khi stack được tạo ra ví dụ như địa chỉ IP của instance, địa chỉ URL của ứng dụng web được deploy trong stack

```sh
outputs:
  <parameter name>:
    description: <description>
    value: <parameter value>
```

`parameter name`: The output parameter name, which must be unique within the outputs section of a template.

`description`: A short description of the output parameter. This attribute is optional.

`parameter value`: The value of the output parameter. This value is usually resolved by means of a function. See Intrinsic functions for details about the functions. This attribute is required.