---
author: Denney Yang
pubDatetime: 2022-09-23T15:22:00Z
title: Adding new posts in AstroPaper theme
featured: false
draft: false
tags:
- docs
description: Some rules & recommendations for creating or adding new posts using AstroPaperr
---

Here are some rules/recommendations, tips & ticks for creating new posts in AstroPaper blog theme.

<figure>
  <img
    src="https://images.pexels.com/photos/159618/still-life-school-retro-ink-159618.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1"
    alt="Free Classic wooden desk with writing materials, vintage clock, and a leather bag. Stock Photo"
  />
    <figcaption class="text-center">
    Photo by <a href="https://www.pexels.com/photo/brown-wooden-desk-159618/">Pixabay</a>
  </figcaption>
</figure>

## Table of contents

## Creating a Blog Post

To write a new blog post, create a markdown file inside the `src/data/blog/` directory.

> Prior to AstroPaper v5.1.0, all blog posts had to be in `src/data/blog/`, meaning you couldn't organize them into subdirectories.

Starting from AstroPaper v5.1.0, you can now organize blog posts into subdirectories, making it easier to manage your content.

For example, if you want to group posts under `2025`, you can place them in `src/data/blog/2025/`. This also affects the post URL, so `src/data/blog/2025/example-post.md` will be available at `/posts/2025/example-post`.

If you don’t want subdirectories to affect the post URL, just prefix the folder name with an underscore `_`.

```bash
# Example: blog post structure and URLs
src/data/blog/very-first-post.md          -> mysite.com/posts/very-first-post
src/data/blog/2025/example-post.md        -> mysite.com/posts/2025/example-post
src/data/blog/_2026/another-post.md       -> mysite.com/posts/another-post
src/data/blog/docs/_legacy/how-to.md      -> mysite.com/posts/docs/how-to
src/data/blog/Example Dir/Dummy Post.md   -> mysite.com/posts/example-dir/dummy-post
```

> 💡 Tip: You can override a blog post’s slug in the frontmatter as well. See the next section for more details.

If the subdirectory URL doesn’t appear in the build output, remove node_modules, reinstall packages, and then rebuild.

## Frontmatter

Frontmatter is the main place to store some important information about the blog post (article). Frontmatter lies at the top of the article and is written in YAML format. Read more about frontmatter and its usage in [astro documentation](https://docs.astro.build/en/guides/markdown-content/).

Here is the list of frontmatter property for each post.

| Property           | Description                                                                                                                           | Remark                                         |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| **_title_**        | Title of the post. (h1)                                                                                                               | required<sup>\*</sup>                          |
| **_description_**  | Description of the post. Used in post excerpt and site description of the post.                                                       | required<sup>\*</sup>                          |
| **_pubDatetime_**  | Published datetime in ISO 8601 format.                                                                                                | required<sup>\*</sup>                          |
| **_modDatetime_**  | Modified datetime in ISO 8601 format. (only add this property when a blog post is modified)                                           | optional                                       |
| **_author_**       | Author of the post.                                                                                                                   | default = SITE.author                          |
| **_slug_**         | Slug for the post. This field is optional.                                                                                            | default = slugified file name                  |
| **_featured_**     | Whether or not display this post in featured section of home page                                                                     | default = false                                |
| **_draft_**        | Mark this post 'unpublished'.                                                                                                         | default = false                                |
| **_tags_**         | Related keywords for this post. Written in array yaml format.                                                                         | default = others                               |
| **_ogImage_**      | OG image of the post. Useful for social media sharing and SEO. This can be a remote URL or an image path relative to current folder.  | default = `SITE.ogImage` or generated OG image |
| **_canonicalURL_** | Canonical URL (absolute), in case the article already exists on other source.                                                         | default = `Astro.site` + `Astro.url.pathname`  |
| **_hideEditPost_** | Hide editPost button under blog title. This applies only to the current blog post.                                                    | default = false                                |
| **_timezone_**     | Specify a timezone in IANA format for the current blog post. This will override the `SITE.timezone` config for the current blog post. | default = `SITE.timezone`                      |

> Tip! You can get ISO 8601 datetime by running `new Date().toISOString()` in the console. Make sure you remove quotes though.

Only `title`, `description` and `pubDatetime` fields in frontmatter must be specified.

Title and description (excerpt) are important for search engine optimization (SEO) and thus AstroPaper encourages to include these in blog posts.

`slug` is the unique identifier of the url. Thus, `slug` must be unique and different from other posts. The whitespace of `slug` should to be separated with `-` or `_` but `-` is recommended. Slug is automatically generated using the blog post file name. However, you can define your `slug` as a frontmatter in your blog post.

For example, if the blog file name is `adding-new-post.md` and you don't specify the slug in your frontmatter, Astro will automatically create a slug for the blog post using the file name. Thus, the slug will be `adding-new-post`. But if you specify the `slug` in the frontmatter, this will override the default slug. You can read more about this in [Astro Docs](https://docs.astro.build/en/guides/content-collections/#defining-custom-slugs).

If you omit `tags` in a blog post (in other words, if no tag is specified), the default tag `others` will be used as a tag for that post. You can set the default tag in the `content.config.ts` file.

```ts file="src/content.config.ts"
export const blogSchema = z.object({
  // ...
  draft: z.boolean().optional(),
  // [!code highlight:1]
  tags: z.array(z.string()).default(["others"]), // replace "others" with whatever you want
  // ...
});
```

### Sample Frontmatter

Here is the sample frontmatter for a post.

```yaml file="src/data/blog/sample-post.md"
