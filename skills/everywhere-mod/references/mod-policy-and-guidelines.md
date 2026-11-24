# Everywhere Mod Ecosystem and Acceptable Use Policy
# Everywhere 模组生态与可接受使用政策

This document sets forth the licensing architecture, brand boundaries, security and content standards, platform integrity requirements, and enforcement terms governing mods developed for or distributed within the **Everywhere** (`Sylinko/Everywhere`) ecosystem. Both the English text and the Simplified Chinese translation express the same policy; in the event of any discrepancy, the English text governs interpretation.

---

# Part I — English Policy

## 1. Scope and Legal Effect

1. **Ecosystem and Distribution Terms**: This Policy operates as the governing acceptable use, repository submission, cryptographic verification, and distribution policy for official or curated Everywhere mod repositories, official indexes, and trust designations operated by **Sylinko Inc.** ("Sylinko").
2. **Relationship to Open-Source Licenses**: This Policy is maintained separately from the verbatim open-source software licenses applicable to code in `Sylinko/Everywhere` and `Sylinko/skills`. It does not modify the text of the GNU Affero General Public License v3.0 (`AGPL-3.0-only`) or the Apache License, Version 2.0 (`Apache-2.0`).
3. **Relationship to the Mod Interface Additional Permission**: Technical interface compliance under Section 6 (Platform Integrity and Security Boundaries) defines the scope boundary of the Mod Interface Additional Permission granted under Section 7 of GNU AGPLv3 (published in `EXCEPTIONS` at the root of the `Sylinko/Everywhere` repository). A mod that exceeds that technical boundary falls outside the scope of the Additional Permission.

## 2. Licensing Architecture and Brand Rights

1. **Modding SDK, Contracts, and Skills (`Apache-2.0`)**:
   - `Everywhere.Versioning`, `Everywhere.Modding.Abstractions`, `Everywhere.Modding.Packaging`, `Everywhere.Modding.Sdk`, the sample mod (`Everywhere.Mods.Sample`), and the `Sylinko/skills` knowledge base (`everywhere`, `everywhere-mod`) are licensed under the **Apache License, Version 2.0 (`Apache-2.0`)**.
2. **Everywhere Host and First-Party Mods (`AGPL-3.0-only` + Commercial Licensing)**:
   - The Everywhere host application (`Everywhere.Core` and platform host projects) and official first-party mods published by Sylinko (such as `everywhere.everything` and `everywhere.themes`) are licensed under **GNU Affero General Public License v3.0 (`AGPL-3.0-only`)**, with separate commercial licensing available from Sylinko Inc. (`contact@sylinko.com`; see `LICENSE-COMMERCIAL.md` at the root of `Sylinko/Everywhere`).
3. **Independent Third-Party Mods (Mod Interface Additional Permission)**:
   - Subject to the Mod Interface Additional Permission granted under Section 7 of GNU AGPLv3, an independent third-party mod that interacts with Everywhere solely through documented public mod loading and host service interfaces, does not modify or redistribute the Everywhere host itself, and respects Section 6 of this Policy may be distributed under **any license selected by its author** (including `Apache-2.0`, `MIT`, `GPL-3.0`, or proprietary/commercial terms).
   - Authors must declare their chosen license accurately via `<EverywhereModLicense>` and, where applicable, bundle the license text and third-party attribution notices via `<EverywhereModLicenseFile>` and `ThirdPartyFiles`.
4. **Brand Identity and Reserved Visual Artwork (`All Rights Reserved`)**:
   - The brand name **Sylinko**, the official product identity of **Everywhere** as published by Sylinko Inc., and the original logos, application icons, banners, and visual artwork enumerated in `TRADEMARK.md` at the root of `Sylinko/Everywhere` are **All Rights Reserved** by Sylinko Inc.
   - No open-source code license grants permission to reuse those reserved artwork assets or present a third-party mod or fork in a way that implies official origin, affiliation, or endorsement by Sylinko Inc.

## 3. Security, Privacy, and Transparency Requirements

