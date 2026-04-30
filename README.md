# buyu_pywencai

基于 [pywencai](https://github.com/zsrl/pywencai) 的个人维护版本，修复并升级了 `hexin-v` 反爬 token 生成模块，适配同花顺问财当前版本。

## 与原版的区别

| 项目 | 原版 pywencai | 本版本 buyu_pywencai |
|------|-------------|-------------------|
| TOKEN_SERVER_TIME | 硬编码 2022 年时间戳 | 动态获取当前时间 |
| Cookie/请求头名 | `v-nixeh`（旧版反写） | `hexin-v`（与当前问财一致） |
| hexin-v.bundle.js | 旧版打包 | 重新打包 |

## 声明

1. 本项目基于 pywencai 开源项目修改，并非同花顺官方提供的工具。
2. 该工具只是效率工具，为了便于通过 python 获取问财数据，用于量化研究和学习。
3. 建议低频使用，反对高频调用，高频调用会被问财屏蔽，请自行评估技术和法律风险。
4. 项目代码遵循 MIT 开源协议，但不赞成商用，商用请自行评估法律风险。

## 环境依赖

- Python 3.8+
- [Node.js](https://nodejs.org/en/) **v16+**（hexin-v token 生成需要）

## 安装

```bash
git clone https://github.com/zhaobuyu/buyu_pywencai.git
cd buyu_pywencai
pip install -e .
```

## Demo

```python
import pywencai

res = pywencai.get(query='退市股票', sort_key='退市@退市日期', sort_order='asc', cookie='你的cookie')
print(res)
```

## API

### get(**kwargs)

根据问财语句查询结果

#### 参数

##### query

必填，查询问句

##### sort_key

非必填，指定用于排序的字段，值为返回结果的列名

##### sort_order

非必填，排序规则，值为 `asc`（升序）或 `desc`（降序）

##### page

非必填，查询的页号，默认为1

##### perpage

非必填，每页数据条数，默认值100，最大值为100

##### loop

非必填，是否循环分页，返回多页合并数据。默认值为 `False`，可以设置为 `True` 或具体数值。

- 设置为 `True` 时，循环到最后一页，返回全部数据
- 设置具体数值 `n` 时，循环请求 n 页，返回 n 页合并数据

##### query_type

非必填，默认为 `stock`，取值如下：

| 取值 | 含义 |
|-|-|
| stock | 股票 |
| zhishu | 指数 |
| fund | 基金 |
| hkstock | 港股 |
| usstock | 美股 |
| threeboard | 新三板 |
| conbond | 可转债 |
| insurance | 保险 |
| futures | 期货 |
| lccp | 理财 |
| foreign_exchange | 外汇 |

##### retry

非必填，默认为10，请求失败后的重试次数。

##### sleep

非必填，默认为0，循环请求时每次间隔秒数。

##### log

非必填，默认为 `False`，是否在控制台打印日志。

##### pro

非必填，默认为 `False`，付费版传 `True`。必须传入 cookie 参数才能使用付费版。

##### cookie

必填，从浏览器请求头中复制 Cookie 字段值。cookie 有效期约 7 天（由 `sess_tk` JWT 控制）。

##### request_params

非必填，默认为 `{}`，可设置额外的 request 参数：

```python
pywencai.get(query='昨日涨幅', loop=True, log=True, cookie='xxx',
             request_params={'proxies': proxies})
```

##### no_detail

非必填，默认为 `False`。设为 `True` 时，详情类问题不再返回字典而返回 `None`，保证结果类型始终为 `pd.DataFrame` 或 `None`。

##### find

非必填，默认为 `None`，传数组如 `['600519', '000010']`，对应标的排列在最前面。配置后 loop 参数失效，结果只返回前100条。

##### user_agent

非必填，默认为 `None`，自定义 user_agent。

### 返回值

- 列表查询：返回 `pandas.DataFrame`
- 详情查询：返回 `dict`，包含文本和 `DataFrame`

## Cookie 获取方法

1. 打开浏览器访问 [问财](https://www.iwencai.com/) 并登录
2. 按 F12 打开开发者工具 → Network 标签
3. 在问财输入任意查询，找到请求
4. 复制请求头中的 Cookie 字段值

## hexin-v 更新说明

当问财更新反爬策略导致 token 失效时：

1. 用浏览器 F12 找到新版 `chameleon.x.x.min.xxxx.js` 的 URL
2. 下载并替换 `pywencai/hexin-v.js`（需要添加 jsdom 环境适配 + `function v(){return _red()}` 导出）
3. 执行 `cd pywencai && npx webpack` 重新打包
4. 测试 `node pywencai/hexin-v.bundle.js` 是否输出 token
