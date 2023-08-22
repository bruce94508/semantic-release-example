# semantic-release-template  
semantic-release with drone ci use case

# semantic-release

 用於自動化的管理語意化版本、套件發行，它定義如下release steps
 1. 驗證執行條件*
 1. 藉由 git tag 取得上次發行資訊
 1. 分析提交訊息*
 1. 驗證發布*
 1. 生成日誌*
 1. 創建 git tag 打版號
 1. 準備發布*
 1. 執行發布*
 1. 成功通知或報錯*

 透過兩種方式控制發行觸發、中斷、階段內容
 1. 設定 semantic-release
 2. 設定 plugins

 reference
 1. [semantic-release github](https://github.com/semantic-release/semantic-release)
 2. [plugin list](https://semantic-release.gitbook.io/semantic-release/extending/plugins-list)


## 設定 semantic-release
 1. branches: 在哪些分支執行發布
     - name: 如 "master", "release/*"
     - channel: 發布頻道如 "release"
     - prerelease: 如 true 或 "beta"
       - { name: "release/*", channel: "release", prerelease: "apple" } => 1.1.0-apple.1
       - { name: "banana", channel: "release", prerelease: true } => => 1.1.0-banana.1
 2. repositoryUrl
 3. tagFormat: git tag 格式, 如 "${version}"
 3. plugins: 如 [["pluginName",options]]


## plugins
 - 單個插件可能涉及一或多個發行階段 (*星號標記部分)
 - semantic-release 依序執行各階段時，會各掃一遍所有插件，找出有實作這階段的插件來執行
 - 分析提交訊息被多個插件實作時，會採用其中最高層級的發行類型
 - 傳入配置項時，將會覆蓋而不是合併其預設值

### @semantic-release/commit-analyzer
 - 預設 preset: [AngularJS Git Commit Message Conventions](https://docs.google.com/document/d/1QrDFcIiPjSLDn3EL15IJygNPiHORgU1_OOAqWjiDU5Y/edit#heading=h.uyo6cb12dt6w)
 - 分析提交訊息、類型，以決定下次發行版號
 ```
 <type>(<scope>): <subject>
 <BLANK LINE>
 <body>
 <BLANK LINE>
 <footer> (BREAKING CHANGE)
 ```
 - [語意化版本 Semantic Versioning (semver)](https://semver.org/)
   - major 主版號: api行為改變，且與舊版不相容
   - minor 次版號: 新增功能，相容舊版
   - patch 修訂號: bug修復，相容舊版
 - 發行規則 releaseRules
   - 類型、版號部位之對應
   - 出現某類型的 commit 時，要遞增哪一部位之版號
   - ex: [{ "type": "refactor", "release": "patch" }, ...]

### @semantic-release/release-notes-generator
 - 依照提交訊息分析結果，自動產生日誌內容

### @semantic-release/changelog
 - 準備階段: 依照產生的日誌內容，創建或更新日誌檔

### @semantic-release/git
 - 準備階段: 推送提交

### @semantic-release/npm
 - 準備階段: 更新 package.json 檔
 - 執行階段: 發行 npm 套件，我們目前未使用

## CI

### install
 - npm install 依照 package.json devDependencies 安裝特定版本 semantic-release 及插件
 - 也可以不事先安裝，在 release 階段隨裝隨用

### release
 - 觸發條件: 推送到 master 或 release 分支時觸發
 - 環境變數: 配置與 git 有關環境變數
   - GIT_CREDENTIALS: 格式 username:password，需進bitbucket帳號個人設定頁面產生
   - 詳 drone.yml 範例
 - npx semantic-release
   - 將會優先使用本地已安裝的套件
   - 若無，則會下載最新版本
   - 可以 npx semantic-release@17.4.2 指定套件版本

### push; publish
 - 觸發條件: 打版號標籤
 - 打包 docker 映像檔，推送到私有庫
