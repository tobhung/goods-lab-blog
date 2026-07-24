# 好物研究所

知識型生活選物網站，用 [Jekyll](https://jekyllrb.com/) 打造，主打**開箱文**與**收納推薦**兩大分類，設計為部署在 GitHub Pages，並可串接 Cloudflare 上的自訂網域。

## 網站結構

```
goods-lab-blog/
├── _config.yml              # 網站設定（標題、網址、外掛…）
├── _layouts/                # 版型：default（外殼）、post（文章）、page（一般頁面）
├── _includes/                # 共用區塊：head、header（導覽列）、footer
├── _posts/                  # 所有文章都放這裡，檔名格式 YYYY-MM-DD-標題.md
├── assets/css/style.css     # 全站樣式
├── assets/images/posts/     # 文章封面圖（目前是 SVG 佔位圖，建議換成實拍照片）
├── index.html                # 首頁
├── category-unboxing.html   # 開箱文分類頁
├── category-storage.html    # 收納推薦分類頁
├── about.md                  # 關於我們
└── 404.html                  # 找不到頁面
```

## 本機預覽

需要先安裝 Ruby（建議用 [rbenv](https://github.com/rbenv/rbenv) 或 [rvm](https://rvm.io/) 管理版本）。

```bash
cd goods-lab-blog
bundle install
bundle exec jekyll serve
```

啟動後打開 `http://localhost:4000` 就能看到網站，存檔後會自動重新整理。

## 怎麼新增一篇文章

1. 在 `_posts/` 新增一個檔案，檔名格式：`YYYY-MM-DD-文章英文短網址.md`
   例如：`2026-08-01-air-fryer-unboxing.md`
2. 檔案開頭加上 front matter：

   ```yaml
   ---
   title: "文章標題"
   date: 2026-08-01 09:00:00 +0800
   categories: [開箱文]   # 或 [收納推薦]
   tags: [標籤1, 標籤2]
   image: /assets/images/posts/你的封面圖.jpg
   ---
   ```

3. 下面直接寫 Markdown 內文即可，首頁跟分類頁會自動抓到這篇文章，不用手動加連結。
4. 封面圖建議放在 `assets/images/posts/`，尺寸建議 16:10（例如 1200x750）。

分類目前只用兩個字串：`開箱文`、`收納推薦`。如果要新增第三個分類，記得同步：
- 複製一份 `category-unboxing.html`，把 `site.categories['開箱文']` 改成新分類名稱
- 在 `_includes/header.html` 跟 `_includes/footer.html` 加上新的導覽連結

## 部署到 GitHub Pages

1. 在 GitHub 建立一個新的 repository（例如 `goods-lab-blog`）。
2. 在這個資料夾裡初始化 git 並推上去：

   ```bash
   git init
   git add .
   git commit -m "Initial commit: 好物研究所"
   git branch -M main
   git remote add origin https://github.com/你的帳號/goods-lab-blog.git
   git push -u origin main
   ```

3. 到 GitHub repo 的 **Settings → Pages**：
   - Source 選擇 `Deploy from a branch`
   - Branch 選 `main`，資料夾選 `/ (root)`
   - 存檔後等 1-2 分鐘，GitHub 會自動用 Jekyll build 並給你一個 `https://你的帳號.github.io/goods-lab-blog/` 網址

> 如果之後要串自訂網域（見下方），記得回來把 `_config.yml` 裡的 `url` 改成你的正式網址，例如 `"https://www.example.com"`，這樣 SEO meta 標籤、sitemap、RSS 才會產生正確的絕對網址。

## 串接 Cloudflare 網域

假設你的網域已經在 Cloudflare 管理 DNS。依你要用**根網域**（example.com）還是**子網域**（www.example.com / blog.example.com），設定方式不同：

### 情境 A：子網域（推薦，設定較單純）

1. Cloudflare DNS 新增一筆紀錄：
   - Type: `CNAME`
   - Name: `www`（或你想用的子網域，例如 `blog`）
   - Target: `你的帳號.github.io`
   - Proxy status：**先設成 DNS only（灰色雲朵）**，等 GitHub 憑證簽發完成後再打開 Proxy（橘色雲朵）
2. 在 repo 根目錄新增一個檔案叫 `CNAME`（沒有副檔名），內容就是你的網域，例如：
   ```
   www.example.com
   ```
   或直接在 **Settings → Pages → Custom domain** 欄位輸入網域，GitHub 會自動幫你建立這個檔案並 commit 回 repo。
3. 等 DNS 生效（通常幾分鐘到數小時），GitHub Pages 會自動簽發 HTTPS 憑證，簽發完成後回到 Settings → Pages 勾選 **Enforce HTTPS**。

### 情境 B：根網域（example.com，不加 www）

1. Cloudflare DNS 新增 4 筆 `A` 紀錄，Name 都填 `@`，分別指向 GitHub Pages 的 IP：
   ```
   185.199.108.153
   185.199.109.153
   185.199.110.153
   185.199.111.153
   ```
   （可選）再加一組 `AAAA` 紀錄指向 GitHub 的 IPv6，資料可參考 [GitHub 官方文件](https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site)。
2. 其餘步驟跟情境 A 一樣：加 `CNAME` 檔案（內容填根網域）、Settings → Pages 設定自訂網域、等憑證簽發後開啟 Enforce HTTPS。

### Cloudflare SSL/TLS 模式提醒

在 Cloudflare 的 **SSL/TLS** 設定裡，把加密模式設成 **Full**（不要用 Flexible），否則容易出現重新導向迴圈（ERR_TOO_MANY_REDIRECTS）。等 GitHub 那邊的憑證簽發、Enforce HTTPS 開啟後，再把 Cloudflare 的 Proxy 打開（橘色雲朵），這樣才能同時吃到 Cloudflare 的 CDN／防護，又不會卡在憑證驗證階段。

## 之後可以做的優化

- 把 `assets/images/posts/` 裡的 SVG 佔位圖換成實拍照片或去背素材圖
- 幫每篇文章寫更完整的 `description`（可透過 front matter 加 `excerpt` 覆蓋首段自動摘要）
- 如果文章量變多，可以考慮加上分頁（`jekyll-paginate`，GitHub Pages 有原生支援）
- 想要留言功能可以串接 Giscus（用 GitHub Discussions，免費且不用自架後端）