1. **Prohibited Malicious Functionality**: Mods must not contain, deliver, or facilitate malware, trojans, backdoors, keyloggers, credential, token, cookie, or wallet exfiltration, ransomware, unauthorized cryptocurrency mining, spyware, denial-of-service routines, or code designed to destabilize user systems.
2. **Reviewability and Undeclared Payloads**: Mods submitted to official or curated repositories must not employ malicious obfuscation, packing, encrypted payloads, anti-analysis techniques, or silent downloading and execution of undeclared binaries or scripts.
3. **Capability and Telemetry Disclosure**:
   - Whenever a mod accesses local files outside its assigned `DataDirectory`, launches external processes, performs outbound network communication, or observes system input or screen state, its README and permission metadata must accurately disclose those behaviors and their purpose.
   - Any analytics or telemetry collection must be strictly opt-in, disabled by default, and transparently documented.

## 4. Content Compliance Requirements

Mods developed for distribution through official or community Everywhere indexes must not contain, generate, link to, or promote:
- Sexually explicit or pornographic material (NSFW), or any form of child sexual exploitation and abuse (CSAM);
- Hate speech, harassment, doxxing, non-consensual disclosure of personal data, credible threats, or incitement to extreme violence;
- Phishing, fraud, impersonation, or tooling intended specifically to facilitate unlawful acts or unauthorized system access.

## 5. Intellectual Property and Reserved Identifiers

1. **Third-Party Rights**: Authors must own or hold valid distribution rights for all source code, compiled binaries, icons, fonts, themes, and third-party libraries included in a mod package, and must retain all required license and copyright notices.
2. **Reserved Namespaces**: The mod identifier prefixes **`everywhere.*`** and **`sylinko.*`** are strictly reserved for official mods published by Sylinko Inc. Third-party mods must use an author-owned or reverse-domain identifier (for example, `alice.translator` or `io.github.alice.pomodoro`) and must not impersonate official Sylinko packages.

## 6. Platform Integrity and Security Boundaries

Because mods execute within the host application's managed process, strict separation from core host services and security mechanisms is required. Mods **must not** modify, conceal, spoof, intercept, replace, disable, circumvent, or degrade any of the following Everywhere subsystems—whether through non-public reflection, runtime IL patching (such as Harmony, MonoMod, or detours), dependency-injection container replacement, host configuration tampering, traffic interception, or UI occlusion:

1. **Account and Authentication Subsystem**: Login workflows, OAuth credentials and tokens, session state, device identifiers, and account or license entitlement verification.
2. **Cloud Sync Subsystem**: Synchronization transport, service endpoints, quota enforcement, and user data synchronization integrity.
3. **Official Model Gateway**: Official preset model access, rate limiting, billing and quota accounting, gateway routing, and provider safety headers.
4. **Mod Verification and Repository Trust Controls**: Mod cryptographic signature verification and official repository verification badges or trust indicators.
5. **Official Links, Attribution, and Sponsorship Notices**: Built-in URLs, references, or UI elements pointing to official Everywhere repositories, websites, documentation, license or copyright notices, or official sponsorship and donation channels.

Any mod that breaches the technical boundaries defined in this Section exceeds the scope of the Mod Interface Additional Permission under Section 7 of GNU AGPLv3.

## 7. Enforcement, Compatibility Disclaimer, and Policy Updates

1. **Enforcement Actions**: Sylinko reserves the right to reject, delist, revoke cryptographic trust or verification signatures for, or blocklist any mod package, identifier, or repository source that violates this Policy, without prior notice where user security or legal compliance is at risk.
2. **No Stability Guarantee for Non-Public Internals**: Internal or non-public host types and members do not constitute a supported mod API and may change or be removed in any release without notice.
3. **Independent Responsibility and Disclaimer**: Third-party mods are authored and distributed under the sole responsibility of their respective publishers. Sylinko makes no warranty regarding third-party mods and assumes no liability for damages arising from their installation or use.
4. **Policy Revisions**: Sylinko may revise this Policy to reflect new platform capabilities, security requirements, or legal obligations. Continued submission to or distribution through official Everywhere channels constitutes acceptance of the current version.

---

