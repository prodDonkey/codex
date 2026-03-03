# 放行 GitHub HTTPS CONNECT 指南

如果在拉取仓库时出现如下错误：

- `CONNECT tunnel failed, response 403`
- `fatal: unable to access 'https://github.com/...': CONNECT tunnel failed`

通常表示出口代理/网关阻止了 HTTPS CONNECT 隧道。

## 需要放行的域名

至少放行以下目标（443 端口）：

- `github.com`
- `raw.githubusercontent.com`

建议同时放行（避免后续依赖下载失败）：

- `api.github.com`
- `codeload.github.com`
- `objects.githubusercontent.com`

## 代理侧策略建议

1. 允许 `CONNECT github.com:443`。
2. 允许 `CONNECT raw.githubusercontent.com:443`。
3. 如启用 TLS 解密，请确保不会阻断 Git 客户端握手。
4. 若按分类策略拦截，需将 GitHub 相关域名加入白名单策略组。

## 快速自检命令

```bash
curl -I https://github.com
curl -I https://raw.githubusercontent.com
git ls-remote https://github.com/web-infra-dev/midscene.git
```

三条命令都成功时，通常即可正常 `git clone`。

## 常见平台示例

### Squid

在 `squid.conf` 中加入允许目标并重载：

```conf
acl github_ssl dstdomain .github.com .githubusercontent.com
http_access allow CONNECT github_ssl
```

### 企业防火墙 / SASE 网关

- 新建“目标域名白名单”规则。
- 协议选择 HTTPS（TCP/443）。
- 动作选择 Allow。
- 生效范围覆盖当前执行环境对应的出口策略。

## 变更后验证

1. 在同一执行环境重试 `curl -I` 与 `git ls-remote`。
2. 确认不再出现 `CONNECT tunnel failed`。
3. 再执行：

```bash
git clone --depth 1 https://github.com/web-infra-dev/midscene.git
```
