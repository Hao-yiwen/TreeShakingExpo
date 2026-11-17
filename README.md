# TreeShakingExpo

基于 **Expo SDK 52 canary** 与 **Expo Router** 的跨端示例。界面沿用 `create-expo-app` 的 Tabs 起步页，并在 Metro 上打开了实验性的包体积优化与 tree shaking，用来导出 iOS、Android 和 Web 包。

[![Expo](https://img.shields.io/badge/Expo-52.0.0--canary-000020?logo=expo&logoColor=white)](https://expo.dev)
[![React Native](https://img.shields.io/badge/React_Native-0.75.2-20232A?logo=react&logoColor=61DAFB)](https://reactnative.dev)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.3.3-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)

仓库里没有独立的业务功能，也没有摇树前后的对比数据。页面仍是模板自带的 Home / Explore 说明页；和 tree shaking 相关的只有 Metro 配置和 `export` 脚本。

## 页面与内容

| 路由 | 文件 | 实际内容 |
| --- | --- | --- |
| `/` | `app/(tabs)/index.tsx` | Home。视差头图、挥手动画，以及三段起步说明（改首页、打开 Explore、运行 `reset-project`） |
| `/explore` | `app/(tabs)/explore.tsx` | Explore。可折叠介绍文件路由、三端运行、图片倍率、SpaceMono 字体、浅色/深色、Reanimated 动画 |
| 未匹配路由 | `app/+not-found.tsx` | 提示页面不存在，并链回首页 |
| Web 文档壳 | `app/+html.tsx` | 静态导出时的根 HTML，按系统浅色/深色设置页面背景 |

另外包含：

- 底部两个 Tab（Home、Explore），图标来自 `@expo/vector-icons` 的 Ionicons
- 浅色 / 深色配色（`constants/Colors.ts`）。原生端跟随系统；Web 端 `hooks/useColorScheme.web.ts` 固定返回 `light`，Tab 配色因此始终按浅色计算
- 主题文字与容器、外链（原生端用应用内浏览器打开）、可折叠区块、视差滚动
- 一条 Jest 快照测试：`components/__tests__/ThemedText-test.tsx`
- 已提交的 Android / iOS 原生工程（`expo prebuild` 产物），以及一份 Metro 包 `ios/main.jsbundle1`（约 2.0 MB）。该文件不是 `npm run export` 的输出

## 技术栈

版本以 `package.json` 与 `package-lock.json` 锁定结果为准。

| 项目 | 版本 |
| --- | --- |
| Expo | `52.0.0-canary-20240904-69100c1` |
| Expo Router | `4.0.0-canary-20240904-69100c1` |
| React / React DOM | `18.3.1` |
| React Native | `0.75.2`（`engines.node` 为 `>=18`） |
| react-native-web | `~0.19.10` |
| react-native-reanimated | `~3.15.0` |
| React Navigation | `@react-navigation/native` `^6.0.2` |
| TypeScript | `5.3.3`（声明为 `~5.3.3`） |
| Jest / jest-expo | `29.7.0` / 与 Expo 相同的 canary |
| 应用版本 | `1.0.0` |

应用标识（`app.json` 与原生工程一致）：

- 显示名 `TreeShakingExpo`，URL scheme `myapp`
- iOS Bundle ID 与 Android 包名均为 `com.haoyiwen.TreeShakingExpo`
- 竖屏；`userInterfaceStyle` 为 `automatic`
- Web 使用 Metro，`output` 为 `static`，并开启 `experiments.typedRoutes`
- JS 引擎为 Hermes。Android `newArchEnabled=false`（未开新架构）
- Android：`minSdk 23`，`compileSdk` / `targetSdk 34`，Gradle `8.8`，Kotlin `1.9.24`
- iOS 部署目标 `15.1`（`ios/Podfile`）

## 目录

```text
.
├── app/                     Expo Router 路由
│   ├── _layout.tsx          根 Stack、SpaceMono 字体、启动屏
│   ├── +html.tsx            Web 根 HTML
│   ├── +not-found.tsx
│   └── (tabs)/              Home 与 Explore
├── assets/                  图标、启动图、React Logo、SpaceMono 字体
├── components/              主题组件、动画、导航图标；含一条 Jest 测试
├── constants/Colors.ts      浅色 / 深色色板
├── hooks/                   配色方案与主题色
├── scripts/reset-project.js 把现有 app/ 挪到 app-example/ 并生成空白页
├── android/                 已生成的 Android 工程
├── ios/                     已生成的 iOS 工程，另含 main.jsbundle1
├── app.json                 Expo 配置
├── metro.config.js          打开 experimentalImportSupport
├── babel.config.js          babel-preset-expo
└── tsconfig.json            继承 expo/tsconfig.base，路径别名 @/*
```

`node_modules/`、`.expo/`、`dist/` 已在 `.gitignore` 中。

## Tree shaking 相关配置

业务代码没有额外的摇树示例。导出时是否启用实验优化，只由下面两处决定。

`metro.config.js` 为转换器打开 Metro 的实验性 import 支持：

```javascript
config.transformer.getTransformOptions = async () => ({
  transform: {
    experimentalImportSupport: true,
  },
});
```

`package.json` 的 `export` 脚本：

```bash
EXPO_UNSTABLE_METRO_OPTIMIZE_GRAPH=1 EXPO_UNSTABLE_TREE_SHAKING=1 expo export --no-bytecode
```

这一版 Expo CLI 的行为是：

- 只打开 `EXPO_UNSTABLE_TREE_SHAKING`、不打开 `EXPO_UNSTABLE_METRO_OPTIMIZE_GRAPH` 时，CLI 会报错退出。当前 `export` 脚本两个都设置了
- 两者都打开后，导出日志会打印 `Experimental bundle optimization is enabled.` 与 `Experimental tree shaking is enabled.`
- `--no-bytecode` 不生成 Hermes 字节码。CLI 会提示这只适合调试，正式包不建议关掉字节码

这些开关都带 `UNSTABLE`，只在当前这份 canary 上按上述方式接线，不保证后续 SDK 的默认行为相同。

## 环境要求

- Node.js 18 或更高（本仓库在 Node.js `v22.23.4`、npm `10.9.9` 上安装并跑过下列命令）
- 开发服务器：`npm start` 后，终端会列出 Expo Go 与模拟器选项。依赖是 2024-09-04 的 SDK 52 canary，商店版 Expo Go 不一定能加载这份运行时。本环境没有手机或模拟器，未验证 Expo Go
- `npm run android`：Android SDK、模拟器或设备。当前 Linux 环境没有 `adb`，未编译 Android
- `npm run ios`：macOS、Xcode、CocoaPods。`ios/` 已入库，但没有 `Pods/`，需要在 Mac 上先 `pod install`。本环境无法编译 iOS

## 安装与运行

```bash
npm ci
```

启动 Metro 开发服务器（交互式，终端里可选 Expo Go、模拟器或按 `w` 打开 Web）：

```bash
npm start
```

只开 Web 开发服务器：

```bash
npm run web
```

用实验性 tree shaking 做一次静态导出（产物在 `dist/`，已被 git 忽略）：

```bash
npm run export
```

在本环境执行 `npm run export` 已成功。CLI 确认打开了实验性包优化和 tree shaking，并写出：

- iOS bundle：`dist/_expo/static/js/ios/`（约 1.59 MB）
- Android bundle：`dist/_expo/static/js/android/`（约 1.59 MB）
- Web bundle：`dist/_expo/static/js/web/`（约 1.04 MB）
- 静态路由：`/`、`/explore`、`/_sitemap`、`/+not-found` 等

原生工程编译（本环境未跑通，命令本身与 `package.json` 一致）：

```bash
npm run android   # expo run:android
npm run ios       # expo run:ios，仅 macOS
```

## 脚本

| 命令 | 作用 |
| --- | --- |
| `npm start` | `expo start` |
| `npm run web` | `expo start --web` |
| `npm run android` | `expo run:android` |
| `npm run ios` | `expo run:ios` |
| `npm run export` | 打开实验性 tree shaking 后 `expo export --no-bytecode` |
| `npm run prebuild` | `expo prebuild --no-install --clean`，会清掉并重写 `android/`、`ios/` |
| `npm test` | `jest --watchAll`，进入监听模式，不会自己退出 |
| `npm run lint` | `expo lint`。仓库未声明 `eslint` / `eslint-config-expo`，也没有 ESLint 配置文件 |
| `npm run reset-project` | 把 `app/` 改名为 `app-example/`，再写一份只有一行文字的空白 `app/` |

一次性跑测试（已通过，1 个快照用例）：

```bash
CI=1 npx jest --watchAll=false
```

类型检查不是 npm 脚本，可直接执行（本环境已通过）：

```bash
npx tsc --noEmit
```

## 已核对与未核对

在 Linux 上核对通过：

- `npm ci`
- `npx tsc --noEmit`
- `CI=1 npx jest --watchAll=false`
- `npm run export`（iOS、Android、Web 均导出）

未在本环境核对：交互式 `npm start` / `npm run web`、Android 与 iOS 原生编译、`npm run lint`。仓库内没有应用截图，只有模板图标、启动图和 React Logo，因此这里不放界面图。

## 注意

- `npm run reset-project` 和 `npm run prebuild` 都会改动现有目录，运行前先确认不需要保留当前的 `app/` 或原生工程。
- `app.json` 的 `plugins` 里 `expo-router` 写了两次，旁边还有 `expo-font`。
- 路径别名 `@/*` 指向仓库根目录，页面里的 `@/components`、`@/hooks` 依赖这项配置。

## 参考

- [Expo 文档](https://docs.expo.dev/)
- [Expo Router 简介](https://docs.expo.dev/router/introduction/)
