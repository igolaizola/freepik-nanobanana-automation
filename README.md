# Freepik NanoBanana Automation 🍌

Automate AI image generation on **Freepik** with **NanoBanana**, Google's latest AI image model.
Scale workflows via no-code or API, run batch jobs, chain outputs, leverage unlimited generations, and ship an album HTML for instant previewing and bulk downloads.

> [!TIP]
>
> You can use Freepik NanoBanana Automation directly from APIFY platform [apify.com/igolaizola/freepik-nanobanana](https://apify.com/igolaizola/freepik-nanobanana)

## 🎨 What can this actor do?

- 🤖 Run **multiple generations** in one go (with different prompts, images, and aspect ratios)
- 🔁 **Chain** steps: use outputs from earlier generations as inputs to later ones
- 🖼️ Mix inputs from **URLs**, **uploaded files**, and **Google Drive links**
- 🚦 Control **concurrency**, **random waits**, and **timeouts** to be gentle on the platform
- 📁 Get an **album.html** to preview and bulk-download your images

## 🔑 Prerequisites

1. A Freepik account with access to AI image generation (NanoBanana)
2. Your **freepik.com cookie** for authentication
3. The [Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm) browser extension

**How to get your cookie**

1. Install [Cookie-Editor](https://chromewebstore.google.com/detail/cookie-editor/hlkenndednhfkekhgcdicdfddnkalmdm)
2. Log in to **[https://freepik.com](https://freepik.com)**
3. Click the extension icon → grant permissions if asked
4. Click **Export** → **Export as Header String**
5. Copy the exported string and paste it into the `cookie` field

## 🚀 Quick start

1. Paste your cookie in `cookie`
2. Add one or more **generation steps** under `generations` (see examples below)
3. (Optional) Upload images in `upload1` … `upload10` and reference them as `upload:1`, `upload:2`, …
4. Keep defaults or tune `concurrency`, `minWait`, `maxWait`, and `jobTimeout`
5. (Optional) Keep Apify residential proxy enabled for reliability
6. Run the actor and open the **Album URL** from results to browse your images

## ⚙️ Input parameters

| Parameter              | Type             | Description                                                                                                                           | Default                                                      |
| ---------------------- | ---------------- | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `cookie`               | string           | Your **freepik.com** authentication cookie. **Required.**                                                                             | —                                                            |
| `generations`          | array of objects | The **recipe** for your run. Each item supports: `prompt`, `images`, `aspectRatio`, `runs`, `name`. See examples below. **Required.** | —                                                            |
| `concurrency`          | integer          | Parallel jobs running at once.                                                                                                        | `1`                                                          |
| `minWait`              | integer (sec)    | Minimum random wait **between tasks**.                                                                                                | `5` (if not set)                                             |
| `maxWait`              | integer (sec)    | Maximum random wait **between tasks**.                                                                                                | `10` (if not set)                                            |
| `jobTimeout`           | integer (sec)    | Max time to wait for each generation to finish.                                                                                       | `300`                                                        |
| `upload1` … `upload10` | file             | Optional uploads you can reference as `upload:1`, … `upload:10`.                                                                      | —                                                            |
| `proxyConfiguration`   | object           | Apify proxy settings. Prefilled to **RESIDENTIAL**.                                                                                   | `{ useApifyProxy: true, apifyProxyGroups: ["RESIDENTIAL"] }` |

**Notes**

- `aspectRatio` defaults to **`"1:1"`** if you omit it.
- `runs` defaults to **`1`** if you omit it.
- If a generation has **neither** `prompt` **nor** `images`, it's skipped.
- `images` supports:

  - Direct **URL** (PNG/JPG/…)
  - **`upload:n`** (e.g., `upload:3`)
  - **`output:name`** (use outputs of a previous named step)
  - **Google Drive** links (`https://drive.google.com/...`) are accepted; they'll be converted automatically.

## 🧠 The `generations` field — how to write it (with recipes)

Each item in `generations` is a **step**. You can run independent steps or chain them together.
Fields per step:

- `prompt` _(string)_ – Text instruction.
- `images` _(string array)_ – Inputs (URLs, `upload:n`, or `output:name`).
- `aspectRatio` _(string)_ – e.g., `"1:1"`, `"16:9"`, `"9:16"`. Default `"1:1"`.
- `runs` _(integer)_ – How many images to create for this step.
- `name` _(string)_ – If set, later steps can reference **this step's outputs** via `output:name`.

> When a step references `output:some-name` that produced **multiple images**, one of them is picked at random for each use, which is great for variation.

### 1) Basic single prompt (no images)

```json
{
  "generations": [
    {
      "prompt": "A cozy minimalist living room with warm evening light",
      "aspectRatio": "16:9",
      "runs": 2
    }
  ]
}
```

### 2) Use an image URL as input

```json
{
  "generations": [
    {
      "prompt": "Improve lighting, keep composition the same",
      "images": ["https://picsum.photos/id/17/800/600"],
      "runs": 1
    }
  ]
}
```

### 3) Use uploaded files (`upload:n`)

Upload files into `upload1`, `upload2`, … and reference them:

```json
{
  "generations": [
    {
      "prompt": "Convert @img1 into a watercolor illustration",
      "images": ["upload:1"],
      "aspectRatio": "1:1",
      "runs": 3
    }
  ]
}
```

### 4) Combine multiple images

```json
{
  "generations": [
    {
      "prompt": "Combine @img1 and @img2 into a single cinematic poster",
      "images": [
        "https://picsum.photos/id/17/800/600",
        "https://picsum.photos/id/22/800/600"
      ],
      "aspectRatio": "16:9",
      "runs": 1,
      "name": "poster-v1"
    }
  ]
}
```

### 5) Chain steps using `output:name`

Create a combo, then restyle it using the previous result:

```json
{
  "generations": [
    {
      "prompt": "Merge @img1 and @img2 into a single clean composition",
      "images": ["upload:1", "upload:2"],
      "aspectRatio": "16:9",
      "runs": 2,
      "name": "merge"
    },
    {
      "prompt": "Make @img1 look 10 years older in style",
      "images": ["output:merge"],
      "aspectRatio": "1:1",
      "runs": 2,
      "name": "older-style"
    },
    {
      "prompt": "Change the style of @img1 using @img2 as reference",
      "images": ["output:older-style", "upload:3"],
      "aspectRatio": "9:16",
      "runs": 2
    }
  ]
}
```

### 6) Variation pipeline (branching outputs)

If a named step creates multiple images, later steps can branch from it. Each reference to `output:NAME` will pick **one** of those images at random:

```json
{
  "generations": [
    {
      "prompt": "Generate a mascot character, friendly and simple",
      "runs": 4,
      "name": "mascot"
    },
    {
      "prompt": "Vectorize @img1 and make it logo-ready",
      "images": ["output:mascot"],
      "runs": 3,
      "name": "logo-ready"
    },
    {
      "prompt": "Create a sticker sheet from @img1",
      "images": ["output:mascot"],
      "aspectRatio": "1:1",
      "runs": 2
    }
  ]
}
```

### 7) Use Google Drive images

You can paste Google Drive links directly; they'll be converted to direct downloads:

```json
{
  "generations": [
    {
      "prompt": "Retouch @img1 and remove the background",
      "images": ["https://drive.google.com/file/d/FILE_ID/view?usp=sharing"],
      "runs": 1
    }
  ]
}
```

### 8) Full example (mixing everything)

```json
{
  "generations": [
    {
      "prompt": "Combine these two into a creative collage",
      "images": [
        "https://picsum.photos/id/17/800/600",
        "https://picsum.photos/id/22/800/600"
      ],
      "aspectRatio": "16:9",
      "runs": 1,
      "name": "combination-01"
    },
    {
      "prompt": "Make the style of @img1 10 years older",
      "images": ["output:combination-01"],
      "aspectRatio": "1:1",
      "runs": 2,
      "name": "older-style"
    },
    {
      "prompt": "Change the style of @img1 using @img2 as reference",
      "images": ["output:older-style", "upload:1"],
      "aspectRatio": "9:16",
      "runs": 2
    }
  ]
}
```

**Gotchas to avoid**

- If you reference `output:some-name` that **doesn't exist yet**, the step is skipped. Always place your **producer** steps before **consumer** steps.
- If you set `concurrency > 1` and have **chained steps**, the actor might need to wait for earlier steps to finish before starting later ones. This can reduce parallelism.

## 📤 Output format

Each generated image is saved as an item:

```json
[
  {
    "album_url": "https://api.apify.com/v2/key-value-stores/xxx/records/album.html#2_12345",
    "prompt": "Make the style of @img1 10 years older",
    "index": 2,
    "job_id": 12345,
    "url": "https://some.cdn.freepik.com/path/to/image.png"
  }
]
```

## 📸 Accessing Generated Images (Album Gallery)

The actor saves an **`album.html`** file to the KeyValue store for easy browsing and downloading of generated images. You can access it by:

- Clicking any "Album URL" field in the results table
- Going to Run > Storage > KeyValueStore and finding the "album.html" file

The album interface provides several features:

- Filter images by prompt text or other metadata
- Select multiple images for batch operations
- Open images in new tabs for detailed viewing
- Download selected images as a ZIP file

Here's an example album showcasing some generated images so you can see how your results will be displayed: [Browse Example Gallery](https://igolaizola.github.io/freepik-nanobanana-automation/)

## ⚠️ Usage guidelines & best practices

**Account safety**

- Automation of Freepik accounts can result in termination, so use responsibly.
- Avoid running huge, continuous workloads.
- Prefer **residential proxies** near your region.

**Reliability tips**

- Start with `concurrency = 1`, then scale gradually.
- Tune `minWait`/`maxWait` to introduce natural pacing.
- Increase `jobTimeout` for long queues.
- Keep your cookie fresh (re-export it if you get auth errors).

**Prompting**

- Be explicit: style, mood, medium, constraints.
- For multi-image prompts, refer to them in text as `@img1`, `@img2`, … (the order matches your `images` array).

## 💳 How much will it cost?

Apify gives you **\$5 in free usage credits every month** on the [Free plan](https://apify.com/pricing), which is enough to try this actor.
For regular use, we recommend at least the **Personal** plan.

## ❓FAQ

**Do I need to set `aspectRatio`?**
No. It defaults to `"1:1"` if omitted.

**Can I reuse outputs across many later steps?**
Yes—give a step a `name` and reference it later via `output:that-name`.

**Can I feed Google Drive links?**
Yes—paste the link; it's converted to a direct download internally.
