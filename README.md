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

1. setting up the content model using contentful dashboard
* create space  
* create content model type and add fields
* Add entry to content

2. Login to contentful with command

```
$ contentful login
```

3. Install contentful plugin

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

4. Define the environment variables

create a file on root directory called .contentful.json and use the contentful configuration variables from plugin

```
{
  "CTF_PERSON_ID": "<YOUR_CONTENT_ID>",
  "CTF_BLOG_POST_TYPE_ID": "<YOUR_CONTENT_TYPE>",
  "CTF_SPACE_ID": "<YOUR_SPACE_ID>",
  "CTF_CDA_ACCESS_TOKEN": "<YOUR_ACCESS_TOKEN>"
}
```
5. In the nuxt.config.js you can then require the config file and make it available in the plugins file via the env property.

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

6. Fetch data and render page

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

## Contentful + Algolia

Process to add records to Algolia from contentful using Nuxt.js framework ( it can be add manually on Algolia dashboard)

1. install dependencies to work with Algolia.
```
npm install algoliasearch
```
2. Initialized Algolia index

```
const algoliaIndex = algoliaClient.initIndex(<YOUR_INDEX>);
```


3. we need to get the entries with asyncData, then create a json within contentful's fields, to export Algolia API.

```
...
        client.getEntries({
          'content_type': env.CTF_BLOG_POST_TYPE_ID,
          order: '-sys.createdAt'
        }) 
      ]).then(([entries, posts]) => {

      //create the json data to be sent
        const profile = entries.items.map(post => ({
          name: post.fields.name,
          exp: removeMd(documentToHtmlString(post.fields.ex)),
          honorsAwards: removeMd(documentToHtmlString(post.fields.honorsAwards)),
          newsEvents: removeMd(documentToHtmlString(post.fields.newsEvents)),
          insightInnovations: removeMd(documentToHtmlString(post.fields.insightInnovations)),
          objectID: post.sys.id
        })); 
...
```

we use two dependencies more, specially for text format

rich-text-html-renderer
Render text format from contentful's rich-text 

remove-markdown
Render text without special format contentful mark (kind of json format)

4. Send data to Algolia 

```
const indexedContent = algoliaIndex.saveObjects(profile,true);
```
The records loaded will be added to Algolia index


## Performance

Search from algolia index and show object information.

1. Install instantsearch for nuxt framework

```
npm install vue-instantsearch
```

2. Instantsearch use modules and it needs to be executed, add the following configuration to nuxt.config.js

```
module.exports = {
  build: {
    transpile: ['vue-instantsearch', 'instantsearch.js/es'],
  },
};
```
3. we specify what widgets used from vue-instantsearch library.

```
<template>
  <ais-instant-search :search-client="searchClient" index-name="instant_search">
    <ais-search-box />
    <ais-stats />
    <ais-refinement-list attribute="brand" />
    <ais-hits>
      <template slot="item" slot-scope="{ item }">
        <p>
          <ais-highlight attribute="name" :hit="item" />
        </p>
        <p>
          <ais-highlight attribute="brand" :hit="item" />
        </p>
      </template>
    </ais-hits>
    <ais-pagination />
  </ais-instant-search>
</template>
```

```
import {
  AisInstantSearch,
  AisRefinementList,
  AisHits,
  AisHighlight,
  AisSearchBox,
  AisStats,
  AisPagination,
  createServerRootMixin,
} from 'vue-instantsearch';
```

4. Called data from algolia

```
export default {
  ...
data() {
    return {
      searchClient,
    };
  },
...
};
```

5. Configure Search attributes on Algolia Dashboard

* Add a searchable attribute
Algolia - index - <INDEX_NAME> - configuration - Searchable attributes
* Add custom ranking attribute and sort-by attribute
Algolia - index - <INDEX_NAME> - configuration - Ranking and Sorting
* Optional, Pagination and display. Add a highlight tag or attributes to snippet (only show certain number of words on search results).

6. Add link to result page with nuxt-link tag 

To use a variable in other template, we create a payload on nuxt.config.js 

```
generate: {
    routes () {
      return client.getEntries({ content_type: 'profile' }).then(entries => {
        return entries.items.map(entry => {
          return {
            route: entry.fields.name,
            payload: entry
          }
        })
      })
    }
  },
```
Then choose the value to use on link tag(nuxt-link)

```
<ais-hits>
      <template slot="item" slot-scope="{ item }">
        <p>
          <ais-highlight attribute="name" :hit="item" />&nbsp;
        </p>

        <nuxt-link :to="item.name">X</nuxt-link>
        
        <p>
          <ais-snippet attribute="exp" :hit="item" />
        </p>
      </template>
    </ais-hits>
  ```

6. On result template access to payload 

```
export default {
asyncData({ params, error, payload }) {
    if (payload) return { post: payload };
    return client
      .getEntries({
        content_type: 'profile',
        'fields.name': params.name
...
```

7. Create a Template 

```
<template>
  <section class="section">
    <div class="container">
      <div class="columns">
        <div class="column is-offset-2 is-8">
          <p class="subtitle is-6">
            <nuxt-link to="/search">Back to Blog home</nuxt-link>
          </p>
          <h1 class="title is-2">
            {{ post.fields.name }}
          </h1>
           <hr>
          <div v-html="experience"></div>
          <hr>
          <div v-html="richTextHtml"></div>
          <hr>
          <div v-html="newEvents"></div>
          <hr>
          <div v-html="insightsInnovations"></div>
        </div>
      </div>
    </div>
  </section>
</template>
```



...
