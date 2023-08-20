<!-- ALL-CONTRIBUTORS-BADGE:START - Do not remove or modify this section -->

[![All Contributors](https://img.shields.io/badge/all_contributors-6-orange.svg?style=flat-square)](#contributors)

<!-- ALL-CONTRIBUTORS-BADGE:END -->

# （非公式） Dfinity ドキュメント翻訳プロジェクト

https://dfinityjp.netlify.app

## 翻訳手順

翻訳の状況は、[翻訳の概要と進捗状況](https://github.com/Japan-DfinityInfoHub/portal/issues/1)の issues を確認してください。

### 必要な環境

- Node v16

### 手順 1: 翻訳を始める準備

まずは、このリポジトリを右上から Fork してください。

そして、リポジトリをクローンします。`your` には、あなたの GitHub のユーザーネームを入れてください。

```
$ git clone https://github.com/your/portal
$ cd docs
```

翻訳作業を行うためのブランチを作成します。
どのファイルを翻訳するかは、[翻訳の概要と進捗状況](https://github.com/Japan-DfinityInfoHub/portal/issues/1)の翻訳ページ一覧を確認して、翻訳したい箇所をコメントしてください。

ここでは、例として `introduction/pages/welcome.md` を翻訳するためのブランチを作成します。

```
$ git checkout -b introduction/pages/welcome.md
```

これで、翻訳を始める準備は完了です。エディタを使って、翻訳箇所のファイルを編集します。

### 手順 2: 翻訳

[スタイルガイド](https://github.com/Japan-DfinityInfoHub/portal/blob/master/styleguide.md)に目を通してください。
わからないことがあれば [Discord](https://discord.gg/ewAxzfTURX) の#ドキュメント翻訳チャネルで質問してください。

エディタとしては [VSCode](https://azure.microsoft.com/ja-jp/products/visual-studio-code/) を推奨します。

### 手順 3: 翻訳内容の確認

翻訳した文章を確認します。

```
git submodule update --init
npm install
npm start
```

のコマンドを叩くと、http://localhost:3000 でドキュメントを確認することができます。

### 手順 4: 翻訳内容のプルリクを出す

翻訳が終わったら、ローカルリポジトリにコミットしたあと、自分のリモートリポジトリにプッシュします。
コミットが複数になった場合、なるべく[１つのコミットにまとめて](https://dev.classmethod.jp/articles/git-rebase-fixup/)いただければありがたいですが、難しければそのままでも OK です。

```
$ git add modules/introduction/pages/welcome.md
$ git commit -m "translated: introduction/pages/welcome.md"
$ git push origin introduction/pages/welcome.md
```

最後に、Github から[プルリクを出します](https://qiita.com/samurai_runner/items/7442521bce2d6ac9330b)。
このとき、出し先が Japan-DfinityInfoHub/portal になるようにします。

間違えて本家の dfinity/docs に出してしまわないように気をつけてください。

以上です！メンテナーがレビューをして問題なければマージされます。

## adoc ファイルから md ファイルへの変換

以前のドキュメントは adoc ファイルで作成されており、すでに翻訳した内容を新しいドキュメントに移行するために markdown 形式への変換が必要な場合は以下の手順に従います。

1. ローカルの docs リポジトリを最新にする

以前のドキュメント(adoc ファイル)は

https://github.com/Japan-DfinityInfoHub/docs

で管理されています。
`git clone` するか、すでにしてある場合は `git pull` コマンド等を用いて、ローカルリポジトリを最新にします。

```
git pull
```

2. 変換スクリプトを実行する

`convert.sh` という変換スクリプトがリポジトリのルートディレクトリに含まれていますので、以下のようにファイルを変換します。
ここでは例として、`modules/quickstart/pages/1-quickstart.adoc` を md ファイルに変換します。

```
bash convert.sh modules/quickstart/pages/1-quickstart.adoc
```

変換前のファイルと同じディレクトリに `modules/quickstart/pages/1-quickstart.md` が作成されます。
ここまでが docs リポジトリ側での準備です。

3. 翻訳したいドキュメントを修正する。

次に、portal リポジトリ側で、対応する md ファイルを開きます。
変換スクリプトではコメントアウトされた文章は削除されてしまうので、[スタイルガイド](https://github.com/Japan-DfinityInfoHub/portal/blob/master/styleguide.md) に従って原文を `<!--` と `-->` で囲んでコメントアウトしてください（原文に更新があった場合に変更箇所を参照できるように、原文のコメントアウトがされていることは重要です）。

原文をコメントアウトしたら、先ほど変換した md ファイルの中身をコピーして貼り付けてください。

原文を確認し、翻訳後の日本語が原文と対応していることをご確認ください。

### 注意点

変換スクリプトは、テーブルの変換先が html になってしまいますので、原文に合わせて md のテーブル書式に直してください。

## Contributors ✨

Special thanks to these wonderful people ([emoji key](https://allcontributors.org/docs/en/emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore-start -->
<!-- markdownlint-disable -->
<table>
  <tr>
    <td align="center"><a href="https://github.com/hoosan"><img src="https://avatars.githubusercontent.com/u/40290137?v=4" width="100px;" alt=""/><br /><sub><b>hoosan</b></sub></a></td>
    <td align="center"><a href="https://github.com/tokuryoo"><img src="https://avatars.githubusercontent.com/u/92770268?v=4" width="100px;" alt=""/><br /><sub><b>tokuryoo</b></sub></a></td>
    <td align="center"><a href="https://github.com/gelgoog999"><img src="https://avatars.githubusercontent.com/u/84751541?v=4" width="100px;" alt=""/><br /><sub><b>gelgoog999</b></sub></a></td>
    <td align="center"><a href="https://github.com/pontagon333"><img src="https://avatars.githubusercontent.com/u/87188356?v=4" width="100px;" alt=""/><br /><sub><b>pontagon333</b></sub></a></td>
    <td align="center"><a href="https://github.com/numtet"><img src="https://avatars.githubusercontent.com/u/11040952?v=4" width="100px;" alt=""/><br /><sub><b>numtet</b></sub></a></td>
    <td align="center"><a href="https://github.com/hokosugi"><img src="https://avatars.githubusercontent.com/u/38212038?v=4" width="100px;" alt=""/><br /><sub><b>hokosugi</b></sub></a></td>
  </tr>
</table>

<!-- markdownlint-restore -->
<!-- prettier-ignore-end -->

<!-- ALL-CONTRIBUTORS-LIST:END -->

<img src="https://user-images.githubusercontent.com/15371828/158857061-8fa8d079-d33f-4ed2-88aa-56d452d238d8.svg" align="right" alt="DFINITY logo" width="270">

# Internet Computer developer portal

The IC developer portal brings together all the resources needed for developers to build on the
Internet Computer.

https://internetcomputer.org

[![Netlify Status](https://api.netlify.com/api/v1/badges/0ef9e793-aa30-446a-ae7a-a18ac304db58/deploy-status)](https://app.netlify.com/sites/icportal/deploys) [![CD](https://github.com/dfinity/portal/actions/workflows/cd.yml/badge.svg)](https://github.com/dfinity/portal/actions/workflows/cd.yml)

## Contributing

The IC developer portal uses [Docusaurus](https://docusaurus.io/docs).

## Format and style guidelines

The developer documentation must follow the provided style guide in order to be submitted:

[Developer docs style guide](style-guide.md).

### Local development

#### Requirements

- [Node v16](https://nodejs.org/en/blog/release/v16.16.0) **(v18 not recommended)**.

#### Install and run locally

While modifying documentation in the repository, you can preview the changes locally by executing the following
commands.

```bash
cd portal/
git submodule update --init
npm install
npm start
```

The developer portal frontend should appear in your browser under http://localhost:3000.

To preview a specific locale in the context of an internationalization contribution, start the docs with the following
command:

```bash
npm start -- --locale <locale>
```

### Adding a document

Docusaurus is quite easy to use and facilitates the creation of documents.
The developer portal uses Markdown as its default format
with [enhancements provided by Docusuaurus](https://docusaurus.io/docs/markdown-features).

To create a document, head to the `/docs` folder and create a Markdown file in the directory of your choice. Please assure that the document's contents follow the developer docs style guide:

[Developer docs style guide](style-guide.md).

Here is an example of a docs sub-directory:

```
developer-docs/
    developer-docs/
        home.mdx
        quickstart/
            ...
    references/
        ...
```

### Changing the roadmap

The roadmap items are stored as markdown files inside
the [roadmap](https://github.com/dfinity/portal/tree/master/roadmap) directory.

The items are grouped by domain and status.

```
roadmap
├── 1_core-protocol
│   ├── deployed
│   │   ├── network_performance.md
│   │   └── nodes_can_be_reassigned.md
│   ├── in-progress
│   │   ├── 1_btc-integration.md
│   │   ├── 2_tecdsa.md
│   │   ├── 3_https-outcalls.md
│   │   ├── 4_high-replication-subnets.md
│   │   └── 5_deterministic-time-slicing.md
│   ├── index.md
│   └── pending
│       └── eth-integration.md
├── 2_boundary_nodes
│   ├── deployed
│   │   └── icos-nodes.md
│   ├── in-progress
│   │   ├── 1_seo_and_streaming.md
│   │   ├── 2_asset_canister_caching.md
...
```

- To change the title, links, or ETA of an item, edit the metadata section at the top of each markdown document.
- To change the status of an item, eg. from `in-progress` to `deployed`, move the file to the `deployed` directory.
- Items are listed in alphabetical order of their filename, to enforce a certain ordering between items you can prefix
  the filenames with numbers.
- To mark an item as a community request, set `is_community: true` in the item's metadata.

Each domain folder contains an `index.md` file which adds metadata, like a title, a description, and cover images.

### Changing the 'how it works' content

The how it works page cards and subpages are stored as markdown files inside
the [how-it-works](https://github.com/dfinity/portal/tree/master/how-it-works) directory.

```
how-it-works
├── 1_about
│   ├── 01-overview-of-the-internet-computer.card.md
│   ├── 01-overview-of-the-internet-computer.subpage.md
│   ├── canister-lifecycle.card.md
│   ├── canister-lifecycle.subpage.md
│   └── index.md
├── 2_featured
│   ├── direct-integration-with-bitcoin.card.md
│   ├── direct-integration-with-bitcoin.subpage.md
│   ├── index.md
│   ├── sns.card.md
│   ├── threshold-ecdsa-signing.card.md
│   ├── threshold-ecdsa-signing.subpage.md
```

The directory contains markdown files ending in `.card.md` and `.subpage.md`.

The `.card.md` files will show up as part of the `/how-it-works page`, under the section they are grouped in (eg.
Featured).

The `.subpage.md` files will each generate a subpage under `/how-it-works/`, based on the files' metadata and content:

- The `slug` parameter will determine the final URL, eg. the slug `canister-lifecycle` will generate the
  page `/how-it-works/canister-lifecycle`.
- The `title`, `abstract` (optional), and `coverImage` will determine how the page looks when shared on social media

**How to embed YouTube videos:**

You can embed YouTube videos by adding the video's cover image, then turning the image into a link to the video itself,
eg:

```
[![Watch youtube video](https://i.ytimg.com/vi/YWHTNr8RZHg/maxresdefault.jpg)](https://www.youtube.com/watch?v=YWHTNr8RZHg)
```

The page will automatically put the YouTube icon on the cover image, imitating an embedded video.

You can get the cover image of a video by substituting the video ID into the following URL:

```
https://i.ytimg.com/vi/INSERT_VIDEO_ID_HERE/0.jpg
```

If the resulting image is low quality, you can try `https://i.ytimg.com/vi/INSERT_VIDEO_ID_HERE/maxresdefault.jpg`, but
it does not always exist.

### ⚠️ Contributions' musts:

- Making sure that the [`.github/CODEOWNERS`](https://github.com/dfinity/portal/blob/master/.github/CODEOWNERS) file is
  filled with new documents that you added. This way we can ensure that future Pull Requests are reviewed by the right
  people.
- When creating a document, it must be registered
  in [`/sidebars.js`](https://github.com/dfinity/portal/blob/master/sidebars.js), otherwise, it will not appear in the
  side navigation bar.
- All documents must follow the provided [Developer docs style guide](style-guide.md).

> More information about document creation on [Docusaurus's docs](https://docusaurus.io/docs/create-doc).

### Deployed Previews

Whenever a Pull Request is created on the repository, a CI job will appear and deploy a Preview on Netlify so that
reviewers can easily check the changes made the way the end users will.

To access the preview, head to the very bottom of your pull request where you will see the list of deployments.
Once the job is finished, you should see as active the "Preview Netlify" deployment.

<img width="800" alt="Screenshot 2022-03-17 at 11 45 25" src="https://user-images.githubusercontent.com/15371828/158793201-bb41f003-3d8d-4f95-9f91-8798613bc695.png">

Then simply press the "View deployment" button to in your fresh Netlify preview.

### Contribution workflow

Here is a description of how a contribution should be made to the developer portal:

1. The contributor creates a fork/branch where the changes are made.
2. A pull request (PR) is created from this branch to `master`. the preview is generated and the reviewers can directly check
   the preview website.
3. After the pull request is merged into master, CI/CD will deploy the contents to the IC. The changes made will appear
   on the webpage under the version in the dropdown named "Current".
4. Optional: When a repo maintainer tags a commit on master as $TAG, then CI/CD deploys the contents to the IC. The
   webpage will default to the new "$TAG" version, visible in the versions dropdown, and allow users to view the
   contents pinned at that tag.

### Best practices

The portal repo uses git submodules to distribute content authoring across several repositories. This can prove
challenging if you want to make updates to docs that refer to stale content in other repos. Here are some best practices
that may prove helpful when contributing to `portal`.

1. Don't commit changes to `submodules`.
   a. Check your staged changes and see if any files are nested within the `submodules` directory. If so - revert them.
   Those changes must go into the corresponding submodule repository.
2. Be wary of the order of operations when it comes to submodules.
   a. If you do intend to make changes to content under `submodules`, the changes need to be merged into the respective
   repo first.
   b. Once the change has been merged, you will also need to update `portal`'s ref to that repo. You can do that by
   running `git submodule update --remote --merge`
3. Build locally.
   a. You can save yourself some headache if you run `npm run build` locally before you push your changes. If the build
   fails locally, it will almost certainly fail in CI.
   b. Our CI requires that a set of checks pass before a PR can be merged. One of those checks whether the code can be
   deployed to a canister. You do not need to deploy a canister locally, but at the very least `npm run build` should
   pass.

## Community-created agents and CDKs

The [agents](https://internetcomputer.org/docs/current/developer-docs/build/agents)
and [CDKs](https://internetcomputer.org/docs/current/developer-docs/build/cdks) sections should not only contain docs
for DFINITY-created agents and CDKs.
We therefore invite other projects to:

- Link to their own agents or CDKs on the respective index pages (the files to be edited are
  in `docs/developer-docs/build/agents/index.md` or `docs/developer-docs/backend/choosing-language.md`).
- Add their own documentation as a folder under `Agents` or `CDKs`.

## Community-created developer tools

We invite developers to add their IC-focused developer tools to
the [developer tools page](https://internetcomputer.org/tooling) by appending an entry to the `communityToolingItems`
array in `src/components/Common/toolingItems.ts`.

Please make sure to add appropriate tags to make the tool easy to discover for other devs. Avoid introducing new tags if
possible.

## Community-created sample projects

You can submit your sample project to be displayed on the [samples page](https://internetcomputer.org/samples).

Add your submission to [the community projects file](/community/communityProjects.ts) and open a pull request. You can
use an editor with TypeScript support to make sure your submission
follows [the schema](/src/components/Common/sampleItems.ts).

## Showcase submission guidelines

_Disclaimer: You should have a working canister for your product to be showcased. Our team will review the PR and get
back to you for any further questions. In the meantime, please contact devcomms@dfinity.org if you have any questions._

Add your project to the end of [`showcase.json`](/showcase.json). Refer to the object schema below for the required
fields.

Make up a unique project id. For example, if your project is called `Awesome ICP Project!`, your project id could
be `awesome-icp-project`.

Your logo/video/screenshots files should be prefixed with your project id, and placed in the `/static/img/showcase`
folder. For example, if your project id is `awesome-icp-project`, your logo file should be
named `awesome-icp-project_logo.webp` and placed in the `/static/img/showcase` folder.

The [Ecosystem Helper](https://mvw4g-yiaaa-aaaam-abnva-cai.icp0.io/) is an on-chain tool that helps you submit your project to internetcomputer.org. It helps with image conversion, resizing, previewing the project cards used throughout the website, and it produces a valid JSON document you can use. Fill out the form and download the asset bundle in a zip file.

### Asset guidelines

| Asset       |          | Requirements | Format       | Notes                                                                             |
|-------------|----------|--------------|--------------|-----------------------------------------------------------------------------------|
| logo        | required | 112x112px    | webp/svg/png | Currently displayed 56x56px                                                       |
| screenshots | optional | 1024x576px   | webp/jpg     | The schema supports multiple files, but only the first one will be displayed      |
| video       | optional | max 10MB     | webm/mp4     | If there is a video file specified, it will be displayed instead of a screenshot. |

### Tags

The list of tags is not final, and will be updated as the project evolves. For now, the following tags are available:

- `Wallet`
- `Bitcoin`
- `NFT`
- `SocialFi`
- `DeFi`
- `Games`
- `DAO`
- `Metaverse`
- `Tools / Infrastructure`

### Object schema

```
  {
    id: string,
    name: string,
    oneLiner: string, // short description of the project
    website: string, // URL starting with `https://`

    tags: ('Wallet' | 'Bitcoin' | 'NFT' | 'SocialFi' | 'DeFi' | 'Games' | 'DAO' | 'Metaverse' | 'Tools / Infrastructure')[],
    description: string, // description of the project
    stats: string, // eg. "10,000 users"
    logo: string, // url to logo file, eg. /img/showcase/awesome-icp-project_logo.webp
    
    usesInternetIdentity: boolean,
    authOrigins?: string[]; // optional additional (URL) origins that can be utilized for signing in to your dapp

    github?: string, // full URL to github repo, if available
    youtube?: string, // full URL to a YouTube video or channel, if available
    twitter?: string, // full URL to a twitter account, if available

    screenshots?: string[], // optional array of urls to screenshot files

    video?: string, // optional url to video file, eg. /img/showcase/awesome-icp-project_video.webm
    videoContentType?: 'video/webm' | 'video/mp4', // to feed into the type attribute of the video/source element

    submittableId?: string, // optional id of the submittable form
  },
```
