# GitHub.io 邀请中转页部署与使用指南（个人主页方案）

本指南适用于把中转页部署在 GitHub Pages 的**个人/组织主页**（仓库名 `<你的用户名>.github.io`），通过**短码参数**生成邀请链接，并在 QQ/微信内仅提示“浏览器打开”，在系统浏览器内再拼接跳转。

## 1. 目标与边界

- 目标：改善 QQ/微信内置浏览器的分享体验，避免直接暴露完整邀请链接。
- 边界：若最终落地域名已被标红/拦截，本方案**无法解封**，仅改变展示与跳转时机。

## 2. 术语说明

- **中转页**：`github.io` 上的静态页面，负责识别环境、显示提示与拼接跳转。
- **短码参数**：动态变化的邀请参数（例：`2g.T2.1.1`）。
- **落地链接**：真实邀请入口（例：`https://100670.xyz/index.php/invite/s?c=` + 短码）。

## 3. 必要信息

1) 你的 GitHub 用户名  
2) 邀请链接固定前缀  
   - 示例：`https://100670.xyz/index.php/invite/s?c=`  
3) 短码参数规则  
   - 示例：`2g.T2.1.1`

## 4. 仓库与目录结构（个人主页）

- 新建仓库：`<你的用户名>.github.io`
- 目录结构建议：

```
<你的用户名>.github.io/
├─ index.html              (可选：站点主页)
└─ bridge/
   └─ index.html           (中转页)
```

## 5. GitHub Pages 启用步骤

1) 进入仓库 Settings → Pages  
2) Source 选择 `Deploy from branch`  
3) Branch 选择 `main`，路径选择 `/ (root)`  
4) 保存后等待几分钟，即可访问：

```
https://<你的用户名>.github.io/bridge/?k=短码
```

## 6. 分享链接规则

用户分享的链接（仅带短码）：

```
https://<你的用户名>.github.io/bridge/?k=2g.T2.1.1
```

浏览器内最终拼接并跳转的链接：

```
https://100670.xyz/index.php/invite/s?c=2g.T2.1.1
```

## 7. 中转页逻辑（实现思路）

### 7.1 关键行为

- 读取 `k` 参数（短码）
- 识别 UA 是否在 QQ/微信内置浏览器
- QQ/微信内：只提示“请用浏览器打开”，不跳转
- 浏览器内：拼接目标链接并跳转
- 缺少短码：提示错误，不跳转

### 7.2 伪代码

```js
const shortCode = getQueryParam('k');
const isWeixin = /micromessenger/i.test(navigator.userAgent);
const isQQ = /qq/i.test(navigator.userAgent);

if (!shortCode) {
  showError('缺少邀请码');
  return;
}

if (isWeixin || isQQ) {
  showTips('请使用浏览器打开本页面');
  // 不拼接、不跳转
} else {
  const target = 'https://100670.xyz/index.php/invite/s?c=' + shortCode;
  // 可选：显示按钮让用户确认后跳转
  location.href = target;
}
```

## 8. 建议的交互文案

- QQ/微信内提示：
  - “请点击右上角，用浏览器打开本页面。”
- 缺少参数：
  - “缺少邀请码，请检查分享链接。”
- 浏览器内：
  - 可自动跳转或提供“继续访问”按钮（更稳妥）

## 9. 本地测试方式

1) 本地起静态服务（示例端口 8000）
2) 访问：

```
http://127.0.0.1:8000/bridge/?k=2g.T2.1.1
```

3) 用浏览器模拟访问，确认能正确拼接并跳转  
4) 用 QQ/微信内置浏览器访问，确认仅提示不跳转

## 10. 风险与合规提示

- 编码/隐藏链接**不能降低**已红域名的风险
- 若最终落地域名被封，中转域名可能被连带标记
- 建议优先修复落地内容与域名信誉

## 11. 维护建议

- 邀请规则变更时，仅需改中转页拼接逻辑
- 若担心参数被篡改，建议改为短期 token 并在你站内校验

---

如需成品中转页模板（含完整 HTML/JS/CSS），告知需求即可。