# 第二部分 — 中文版《Everywhere 模组生态与可接受使用政策》

本政策规定了面向 **Everywhere**（`Sylinko/Everywhere`）生态开发、提交或分发模组（Mod）所适用的许可架构、品牌边界、安全与内容规范、平台完整性要求及执行机制。本政策的中英文版本具有同等说明目的；如译文与英文原文存在理解偏差，以英文文本为准。

## 一、适用范围与效力说明

1. **平台准入与分发效力**：本政策作为 **Sylinko Inc.**（以下简称“Sylinko”）所运营之 Everywhere 官方模组仓库、精选索引、官方签名与信任标识体系的准入、可接受使用及分发管理规范。
2. **与开源软件许可证的关系**：本政策独立于 `Sylinko/Everywhere` 与 `Sylinko/skills` 代码库所适用的开源许可证原文，不构成对 GNU Affero General Public License v3.0（`AGPL-3.0-only`）或 Apache License 2.0（`Apache-2.0`）条文自身的修改。
3. **与《模组接口附加许可》的衔接**：本政策第六条（平台完整性与安全边界）所界定的技术接口范围，同时构成 GNU AGPLv3 第 7 条下《模组接口附加许可》（正文见 `Sylinko/Everywhere` 仓库根目录的 `EXCEPTIONS` 文件）的适用前提。超出该技术边界的模组不适用该附加许可。

## 二、许可架构与品牌权利

1. **模组开发套件、契约库与技能文档（`Apache-2.0`）**：
   - `Everywhere.Versioning`、`Everywhere.Modding.Abstractions`、`Everywhere.Modding.Packaging`、`Everywhere.Modding.Sdk`、官方示例模组 `Everywhere.Mods.Sample` 以及 `Sylinko/skills` 技能知识库（`everywhere`、`everywhere-mod`）依据 **Apache License 2.0（`Apache-2.0`）** 授权。
2. **Everywhere 宿主程序与官方第一方模组（`AGPL-3.0-only` 及商业许可）**：
   - Everywhere 宿主应用程序（`Everywhere.Core` 及各平台宿主工程）与 Sylinko 发布的官方模组（例如 `everywhere.everything`、`everywhere.themes`）依据 **GNU Affero General Public License v3.0（`AGPL-3.0-only`）** 发布，同时由 Sylinko Inc.（`contact@sylinko.com`）提供单独的商业授权许可（详见 `Sylinko/Everywhere` 仓库根目录的 `LICENSE-COMMERCIAL.md`）。
3. **独立第三方模组（模组接口附加许可）**：
   - 在符合 GNU AGPLv3 第 7 条下《模组接口附加许可》的前提下，凡仅通过已公开的模组加载与宿主服务接口与 Everywhere 交互、不修改或再分发 Everywhere 宿主本体，且遵守本政策第六条的独立第三方模组，作者可自主选择 **任意许可协议**（包括 `Apache-2.0`、`MIT`、`GPL-3.0` 等开源协议或专有商业协议）进行分发。
   - 作者应当通过 `<EverywhereModLicense>` 如实声明许可标识，并在适用时通过 `<EverywhereModLicenseFile>` 与 `ThirdPartyFiles` 随包附带许可文本及第三方归属声明。
4. **品牌标识与保留视觉资产（All Rights Reserved）**：
   - 品牌名称 **Sylinko**、由 Sylinko Inc. 官方发行的 **Everywhere** 产品身份标识，以及 `Sylinko/Everywhere` 仓库根目录 `TRADEMARK.md` 中列明的原创徽标、应用图标、横幅与视觉资产均由 Sylinko Inc. 保留所有权利。
   - 任何代码开源许可证均不授予复用上述保留视觉资产，或以暗示官方出身、隶属关系或官方背书的方式展示第三方模组或衍生版本的权利。

## 三、安全、隐私与透明性要求

