# osiera-data

Starter dataset for the [Osiera](https://github.com/Lingyv07/osiera) WordPress theme — the fixtures the
theme's **Osiera → Starter data** screen fetches at import time. This repository holds content only:
JSON fixtures and demo media, no PHP.

The theme points here by default (`init-config.json`):

```json
{
  "data_init_url": "https://raw.githubusercontent.com/Lingyv07/osiera-data/refs/heads/main/cnc-data"
}
```

## How the theme consumes it

- Import is triggered manually from **Osiera → Starter data**. Nothing is published on activation.
- In remote mode the theme downloads each file on demand: JSON via `data/*.json`, media via
  `assets/*`. Expect the first import to take a while and to need an internet connection.
- Each site imports once. The state is recorded in `tpi_data_init_state` / `tpi_starter_content`, and
  the screen offers a re-import afterwards.
- Import never overwrites existing content: a row with the same slug in the same language is skipped.
  Terms, menu items and attachments are matched by name/slug/filename, so re-running does not
  duplicate them.

## Directory layout

```
cnc-data/
├── assets/
│   ├── images/     46 bitmaps, imported into the media library
│   ├── svg/        19 content icons and language flags
│   └── videos/     1 mp4 (company-shop-floor.mp4)
└── data/
    ├── languages.json
    ├── taxonomies.json
    ├── menus.json
    ├── lang-content.json
    ├── page-info.json
    ├── products.json
    ├── applications.json
    ├── faqs.json
    ├── posts.json
    └── assets-alt.json
```

`cnc-data` is the dataset root. Keeping the content under a named folder means a second site or
industry can live beside it (`pipe-data/`, `furniture-data/`) without restructuring the repository.

## Data files

| File | Contents |
|---|---|
| `languages.json` | Default language code and the list of languages: `code`, `name`, `locale`, `flag`. |
| `taxonomies.json` | Terms for `product_cat`, `category`, `application_cat` and `faq_cat`. |
| `menus.json` | Primary menu: `name`, `slug`, `location`, `extra_pages`, `items`. |
| `lang-content.json` | Per-language UI copy: header, footer, home, product, news, applications, faq, about-us, contact-us, certificates, privacy-policy, search. |
| `page-info.json` | Site-wide contact/company settings under `shared`, plus per-language overrides (address, banner and section images, copy). Missing keys fall back to `shared`. |
| `products.json` | 5 products per language. |
| `applications.json` | 3 applications per language. |
| `faqs.json` | 6 FAQ entries per language. |
| `posts.json` | 4 news posts per language. |
| `assets-alt.json` | Alt text keyed by asset path, so imported images and the video get meaningful `alt` instead of a filename. |

Everything except `languages.json` (and `menus.json`, which is not language-keyed) uses the language
code as the top-level key:

```json
{
  "en": [ { "slug": "cnc-machining", "title": "…", "order": 1, "terms": ["…"] } ],
  "ja": [ { "slug": "cnc-machining", "title": "…", "order": 1, "terms": ["…"] } ]
}
```

A product row looks like this:

```json
{
  "slug": "aluminum-housing",
  "title": "Aluminum Housing",
  "order": 3,
  "terms": ["CNC Machining"],
  "excerpt": "…",
  "image": "assets/images/product-aluminum-housing.jpg",
  "gallery": ["assets/images/product-aluminum-housing-2.jpg"],
  "summary": "…",
  "params": "…",
  "warning": "…",
  "content": "<p>…</p>",
  "main_title": "…"
}
```

`applications.json`, `faqs.json` and `posts.json` use the same row shape minus the fields that do not
apply — FAQ rows carry `answer` instead of `gallery`.

## Import order

The importer walks the dataset in a fixed order, because later steps reference earlier ones:

1. languages
2. pages (title from the menu label, falling back to `lang-content`)
3. taxonomy terms
4. primary menu and its items
5. language content (icon paths resolved to upload URLs)
6. page info (`shared`, then per-language overrides)
7. products → posts → applications → FAQs
8. translation links between the two language versions of the same slug

## Conventions

- **Asset paths are relative to the dataset root** — `assets/images/x.jpg`, never an absolute path or
  a full URL. The importer rewrites them to media-library URLs, and the database keeps only the
  upload-relative path, so moving to another domain needs no data change.
- **`slug` is the join key across languages.** The same slug under `en` and `ja` links the two rows as
  translations; different slugs mean two unrelated entries.
- **`terms` must exist in `taxonomies.json`** — the string has to match the term name exactly.
- **`order`** sorts entries inside an archive; equal values fall back to insertion order.
- **`content` is HTML.** Image paths inside `src`, `href`, `poster`, `data-src` and `data-original`
  are rewritten on import; any other attribute is left alone.
- **Media is matched by filename.** Replacing a photo in place is fine; renaming it creates a second
  attachment on sites that already imported.
- **`assets-alt.json`** accepts three spellings of a key: `assets/images/x.jpg`, `images/x.jpg` or
  just `x.jpg`.

## Adding a language

1. Add the language to `data/languages.json` (`code`, `name`, `locale`, `flag`).
2. Add a key with that code to `products.json`, `applications.json`, `faqs.json`, `posts.json` and
   `lang-content.json`.
3. Add the language key to `page-info.json` for the overrides you want to translate.
4. Reuse the same slugs as the existing languages so translation links resolve.
5. Add a flag SVG under `assets/svg/`.
6. Re-import. Existing rows are skipped, new-language rows are created.

## Using your own dataset

Point the theme at a fork instead of editing this repository:

```json
{
  "data_init_url": "https://raw.githubusercontent.com/<you>/<repo>/refs/heads/main/<dataset>"
}
```

The value can also be a theme-relative path (`data-init`), an absolute server path, or any HTTP(S)
directory. The same value can be set in the admin, where it overrides `init-config.json`.

## Media licensing

The photographs and the video are **demo placeholders** for a fictional CNC machining company. They
exist to make an empty install look like a real site; they are not part of the theme's licensed
assets and are not cleared for redistribution as stock imagery. Replace them with your own before
going live, and note the source of anything you keep — the theme's `NOTICE.md` has the tracking
table.

The JSON fixtures are distributed under the same licence as the Osiera theme, GPL-3.0-or-later. If
you prefer a permissive licence for the data alone, change this paragraph and add a `LICENSE` file.

## Notes

- Do not put customer content in this repository. Fork it per project instead.
- Line endings are normalised to LF by `.gitattributes`; do not commit CRLF copies of the JSON files.
- Importing pulls the media it needs, so the dataset should stay small enough for a raw HTTP fetch —
  no single file should approach GitHub's 100 MB limit.

---

## 中文说明

这是 Osiera 主题的**起始数据集**，主题「Osiera → 初始数据」页在导入时从这里拉取，只放内容
（JSON + 演示媒体），不含任何 PHP。

**导入方式**：后台手动触发；远端模式按需下载 `data/*.json` 与 `assets/*`，首次导入需要联网且耗时较长。
每个站点导入一次（状态记在 `tpi_data_init_state`），之后可"重新导入"。导入**不覆盖已有内容**——同
slug 同语言已存在就跳过；分类、菜单项、附件按名称/slug/文件名复用，重复执行不会翻倍。

**目录约定**：`cnc-data/` 是数据集根目录，换行业或换站点可以并列放 `pipe-data/`、`furniture-data/`，
不用改仓库结构。

| 文件 | 内容 |
|---|---|
| `languages.json` | 默认语言 + 语言列表（code / name / locale / flag） |
| `taxonomies.json` | `product_cat`、`category`、`application_cat`、`faq_cat` 的分类项 |
| `menus.json` | 主菜单：name / slug / location / extra_pages / items |
| `lang-content.json` | 各语言的界面文案（页头、页脚、首页、产品、资讯、应用、FAQ、关于、联系、证书、隐私、搜索） |
| `page-info.json` | `shared` 放全站联系方式与公司信息，各语言键放覆盖项（地址、横幅与栏目图、文案） |
| `products.json` / `applications.json` / `faqs.json` / `posts.json` | 每语言 5 / 3 / 6 / 4 条 |
| `assets-alt.json` | 按资源路径写 alt 文案，导入后图片与视频的可读描述不退化文件名 |

**关键约定**

- 资源路径一律相对数据集根目录写（`assets/images/x.jpg`），导入时替换成媒体库地址；数据库只存上传目录
  相对路径，换域名不用改数据。
- `slug` 是跨语言的关联键：`en` 与 `ja` 下同名 slug 视为同一篇的两种语言，slug 不同就是两条无关内容。
- `terms` 里的名称必须与 `taxonomies.json` 完全一致。
- `order` 决定列表页排序，相同值按写入顺序。
- `content` 是 HTML，只有 `src` / `href` / `poster` / `data-src` / `data-original` 里的资源路径会被替换。
- 媒体按**文件名**复用：原地替换图片没问题，改名会在已导入的站点上多出一份附件。

**新增语言**：在 `languages.json` 加一项 → 在四个内容 JSON 与 `lang-content.json` 加同名语言键 →
按需要给 `page-info.json` 加覆盖 → slug 与现有语言保持一致 → 在 `assets/svg/` 放国旗 →
重新导入（已有内容会跳过，只补新语言）。

**媒体授权**：这些照片与视频是给虚构 CNC 公司用的**演示占位素材**，不属于主题授权资源，也未按图库
素材取得转授权；正式上线前请替换成自己的素材，保留的要在主题 `NOTICE.md` 的表格里登记来源。
JSON 数据本身与主题同许可（GPL-3.0-or-later），想单独用 MIT/CC0 就改上面那段并补 `LICENSE`。

**注意**：不要往这个仓库塞客户内容，按项目 fork；`.gitattributes` 已把换行统一为 LF，别提交 CRLF 副本。
