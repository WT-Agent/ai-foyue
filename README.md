<div align="center">

# [网腾无限AI - 每日佛曰]

**[一个支持功德增长电子木鱼与五种特色开示流派的 AI 数字禅解语工具，具备深色玻璃拟态自适应交互与微信端 H5 体验]**

[Vue 3] · [TypeScript] · [Vite] · [Vanilla CSS] · [开源协议 MIT]

[![GitHub stars](https://img.shields.io/github/stars/WT-Agent/ai-foyue?style=social)](https://github.com/WT-Agent/ai-foyue)
[![GitHub license](https://img.shields.io/github/license/WT-Agent/ai-foyue)](https://github.com/WT-Agent/ai-foyue/blob/main/LICENSE)

[在线演示](#在线演示) · [快速启动](#快速启动) · [参与贡献](#参与贡献) · [支持一下](#支持一下)

</div>

## 关于我们

团队成员均来自 C9 等顶尖学府，在字节、腾讯、阿里的工程师组成，全职创业研发开源 AI 应用产品，让所有人感受 AI 的魅力。

本项目融合了传统禅宗思想与现代数字解压文化。用户可在应用中倾诉红尘俗世的烦恼内耗，通过点击高保真电子木鱼（基于 Web Audio API 动态合成敲击声并累加功德值）来进行心理调解。AI 将分析并测算用户的精神状态指数（以数据图表直观展示），并按照所选的开示流派（包含传统禅宗开示、九巨擘禅意论、赛博木鱼解压、当头棒喝警醒、温情心灵抚慰）生成隽永、风趣的解惑谶语。

**我们不搞概念，不卖课，只写能跑起来的代码。**

欢迎 Star、Fork、提 Issue，一起让这个项目变得更好用。

核心特性：
- **极简自适应交互**：提供毛玻璃质感的深色玻璃拟态自适应 Web 界面，高度适配移动端 H5 微信浏览器与 PC 体验。
- **互动电子木鱼 (Cyber Wooden Fish)**：基于前端 Web Audio API 动态合成敲击音效，点击木鱼即可累积功德数并伴随功德渐隐上升动画。
- **五大特色开示视角**：
  - **传统禅宗开示**：以经典佛教公案、古风茶香笔触娓娓道来，采用“佛曰...”的形式切入指点迷津。
  - **九巨擘禅意论**：模拟马斯克、乔布斯、秦始皇等 9 大巨擘的独特视角阐述禅修，开展激烈跨时空激辩。
  - **赛博木鱼解压**：结合现代网梗，用系统清理本地业力缓存、增加功德缓冲区等科幻解压机能消除业障。
  - **当头棒喝警醒**：极其辛辣，毫不留情，一针见血拆穿执迷错觉，如同重锤让人瞬间警醒。
  - **温情心灵抚慰**：文字温暖细腻，如冬日微光，提供可行的心理调试建议。
- **AI 精神图表看板**：自动提取 AI 回复中的共识数据，以简洁的单轨进度条在前端直观展示测算后的功德、禅定、执念、因果和浮躁指数。
- **演示案例与分享卡片**：内置 30 条佛曰解惑精彩演示样例，并支持一键卡片化截图分享。
- **一键零成本部署**：纯前端静态网页结构，支持零成本部署于 Vercel、GitHub Pages 或 CDN/OSS 静态托管服务。
- **安全开发代理**：本地开发支持使用个人 API 密钥发起代理请求，密钥由 Vite 服务器中转，无需担心前端泄露。
- **裂变解锁与留存**：内置微信朋友圈扫码分享拦截与额度重置机制，提升流量转化与留存。

## 快速启动

### 1. 克隆项目
```bash
git clone https://github.com/WT-Agent/ai-foyue.git
cd ai-foyue
```

### 2. 安装依赖
项目强制使用 pnpm 作为包管理器：
```bash
pnpm install
```

### 3. 配置本地开发环境变量
复制并修改环境变量配置文件：
```bash
cp .env.example .env
```
根据微应用的功能类型，在 `.env` 中配置您的开发者密钥：
- `DEEPSEEK_API_KEY`: 您的 DeepSeek 开发者 API 密钥（用于文本生成任务）
- `DASHSCOPE_API_KEY`: 您的通义千问/通义万相开发者 API 密钥（用于多模态与生图任务）

### 4. 启动本地开发服务
```bash
pnpm dev
```
启动成功后在浏览器访问控制台输出的地址即可。

### 5. 生产构建打包
```bash
pnpm build
```
打包后生成的 `dist` 目录即为纯静态网页资源，可直接上传部署。

## 脚手架集成说明

本模板由私有总控仓库 `ai.wuxian.xyz` 中的 `@wuxian/cli` 脚手架统一管理，支持以下批量运维操作：

### 初始化或更新单个子项目

```bash
node bin/cli.js ai-foyue
```

脚手架将自动：
1. 读取子仓库的 `README.md` 首行作为 Prompt 主题。
2. 注入 Vue 3 静态页面结构及标准配置文件。
3. 保留原有的 `.git` 配置与 `README.md`，不覆盖个性化内容。

### 批量同步所有子项目

```bash
node bin/cli.js all
```

将模板的最新变更（如 SSO 逻辑、额度控制）一键同步至全部 31 个子项目。

### Agent 配置维护接口

```bash
# 读取子项目配置
node bin/cli.js get ai-foyue

# 写入/更新配置（支持热更新 prompt、model、title、temperature 等）
node bin/cli.js set ai-foyue prompt "你是一个深谙佛法、智慧空灵的禅宗解惑大师..."
node bin/cli.js set ai-foyue model deepseek-chat
```

## 联系方式

- GitHub Issues: [提交反馈](https://github.com/WT-Agent/ai-foyue/issues)
- 邮箱: us@wuxian.xyz

## 打赏支持

如果本项目对您有帮助，欢迎请作者喝杯咖啡。您的支持是持续维护与更新的动力。

<div align="center">

**微信支付** | **支付宝**
:---:|:---:
<img src="./asset/tenpay.png" width="200" alt="微信支付"> | <img src="./asset/alipay.png" width="200" alt="支付宝">

</div>

## 版权与许可

本项目基于 MIT License 开源协议。

Copyright (c) 2026. All rights reserved.
