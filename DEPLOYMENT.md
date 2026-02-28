# 🚀 部署指南

## GitHub Pages 部署步骤

### 1. 准备工作

确保你的代码已经推送到 GitHub 仓库。

### 2. 配置 GitHub Pages

#### 使用 GitHub Actions（推荐）

1. 进入你的 GitHub 仓库
2. 点击 **Settings** > **Pages**
3. 在 **Source** 下拉菜单中选择 **GitHub Actions**
4. 推送代码到 main 分支，GitHub Actions 会自动构建和部署

```bash
cd /Users/jim/0-Phd/Back-to-future/Web2Tool/webNavigator-homepage/game/snake-game
git add .
git commit -m "feat: 添加贪吃蛇游戏"
git push origin main
```

5. 等待 Actions 完成后，访问：`https://YOUR_USERNAME.github.io/game/`

### 3. 配置 base 路径

如果你的仓库名不是 `game`，需要修改 `vite.config.js`：

```javascript
export default defineConfig({
  base: '/your-repo-name/',  // 改为你的仓库名
  // ...
})
```

### 4. 验证部署

部署完成后，访问你的 GitHub Pages 地址：
- 如果仓库名是 `game`：`https://YOUR_USERNAME.github.io/game/`
- 如果是其他名称：`https://YOUR_USERNAME.github.io/your-repo-name/`

## 🔧 故障排除

### 问题：页面显示 404

**解决方案：**
1. 检查 `vite.config.js` 中的 `base` 配置是否与仓库名匹配
2. 确保 GitHub Pages 已启用
3. 检查 GitHub Actions 是否成功运行

### 问题：样式或资源加载失败

**解决方案：**
1. 确保 `base` 路径配置正确
2. 检查浏览器控制台的错误信息
3. 验证资源路径是否正确

### 问题：Node 版本警告

当前项目使用 Node.js 20.13.1，Vite 7 推荐 20.19+。这不影响开发和构建，但如果遇到问题可以升级 Node.js：

```bash
# 使用 nvm 升级
nvm install 20.19
nvm use 20.19
```

## 📊 构建优化

构建后的文件在 `dist` 目录：
- 自动代码分割
- CSS 压缩
- 资源优化
- Tree-shaking

## 🔄 更新部署

每次推送到 main 分支，GitHub Actions 会自动重新部署。

手动部署：
```bash
npm run deploy
```

## 🌐 自定义域名

如果你想使用自定义域名：

1. 在 `public` 目录下创建 `CNAME` 文件
2. 文件内容为你的域名，例如：`snake.yourdomain.com`
3. 在域名提供商处配置 DNS 记录指向 GitHub Pages