1. **禁止恶意功能**：模组不得包含、投递或协助实施恶意软件、木马、后门、键盘记录、凭证/令牌/Cookie/钱包信息窃取、勒索加密、未经授权的加密货币挖矿、间谍监控、拒绝服务攻击或蓄意破坏用户系统稳定性的代码。
2. **可审查性与未声明载荷限制**：提交至官方或精选仓库的模组不得采用恶意混淆、加壳、加密载荷、反分析对抗手段，亦不得未经声明地静默下载并执行外部可执行程序或脚本。
3. **能力披露与遥测规范**：
   - 当模组读写其专属 `DataDirectory` 以外的本地文件、启动外部进程、发起外部网络通信，或监听系统输入与屏幕状态时，应当在 README 及权限元数据中清晰、准确地披露相关行为与用途。
   - 任何统计分析或遥测收集须默认关闭、仅限用户明确主动同意（Opt-in）后启用，并予以透明说明。

## 四、内容合规要求

面向 Everywhere 官方或社区索引分发、提交的模组，不得包含、生成、链接或传播以下内容：
- 色情露骨内容（NSFW）或任何形式的未成年人色情与剥削虐待内容（CSAM）；
- 仇恨言论、骚扰恐吓、人肉搜索、未经同意披露他人隐私数据、暴力威胁或煽动极端暴力内容；
- 钓鱼欺诈、身份仿冒，或专门用于协助实施违法行为及未授权系统侵入的工具。

## 五、知识产权与保留标识符

1. **第三方权利合规**：作者须对模组包内所含的全部源代码、二进制文件、图标、字体、主题与第三方依赖拥有合法权利或有效授权，并完整保留相应的版权与许可声明。
2. **官方保留命名空间**：**`everywhere.*`** 与 **`sylinko.*`** 模组标识符前缀专供 Sylinko Inc. 官方模组保留使用。第三方模组应当采用作者专属前缀或反向域名规范（例如 `alice.translator`、`io.github.alice.pomodoro`），不得仿冒 Sylinko 官方模组。

## 六、平台完整性与安全边界

鉴于模组运行于宿主应用程序的托管进程内，模组必须与宿主核心平台服务及安全控制保持清晰隔离。模组**不得**通过任何技术手段——包括但不限于非公开成员反射、运行时 IL 补丁（如 Harmony、MonoMod 或 Detour）、替换依赖注入容器服务、篡改宿主配置文件、网络流量劫持或界面遮挡隐藏——对以下任一 Everywhere 核心子系统实施修改、隐藏、伪造、拦截、替换、禁用、绕过或降级：

1. **账号与认证子系统**：登录流程、OAuth 凭证与令牌、会话状态、设备标识及账号或授权权益校验逻辑；
2. **云同步子系统**：同步传输协议、服务端点、配额限制及用户数据同步完整性；
3. **官方模型网关**：官方预设模型访问、速率限制、计费与配额计量、网关路由及上游安全请求头；
4. **模组验证与仓库信任控制机制**：模组加密签名验证以及官方仓库验证标识与信任状态提示；
5. **官方链接、归属声明与赞助渠道**：软件内任何指向 Everywhere 官方代码仓库、官方网站、文档、许可与版权声明，或官方赞助与捐赠渠道的链接、引用及界面元素。

凡突破本条所述技术边界的模组，均超出 GNU AGPLv3 第 7 条下《模组接口附加许可》的授权范围。

## 七、违规处理、兼容性免责与政策修订

1. **违规处置措施**：对于违反本政策的模组包、标识符或仓库来源，Sylinko 有权予以拒绝收录、下架移除、撤销数字签名信任或列入拦截清单；在涉及用户安全或法律合规紧急风险时，有权不经事先通知直接采取处置措施。
2. **非公开内部实现无兼容性承诺**：宿主未公开的内部类型与成员不构成受支持的模组 API 契约，可能在任意版本更新中调整或移除，恕不另行通知。
3. **独立责任与免责声明**：第三方模组由其各自发布者独立承担全部法律与安全责任。Sylinko 不对第三方模组的可用性、安全性或合法性作出担保，亦不因安装或使用第三方模组所导致的任何损害承担责任。
4. **政策修订**：Sylinko 可根据平台能力演进、安全要求或法律法规变化修订本政策。继续向 Everywhere 官方渠道提交或分发模组即视为接受修订后的现行政策。
