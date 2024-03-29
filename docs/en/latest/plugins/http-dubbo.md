---
title: http-dubbo
keywords:
  - Apache APISIX
  - API 网关
  - Plugin
  - Http Dubbo
  - http-dubbo
description: This article provides an overview and usage instructions for the Apache APISIX http-dubbo plugin.
---

<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

## Description

The `http-dubbo` plugin enables HTTP to Dubbo protocol conversion in [**Dubbo**](https://cn.dubbo.apache.org/en/) and uses JSON as the default serialization method.

:::info IMPORTANT

If you are using OpenResty, you need to build it with Dubbo support. See [How do I build the APISIX runtime environment](./../FAQ.md#how-do-i-build-the-apisix-runtime-environment) for details.

:::

## Runtime Attributes

| Name                     | Type    | Required | Default            | Description                                |
|--------------------------|---------| ---------|--------------------|--------------------------------------------|
| service_name             | string  | True  |                    | Dubbo provider service name                |
| service_version          | string  | True    |                    | Dubbo provider service version             |
| method                   | string  | False    | The path of the URI | Dubbo provider service method              |
| params_type_desc         | string  | False    |                    | Dubbo service params                       |
| serialized               | boolean | False    | false              | true/false	                                |
| serialization_header_key | string  | False    |                    |                                            |
| connect_timeout          | number  | False    | 6000               | Connection timeout duration (milliseconds) |
| send_timeout             | number  | False    | 6000               | Send timeout duration (milliseconds)       |
| read_timeout             | number  | False    | 6000               | Read timeout duration (milliseconds)       |

## Enable Plugin

To enable the `http-dubbo` Plugin, you have to add it in your configuration file (`conf/config.yaml`):

```yaml title="conf/config.yaml"
plugins:
  - ...
  - http-dubbo
```

Now, when APISIX is reloaded, you can add it to a specific Route as shown below:

```shell
curl http://127.0.0.1:9180/apisix/admin/upstreams/1  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "nodes": {
        "127.0.0.1:20880": 1
    },
    "type": "roundrobin"
}'

curl http://127.0.0.1:9180/apisix/admin/routes/1  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uris": [
        "/hello"
    ],
    "plugins": {
        "http-dubbo": {
            "service_name": "org.apache.dubbo.sample.tengine.DemoService",
            "service_version": "0.0.0",
            "method": "tengineDubbo"
        }
    },
    "upstream_id": 1
}'
```

## Example usage

You can follow the Quick Start guide in Tengine with the configuration above for testing.

Dubbo returns data in the form Map<String, String>.

If the returned data is:

```json
{
  "status": "200",
  "header1": "value1",
  "header2": "value2",
  "body": "body of the message"
}
```

The converted HTTP response will be:

```
HTTP/1.1 200 OK
...
header1: value1
header2: value2
...

body of the message
```

## Delete Plugin

To remove the `http-dubbo` Plugin, you can delete the corresponding JSON configuration from the Plugin configuration. APISIX will automatically reload and you do not have to restart for this to take effect.

```shell
curl http://127.0.0.1:9180/apisix/admin/routes/1  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "methods": ["GET"],
    "uris": [
        "/hello"
    ],
    "plugins": {
    },
    "upstream_id": 1
    }
}'
```

To completely disable the `http-dubbo` Plugin, you can remove it from your configuration file (`conf/config.yaml`):

```yaml title="conf/config.yaml"
plugins:
  # - http-dubbo
```