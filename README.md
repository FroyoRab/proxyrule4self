# proxyrule4self

个人自定义分流规则库，供 mihomo / Clash.Meta 系客户端以 rule-providers 方式引用。
已内置在节点管理站下发的订阅配置中，**优先级高于所有内置规则**（dler-io/Rules 等）。

## 文件说明

| 文件 | 作用 | 订阅中的规则 |
|---|---|---|
| `rules/Reject.yaml` | 强制拦截（广告、追踪等） | `RULE-SET,Self_Reject,REJECT` |
| `rules/Direct.yaml` | 强制直连 | `RULE-SET,Self_Direct,DIRECT` |
| `rules/Proxy.yaml` | 强制走代理 | `RULE-SET,Self_Proxy,🚀 节点选择` |

命中顺序：`Self_Reject → Self_Direct → Self_Proxy → 内置规则 → 兜底`。同一域名不要写进多个文件。

## 如何添加规则

编辑对应文件的 `payload` 列表，每行一条规则，**只写匹配条件，不写目标**（REJECT/DIRECT/代理由订阅侧决定）：

```yaml
payload:
  - DOMAIN,www.example.com        # 精确匹配单个域名
  - DOMAIN-SUFFIX,example.com     # example.com 及所有子域名
  - DOMAIN-KEYWORD,example        # 域名含关键字（慎用，易误伤）
  - IP-CIDR,1.2.3.4/32            # IP 段
  - GEOSITE,netflix               # geosite 分类（mihomo 系）
  - GEOIP,CN                      # IP 归属地
```

两种改法：

1. **GitHub 网页直接改**：打开文件 → 铅笔图标 → 编辑 → Commit。最简单，适合临时加一两条。
2. **本地改了推送**：

   ```bash
   git clone git@github.com:FroyoRab/proxyrule4self.git
   cd proxyrule4self
   # 编辑 rules/*.yaml
   git add -A && git commit -m "add rules" && git push
   ```

## 生效时间

- 客户端每 **1 小时**自动更新一次自定义规则（rule-provider interval=3600）。
- 等不及就在客户端里手动「更新订阅/更新规则提供者」，立刻生效。
- 空规则（`payload: []`）是合法的，不影响使用。

## 想新增一个规则文件（如 Game.yaml）

1. 在本仓库 `rules/` 下新建文件，格式照抄现有三个。
2. 改节点管理站 `manager/subscription.py` 的 `_SELF_RULESETS`，加一条
   `("Self_Game", "Game.yaml", "DIRECT")`（目标按需），然后重启 pnm 服务。
3. 只加文件不改订阅侧，文件不会生效——是否引用由订阅配置决定。
