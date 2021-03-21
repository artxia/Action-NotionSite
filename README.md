# Action-NotionSite

loconotion + Github Actions + Notion + GitHub Pages + Cloudflare Workers, Create a free website or build a blog.<br>
GitHub 建 blog 的项目蛮多的，但是发布管理还是不够方便。Notion 本身是一款功能丰富的非常优秀的笔记软件，这个项目借助 Notion 搭建网站，做到了免费、快捷、方便、易用。一次部署再也不用管他。

[English Version](#english-version)

Examples 示例站 [leoncvlt.com](https://leoncvlt.com/)

- 利用 Github Actions 定时运行 loconotion 抓取 Notion 页面，生成纯静态 html 页面。
- 将生成的 html 页面推送到 GitHub 仓库，借助 GitHub Pages 生成静态 web 网站。
- 可选，最后用 Cloudflare Workers 进行反向代理，实现独立域名网站。
- 每20分钟运行一次

# Usage

1. 注册并在 Notion 创建你心仪的页面，点分享获得公开链接例如：`https://www.notion.so/Loconotion-Example-Page-03c40xxxxxxxxxxxxxxxx9a8950ef`
2. 注册 GitHub, 点击本项目页面中的 **use this template** （使用这个模版）按钮，创建属于你的新项目。新仓库名字建议用 **blog** 之类。（创建完回来可以顺便给本项目点个 Star）
3. 在你刚新建的 GitHub 项目里面点 **Settings**（设置）然后点 **Secrets**（隐私）新建配置文件。详细配置文件见[原项目说明](https://github.com/leoncvlt/loconotion#advanced-usage)
<details>
<summary>配置文件示例</summary>

**Name:**<br>
`SITE_CONFIG`<br>
**Value:**<br>
```
name = "notion"
page = "https://www.notion.so/Loconotion-Example-Page-03c40xxxxxxxxxxxxxxxx9a8950ef"
theme = "dark"
[site]

  [[site.meta]]
  name = "title"
  content = "Loconotion Test Site"

  [[site.meta]]
  name = "description"
  content = "A static site generated from a Notion.so page using Loconotion"

  [site.fonts]
  site = 'Nunito'
  navbar = ''
  title = 'Montserrat'
  h1 = 'Montserrat'
  h2 = 'Montserrat'
  h3 = 'Montserrat'
  body = ''
  code = ''

  [[site.inject.head.link]]
  rel="icon"
  sizes="16x16"
  type="image/png"
  href="/example/favicon-16x16.png"

  [[site.inject.body.script]]
  type="text/javascript"
  src="/example/custom-script.js"

[pages]

  [pages.d2fa06f244e64f66880bb0491f58223d]
    slug = "games-list"

    [[pages.d2fa06f244e64f66880bb0491f58223d.meta]]
    name = "description"
    content = "A fullscreen list database page, now with a pretty slug"

    [pages.d2fa06f244e64f66880bb0491f58223d.fonts]
    body = 'DM Mono'

  [pages.54dab6011e604430a21dc477cb8e4e3a]
    slug = "film-gallery"

  [pages.2604ce45890645c79f67d92833083fee]
    slug = "books-table"

  [pages.ae0a85c527824a3a855b7f4d31f4e0fc]
    slug = "random-board"
```

</details>

4. 在你刚建的 GitHub 项目里点 **Actions** 然后点左侧 **Deploy to Pages** 切换，然后点 **Run workflow** 开始第一次运行生成 Pages (看不到 Auto Install 的话，点开 .yml 文件随便加一空行保存)。这里生成完了以后一定要先检查一下你这个项目下是否生成了新的分支 **gh-pages** ，看看该分支内是否有文件夹和 html 文件。
5. 在你刚新建的 GitHub 项目里面点 **Settings**（设置）然后下拉，找到 **GitHub Pages**, 选择 `gh-pages / root`
保存后就能开启 GitHub Pages. 接下来就可以使用 `username.github.io/repository/name` 来访问你的静态网站了。这里的 username 是你的 GitHub 名字、repository 是你的项目名字、后面的 name 是配置文件第一行的名字，也对应 gh-pages 分支下的文件夹。
6. 可选。注册并在 Cloudflare 上新增你的域名。新建 Workers， 使用下面的代码对 `username.github.io/repository/name` 进行反代。然后对应域名新建路由对应 Workers 即可实现域名访问。这一步网上很多教程。（如果你建立的是主 GitHub Pages 仓库`<user>.github.io`，则可以直接使用 GitHub Pages 自带的 Custom domain 来实现独立域名。[GitHub 文档](https://docs.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site)）
<details>
<summary>workers.js</summary>

from:[booster.js](https://github.com/xiaoyang-liu-cs/booster.js)
```
const config = {
  basic: {
    upstream: 'https://en.wikipedia.org/',
    mobileRedirect: 'https://en.m.wikipedia.org/',
  },

  firewall: {
    blockedRegion: ['CN', 'KP', 'SY', 'PK', 'CU'],
    blockedIPAddress: [],
    scrapeShield: true,
  },

  routes: {
    TW: 'https://zh.wikipedia.org/',
    HK: 'https://zh.wikipedia.org/',
    FR: 'https://fr.wikipedia.org/',
  },

  optimization: {
    cacheEverything: false,
    cacheTtl: 5,
    mirage: true,
    polish: 'off',
    minify: {
      javascript: true,
      css: true,
      html: true,
    },
  },
};

async function isMobile(userAgent) {
  const agents = ['Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPad', 'iPod'];
  return agents.any((agent) => userAgent.indexOf(agent) > 0);
}

async function fetchAndApply(request) {
  const region = request.headers.get('cf-ipcountry') || '';
  const ipAddress = request.headers.get('cf-connecting-ip') || '';
  const userAgent = request.headers.get('user-agent') || '';

  if (region !== '' && config.firewall.blockedRegion.includes(region.toUpperCase())) {
    return new Response(
      'Access denied: booster.js is not available in your region.',
      {
        status: 403,
      },
    );
  } if (ipAddress !== '' && config.firewall.blockedIPAddress.includes(ipAddress)) {
    return new Response(
      'Access denied: Your IP address is blocked by booster.js.',
      {
        status: 403,
      },
    );
  }

  const requestURL = new URL(request.url);
  let upstreamURL = null;

  if (userAgent && isMobile(userAgent) === true) {
    upstreamURL = new URL(config.basic.mobileRedirect);
  } else if (region && region.toUpperCase() in config.routes) {
    upstreamURL = new URL(config.routes[region.toUpperCase()]);
  } else {
    upstreamURL = new URL(config.basic.upstream);
  }

  requestURL.protocol = upstreamURL.protocol;
  requestURL.host = upstreamURL.host;
  requestURL.pathname = upstreamURL.pathname + requestURL.pathname;

  let newRequest;
  if (request.method === 'GET' || request.method === 'HEAD') {
    newRequest = new Request(requestURL, {
      cf: {
        cacheEverything: config.optimization.cacheEverything,
        cacheTtl: config.optimization.cacheTtl,
        mirage: config.optimization.mirage,
        polish: config.optimization.polish,
        minify: config.optimization.minify,
        scrapeShield: config.firewall.scrapeShield,
      },
      method: request.method,
      headers: request.headers,
    });
  } else {
    const requestBody = await request.text();
    newRequest = new Request(requestURL, {
      cf: {
        cacheEverything: config.optimization.cacheEverything,
        cacheTtl: config.optimization.cacheTtl,
        mirage: config.optimization.mirage,
        polish: config.optimization.polish,
        minify: config.optimization.minify,
        scrapeShield: config.firewall.scrapeShield,
      },
      method: request.method,
      headers: request.headers,
      body: requestBody,
    });
  }

  const fetchedResponse = await fetch(newRequest);

  const modifiedResponseHeaders = new Headers(fetchedResponse.headers);
  if (modifiedResponseHeaders.has('x-pjax-url')) {
    const pjaxURL = new URL(modifiedResponseHeaders.get('x-pjax-url'));
    pjaxURL.protocol = requestURL.protocol;
    pjaxURL.host = requestURL.host;
    pjaxURL.pathname = pjaxURL.path.replace(requestURL.pathname, '/');

    modifiedResponseHeaders.set(
      'x-pjax-url',
      pjaxURL.href,
    );
  }

  return new Response(
    fetchedResponse.body,
    {
      headers: modifiedResponseHeaders,
      status: fetchedResponse.status,
      statusText: fetchedResponse.statusText,
    },
  );
}

// eslint-disable-next-line no-restricted-globals
addEventListener('fetch', (event) => {
  event.respondWith(fetchAndApply(event.request));
});
```

</details>

# Tips
- 访问速度取决于你访问 GitHub 的速度。
- 使用 Cloudflare 免费 Workers 需要注意每日访问量。Cloudflare 具有 CDN 效果，可以一定程度增加访问速度
- 本项目模版只是一个快速搭建的项目，核心项目才是抓取 Notion 的关键。有任何关于“抓去”“生成”的问题，建议直接去核心项目。>[loconotion](https://github.com/leoncvlt/loconotion)
- 默认没有开启每20分钟运行，需要自行修改 `.github/workflows/pages_deploy.yml` 文件，将第9、10行前的两个“#”号去掉就能开启

# English Version
by [TechCrunch](https://www.deepl.com/)

- Use Github Actions to run loconotion regularly to crawl Notion pages and generate pure static html pages.
- Push the generated html pages to the GitHub repository and generate a static web site with GitHub Pages.
- Optional. Finally, reverse proxy with Cloudflare Workers to implement a separate domain site.
- Runs every 20 minutes

### Usage

1. Register and create a page of your choice in Notion, click share to get the public link e.g. `https://www.notion.so/Loconotion-Example-Page-03c40xxxxxxxxxxxxxxxx9a8950ef`
2. To sign up for GitHub, click the **use this template** button on this project page to create your new project. We recommend using something like **blog** for the name of your new repository. (You can give this project a Star when you're done creating it)
3. In the GitHub project you just created, click **Settings** and then** Secrets** to create a new profile. Detailed profile of the [original project](https://github.com/leoncvlt/loconotion#advanced-usage)
<details>
<summary>Example configuration file</summary>

**Name:**<br>
`SITE_CONFIG`<br>
**Value:**<br>
```
name = "notion"
page = "https://www.notion.so/Loconotion-Example-Page-03c40xxxxxxxxxxxxxxxx9a8950ef"
theme = "dark"
[site]

  [[site.meta]]
  name = "title"
  content = "Loconotion Test Site"

  [[site.meta]]
  name = "description"
  content = "A static site generated from a Notion.so page using Loconotion"

  [site.fonts]
  site = 'Nunito'
  navbar = ''
  title = 'Montserrat'
  h1 = 'Montserrat'
  h2 = 'Montserrat'
  h3 = 'Montserrat'
  body = ''
  code = ''

  [[site.inject.head.link]]
  rel="icon"
  sizes="16x16"
  type="image/png"
  href="/example/favicon-16x16.png"

  [[site.inject.body.script]]
  type="text/javascript"
  src="/example/custom-script.js"

[pages]

  [pages.d2fa06f244e64f66880bb0491f58223d]
    slug = "games-list"

    [[pages.d2fa06f244e64f66880bb0491f58223d.meta]]
    name = "description"
    content = "A fullscreen list database page, now with a pretty slug"

    [pages.d2fa06f244e64f66880bb0491f58223d.fonts]
    body = 'DM Mono'

  [pages.54dab6011e604430a21dc477cb8e4e3a]
    slug = "film-gallery"

  [pages.2604ce45890645c79f67d92833083fee]
    slug = "books-table"

  [pages.ae0a85c527824a3a855b7f4d31f4e0fc]
    slug = "random-board"
```
</details>

4. Click **Actions** in the GitHub project you just created, then click **the Deploy to Pages** toggle on the left, and then click **Run workflow** to start generating Pages for the first time.(If you don't see Auto Install, click on the .yml file and add a blank line to save it).
After generating here, make sure to check if a new branch **gh-pages** has been created under your project and see if there are folders and html files in that branch.
5. In the GitHub project you just created, click **Settings** and scroll down to **GitHub Pages** and select `gh-pages / root`.
Save it and you'll be able to start GitHub Pages.
Next, you can use `username.github.io/repository/name` to access your static site.
The username is your GitHub name, the repository is the name of your project, and the name on the first line of the configuration file corresponds to the folder under the gh-pages branch.
6. Optional. Register and add your domain on Cloudflare. Create a new Worker, use the following code to reverse proxy `username.github.io/repository/name` Then create a new route corresponding to the domain name corresponding to Workers to achieve domain access. This step online many tutorials.(If you are setting up the main GitHub Pages repository `<user>.github.io`, you can directly use the Custom domain that comes with GitHub Pages to implement a separate domain. [GitHub Docs](https://docs.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site))
<details>
<summary>workers.js</summary>

from:[booster.js](https://github.com/xiaoyang-liu-cs/booster.js)
```
const config = {
  basic: {
    upstream: 'https://en.wikipedia.org/',
    mobileRedirect: 'https://en.m.wikipedia.org/',
  },

  firewall: {
    blockedRegion: ['CN', 'KP', 'SY', 'PK', 'CU'],
    blockedIPAddress: [],
    scrapeShield: true,
  },

  routes: {
    TW: 'https://zh.wikipedia.org/',
    HK: 'https://zh.wikipedia.org/',
    FR: 'https://fr.wikipedia.org/',
  },

  optimization: {
    cacheEverything: false,
    cacheTtl: 5,
    mirage: true,
    polish: 'off',
    minify: {
      javascript: true,
      css: true,
      html: true,
    },
  },
};

async function isMobile(userAgent) {
  const agents = ['Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPad', 'iPod'];
  return agents.any((agent) => userAgent.indexOf(agent) > 0);
}

async function fetchAndApply(request) {
  const region = request.headers.get('cf-ipcountry') || '';
  const ipAddress = request.headers.get('cf-connecting-ip') || '';
  const userAgent = request.headers.get('user-agent') || '';

  if (region !== '' && config.firewall.blockedRegion.includes(region.toUpperCase())) {
    return new Response(
      'Access denied: booster.js is not available in your region.',
      {
        status: 403,
      },
    );
  } if (ipAddress !== '' && config.firewall.blockedIPAddress.includes(ipAddress)) {
    return new Response(
      'Access denied: Your IP address is blocked by booster.js.',
      {
        status: 403,
      },
    );
  }

  const requestURL = new URL(request.url);
  let upstreamURL = null;

  if (userAgent && isMobile(userAgent) === true) {
    upstreamURL = new URL(config.basic.mobileRedirect);
  } else if (region && region.toUpperCase() in config.routes) {
    upstreamURL = new URL(config.routes[region.toUpperCase()]);
  } else {
    upstreamURL = new URL(config.basic.upstream);
  }

  requestURL.protocol = upstreamURL.protocol;
  requestURL.host = upstreamURL.host;
  requestURL.pathname = upstreamURL.pathname + requestURL.pathname;

  let newRequest;
  if (request.method === 'GET' || request.method === 'HEAD') {
    newRequest = new Request(requestURL, {
      cf: {
        cacheEverything: config.optimization.cacheEverything,
        cacheTtl: config.optimization.cacheTtl,
        mirage: config.optimization.mirage,
        polish: config.optimization.polish,
        minify: config.optimization.minify,
        scrapeShield: config.firewall.scrapeShield,
      },
      method: request.method,
      headers: request.headers,
    });
  } else {
    const requestBody = await request.text();
    newRequest = new Request(requestURL, {
      cf: {
        cacheEverything: config.optimization.cacheEverything,
        cacheTtl: config.optimization.cacheTtl,
        mirage: config.optimization.mirage,
        polish: config.optimization.polish,
        minify: config.optimization.minify,
        scrapeShield: config.firewall.scrapeShield,
      },
      method: request.method,
      headers: request.headers,
      body: requestBody,
    });
  }

  const fetchedResponse = await fetch(newRequest);

  const modifiedResponseHeaders = new Headers(fetchedResponse.headers);
  if (modifiedResponseHeaders.has('x-pjax-url')) {
    const pjaxURL = new URL(modifiedResponseHeaders.get('x-pjax-url'));
    pjaxURL.protocol = requestURL.protocol;
    pjaxURL.host = requestURL.host;
    pjaxURL.pathname = pjaxURL.path.replace(requestURL.pathname, '/');

    modifiedResponseHeaders.set(
      'x-pjax-url',
      pjaxURL.href,
    );
  }

  return new Response(
    fetchedResponse.body,
    {
      headers: modifiedResponseHeaders,
      status: fetchedResponse.status,
      statusText: fetchedResponse.statusText,
    },
  );
}

// eslint-disable-next-line no-restricted-globals
addEventListener('fetch', (event) => {
  event.respondWith(fetchAndApply(event.request));
});
```

</details>

### Tips
- The speed of access depends on how fast you can access GitHub.
- Using Cloudflare free workers need to pay attention to the number of daily visits. cloudflare has a CDN effect, which can increase the speed of access to some extent.
- This project template is just a quick build project, the core project is the key to capture Notion. If you have any questions about "crawling" and "generation", we suggest you go directly to the core project. >[loconotion](https://github.com/leoncvlt/loconotion)
- The default is not enabled to run every 20 minutes, you need to modify the `.github/workflows/pages_deploy.yml` file, remove line 9 and 10 "#" signs to enable.

# Acknowledgments
- [loconotion](https://github.com/leoncvlt/loconotion)
- [booster.js](https://github.com/xiaoyang-liu-cs/booster.js)
# License
[MIT](https://github.com/artxia/Action-NotionSite/blob/main/LICENSE) © XIA
