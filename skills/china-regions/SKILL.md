---
name: china-regions
description: 从民政部 API 获取或更新全国省市区行政区划数据，生成 Ant Design Cascader 格式的 JSON 文件。仅在用户明确要求获取、抓取、同步、更新全国省市区行政区划数据时触发。不要在用户只是提到地区选择、地址联动等 UI 需求时触发。
---

# 中国省市区行政区划数据集成

从民政部行政区划 API 获取最新数据，转换为 Ant Design Cascader 格式并保存到项目中。

## 数据来源

民政部全国行政区划信息查询平台：`https://dmfw.mca.gov.cn/XzqhVersionPublish.html`

核心 API（一个接口即可获取完整三级数据）：

```
GET https://dmfw.mca.gov.cn/xzqh/getList?code=0&trimCode=true&maxLevel=3&_={timestamp}
```

**必须携带以下请求头**（否则返回 403）：

```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36
Referer: https://dmfw.mca.gov.cn/XzqhVersionPublish.html
Accept: application/json, text/plain, */*
```

**`_` 参数**是毫秒级时间戳，用 `Date.now()` 生成即可。

## 响应结构

```json
{
  "data": {
    "code": "00",
    "children": [
      {
        "code": "11",
        "name": "北京市",
        "level": 1,
        "type": "直辖市",
        "children": [
          { "code": "110101", "name": "东城区", "level": 3, "type": "市辖区", "children": [] }
        ]
      },
      {
        "code": "44",
        "name": "广东省",
        "level": 1,
        "type": "省",
        "children": [
          {
            "code": "4401",
            "name": "广州市",
            "level": 2,
            "type": "地级市",
            "children": [
              { "code": "440103", "name": "荔湾区", "level": 3, "type": "市辖区", "children": [] }
            ]
          }
        ]
      }
    ]
  }
}
```

数据特点：
- 直辖市（北京/天津/上海/重庆）的 children 直接是 level=3 的区县
- 普通省份的 children 是 level=2 的地级市，地级市下再有 level=3 的区县
- 共 34 个省级行政区（含港澳台），约 450 个地级市，约 2700+ 个区县

## 执行步骤

### 1. 获取数据

用 curl 获取原始数据，保存为临时文件：

```bash
TIMESTAMP=$(node -e "console.log(Date.now())")
curl -s \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36" \
  -H "Referer: https://dmfw.mca.gov.cn/XzqhVersionPublish.html" \
  -H "Accept: application/json, text/plain, */*" \
  "https://dmfw.mca.gov.cn/xzqh/getList?code=0&trimCode=true&maxLevel=3&_=${TIMESTAMP}" \
  -o <临时文件路径>
```

### 2. 转换格式

用 Node.js 脚本将原始数据转换为两种格式：

```javascript
const fs = require('fs');
const raw = JSON.parse(fs.readFileSync('<临时文件路径>', 'utf8'));

// 转换为 Cascader 格式: { value, label, children }
function convert(node) {
  const item = { value: node.code, label: node.name };
  if (node.children && node.children.length > 0) {
    item.children = node.children.map(convert);
  }
  return item;
}

const cascaderData = raw.data.children.map(convert);

// 生成扁平映射: code -> name（用于根据 code 反查名称）
const flatMap = {};
function flatten(nodes) {
  for (const node of nodes) {
    flatMap[node.code] = node.name;
    if (node.children && node.children.length > 0) flatten(node.children);
  }
}
flatten(raw.data.children);

// 输出为 JSON（不格式化，减小体积）
fs.writeFileSync('<输出路径>', JSON.stringify({ cascader: cascaderData, flatMap }));
```

### 3. 创建文件

在项目中创建两个文件：

**`src/data/regions.json`** — 上一步生成的 JSON 数据（约 200KB）

**`src/data/regions.ts`** — TypeScript 导出：

```typescript
import type { DefaultOptionType } from 'antd/es/cascader';

import data from './regions.json';

export const regionCascaderOptions =
  data.cascader as unknown as DefaultOptionType[];

export const regionNameMap = data.flatMap as Record<string, string>;
```

注意：`src/data/` 目录可能不存在，需要先创建。

### 4. 验证

- 确认 JSON 文件大小约 200KB
- 确认省份数为 34（含港澳台）
- 确认 TypeScript 编译无错误

## 使用示例

```tsx
import { Cascader } from 'antd';
import { regionCascaderOptions, regionNameMap } from '@/data/regions';

// Cascader 选择器
<Cascader options={regionCascaderOptions} placeholder="请选择省/市/区" />;

// 根据 code 反查名称
const name = regionNameMap['110105']; // → "朝阳区"
```

## 数据更新

行政区划数据每年更新。如需更新，重新执行步骤 1-2 即可覆盖 `regions.json`，TypeScript 文件无需改动。
