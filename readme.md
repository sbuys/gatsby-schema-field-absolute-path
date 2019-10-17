# gatsby-schema-field-absolute-path

This plugin resolves absolute path (i.e `content/assets`) to the correct File node in your Gatsby graphql schema.

For example, if you have a content structure like this

```
root
  |--content
  |     |--posts
  |     |   `--hello-world.md
  |     `--assets
  |         `--cat.jpg
  |--src
  |--gatsby-config.js
 ...
```

And in `hello-world.md`, you have this frontmatter

```md
---
slug: hello-word
featuredImage: assets/cat.jpg
---

What a nice day!
```

This plugin will resolve `assets/cat.jpg` to a File node instead of a string, so you can query the image as expected:

```gql
query PostContent {
  frontmatter {
    featuredImage {
      childImageSharp {
        fluid(maxWidth: 1280) {
          ...gatsbyImageSharpFluid
        }
      }
    }
  }
}
```

Of course this is not done by magic, you'd have to add the generated field extension to your field via `createTypes` in `gatsby-node.js`.

```js
// gatsby-node.js

exports.createSchemaCustomization = ({ actions }) => {
  const { createTypes } = actions

  // @fileByDataPath is a field extension generated by this plugin
  const typeDefs = `
    type Frontmatter @infer {
      featureImage: File @fileByDataPath
    }

    type MarkdownRemark implements Node @infer {
      frontmatter: Frontmatter
    }
  `

  createTypes(typeDefs)
}
```

### Usage

#### Install
```sh
yarn add gatsby-schema-field-absolute-path

# or
npm i gatsby-schema-field-absolute-path
```

#### Add to Gatsby config

```js
// gatsby-config.js

module.exports = {
  plugins: [
    {
      resolve: 'gatsby-schema-field-absolute-path',
      options: {
        // 1. single directory
        dirs: 'content/assets'

        // 2. array of directories
        dirs: ['content/assets', 'src/processed/images']

        // or 3. object with named field extension
        dirs: {
          'content/assets': 'fileByAssetPath',
          'src/processed/images': 'fileByImagePath',
        }
      }
    }
  ]
}

```


#### Specify target fields

```js
// gatsby-node.js

exports.createSchemaCustomization = ({ actions }) => {
  const { createTypes } = actions

  const typeDefs = `
    type Frontmatter @infer {
      featureImage: File @fileByAssetPath
    }

    type MarkdownRemark implements Node @infer {
      frontmatter: Frontmatter
    }
  `

  createTypes(typeDefs)
}
```

### More about options

Currently this plugin only takes 1 option (it's a really simple one after all), `dirs`, which accepts a string, an array of string, or an object.

```ts
type Path = string
type FieldExtensionName = string

interface Options {
  dirs: Path | Path[] | Record<Path, FieldExtensionName>
}
```

- The path must be relative to your root directory. For example, `src` / `/src` / `./src` will all resolve to `<project root>/src`.
- The field extension name is generated automatically by the directory:

| directory | generated field extension name |
|---|---|
| `content/posts` | `pathByPostsPath` |
| `content/assets` | `pathByAssetsPath` |

If you're unsure what the generated name maybe, this plugin will tell you in development:
```
info gatsby-plugin-file-absolute-path: Field extension created! Use @fileByDataPath for 'src/data'
```

- If you pass an object to `dir`, you can specify the field extension name instead of generated one:

```js
// option
dirs: { 'content/assets': 'myFieldExtensionName' }

// usage
`
  type Frontmatter @infer {
    featuredImage: File @myFieldExtensionName
  }
`
```

---