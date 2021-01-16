---
title: Soul 网关学习笔记02-divide插件初体验
date: 2021-01-16 19:34:34
tags:
categories:
---
使用 divide 插件，体验 http 代理
<!--more-->
## 启动项目

运行 soul-admin 模块的主函数，启动 soul-admin。

运行 soul-bootstrap 模块的主函数，启动 soul-bootstrap。

使用 admin 账号登录管理台，点击 PluginList - divide 模块，此时选择器列表（SelectorList）和规则列表（RuleList）均为空。可以手动添加选择器和规则。

运行 soul-examples 模块下的 http 子模块，默认启动端口为 8188。此时重新进入管理台 divide 模块可以看见已经注册了一个 `/http` 选择器和多个规则，分别对应 HttpTestController 和 OrderController 中的多个映射地址。规则名称是根据注解 `@SoulSpringMvcClient` 注册并显示，该名称是**上下文地址**加上标注在**类上的注解**再加上**方法上的注解**组成。配置文件中上下文地址为 /http。举例：

```java
@RestController
@RequestMapping("/order")
@SoulSpringMvcClient(path = "/order")
public class OrderController {
    @PostMapping("/save")
    @SoulSpringMvcClient(path = "/save" , desc = "Save order")
    public OrderDTO save(@RequestBody final OrderDTO orderDTO) {
        orderDTO.setName("hello world save order");
        return orderDTO;
    }
    ......
    @GetMapping("/path/{id}/name")
    @SoulSpringMvcClient(path = "/path/**/name")
    public OrderDTO testRestFul(@PathVariable("id") final String id) {
        OrderDTO orderDTO = new OrderDTO();
        orderDTO.setId(id);
        orderDTO.setName("hello world restful inline " + id);
        return orderDTO;
    }
}
```

分别对应规则名 `/http/order/save` 和 ``/http/order/path/**/name`。

回到选择器本身，可以进行修改和删除。打开修改可以看到配置项写明了 IP、port 和权重。现在笔者通过 postman 发送请求到 soul 网关，测试网关转发功能。明显的，soul-bootstrap 日志中可以看到成功收到请求，并转发给了配置中的 IP。

```markdown
The request urlPath is http://192.168.163.1:8188/order/findById?id=3, retryTimes is 0
```

## 多端口

现在修改 http 子模块的配置文件，将端口号改为 8189 并启动项目。此时观察 divide 模块的选择器，打开修改可以看到配置项出现了多行，包括 8188 和 8189 两个端口。再次使用 postman 进行测试。

```markdown
divide selector success match , selector name :/http
divide selector success match , selector name :/http/order/findById
The request urlPath is http://192.168.163.1:8188/order/findById?id=3, retryTimes is 0
divide selector success match , selector name :/http
divide selector success match , selector name :/http/order/findById
The request urlPath is http://192.168.163.1:8189/order/findById?id=3, retryTimes is 0
Unknown channel option 'SO_TIMEOUT' for channel '[id: 0x7d243849]'
divide selector success match , selector name :/http
divide selector success match , selector name :/http/order/findById
The request urlPath is http://192.168.163.1:8188/order/findById?id=3, retryTimes is 0
divide selector success match , selector name :/http
divide selector success match , selector name :/http/order/findById
The request urlPath is http://192.168.163.1:8189/order/findById?id=3, retryTimes is 0
```

在 4 次测试中，8188 和 8189 的概率相同，符合权重描述的 50 对 50。修改 8189 的权重为 100，8188 的权重为 1，再次测试。

```markdown
divide selector success match , selector name :/http
divide selector success match , selector name :/http/order/findById
The request urlPath is http://192.168.163.1:8189/order/findById?id=3, retryTimes is 0
divide selector success match , selector name :/http
divide selector success match , selector name :/http/order/findById
The request urlPath is http://192.168.163.1:8189/order/findById?id=3, retryTimes is 0
```

两次测试中均转发到了 8189 端口，符合权重描述的 1 对 100。

## 修改规则

- 打开名为 `/http/test/**` 的规则并修改条件。将 `match` 改为 `=`，将 `/http/test/**` 改为 `/http/test/findByUserId`。此时通过 postman 测试。

```markdown
divide selector success match , selector name :/http
divide selector success match , selector name :/http/test/**
The request urlPath is http://192.168.163.1:8189/test/findByUserId?userId=007, retryTimes is 0
```

访问成功。

- 修改 postman 访问地址重新测试。

```markdown
divide selector success match , selector name :/http
can not match rule data: divide
```

访问失败。

- 将规则重置后重新访问 `http://127.0.0.1:9195/http/test/path/007?name=zhangsan`。

```markdown
divide selector success match , selector name :/http
divide selector success match , selector name :/http/test/**
The request urlPath is http://192.168.163.1:8189/test/path/007?name=zhangsan, retryTimes is 0
```

访问成功。
