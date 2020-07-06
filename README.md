# Barbarian

## Build Setup

```bash
# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm run start

# generate static project
$ npm run generate
```
## prerequisites 

Dependencies

```
npm install --save contentful
npm install @contentful/rich-text-html-renderer
npm install algoliasearch
npm install remove-markdown
```


## Contentful + NuxtJs

setting up the content model using contentful dashboard

create space home
create content model type and add fields
Add entry to content

* Login to contentful with command

```
$ contentful login
```

* Install contentful plugin

create a new file in the plugin directory called contentful.js to create an SDK client with pre-defined environment variables

```
// ./plugins/contentful.js

const contentful = require('contentful')
// use default environment config for convenience
// these will be set via `env` property in nuxt.config.js
const config = {
  space: process.env.CTF_SPACE_ID,
  accessToken: process.env.CTF_CDA_ACCESS_TOKEN
}

// export `createClient` to use it in page components
module.exports = {
  createClient () {
    return contentful.createClient(config)
  }
}
```

* Define the environment variables

create a file on root directory called .contentful.json and use the contentful configuration variables from plugin

```
{
  "CTF_PERSON_ID": "<YOUR_CONTENT_ID>",
  "CTF_BLOG_POST_TYPE_ID": "<YOUR_CONTENT_TYPE>",
  "CTF_SPACE_ID": "<YOUR_SPACE_ID>",
  "CTF_CDA_ACCESS_TOKEN": "<YOUR_ACCESS_TOKEN>"
}
```
* In the nuxt.config.js you can then require the config file and make it available in the plugins file via the env property.

```
const config = require('./.contentful.json')
export default {
...
env: {
    CTF_SPACE_ID: config.CTF_SPACE_ID,
    CTF_CDA_ACCESS_TOKEN: config.CTF_CDA_ACCESS_TOKEN,
    CTF_PERSON_ID: config.CTF_PERSON_ID,
    CTF_BLOG_POST_TYPE_ID: config.CTF_BLOG_POST_TYPE_ID
  }
...
```

* Fetch data and render page

```
<template>
  <div>
    <!-- render data of the person -->
    <h1>{{ person.fields.name }}</h1>
    <!-- render blog posts -->
    <ul>
      <li v-for="post in posts">
        {{ post.fields.title }}
      </li>
    </ul>
  </div>
</template>

<script>
  import {createClient} from '~/plugins/contentful.js'

  const client = createClient()

  export default {
    // `env` is available in the context object
    asyncData ({env}) {
      return Promise.all([
        // fetch the owner of the blog
        client.getEntries({
          'sys.id': env.CTF_PERSON_ID
        }),
        // fetch all blog posts sorted by creation date
        client.getEntries({
          'content_type': env.CTF_BLOG_POST_TYPE_ID,
          order: '-sys.createdAt'
        })
      ]).then(([entries, posts]) => {
        // return data that should be available
        // in the template
        return {
          person: entries.items[0],
          posts: posts.items
        }
      }).catch(console.error)
    }
  }
</script>
```


...

## Contentful + Algolia

...
