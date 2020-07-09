<template>
  <div class="container">
    <h1 class="title is-2">hecho</h1>
    <hr>
    <nuxt-link to="/">Back to Blog home</nuxt-link>
  </div>
        
</template>

<script>
    import { documentToHtmlString } from '@contentful/rich-text-html-renderer';
    import {createClient} from '~/plugins/contentful.js'
    import algoliasearch from 'algoliasearch/lite';
    import removeMd from 'remove-markdown';

    const client = createClient();

export default {

  asyncData ({env}) {
    const algoliaClient = algoliasearch(
      env.ALGOLIA_APP_ID,
      env.ALGOLIA_ADMIN_KEY
    );
    const algoliaIndex = algoliaClient.initIndex(env.ALGOLIA_INDEX);

//                Promise
//---------------------------------------------------
    return client.getEntries({
            'content_type': env.CTF_BLOG_POST_TYPE_ID,
          })
        .then(entries => {
          //create the json data to be sent

            const profile = entries.items.map(post => ({
              name: post.fields.name,
              exp: removeMd(documentToHtmlString(post.fields.ex)),
              honorsAwards: removeMd(documentToHtmlString(post.fields.honorsAwards)),
              newsEvents: removeMd(documentToHtmlString(post.fields.newsEvents)),
              insightInnovations: removeMd(documentToHtmlString(post.fields.insightInnovations)),
              objectID: post.sys.id
            }));  

          //send data to algolia and create records 
          const indexedContent = algoliaIndex.saveObjects(profile,true);
          console.log(profile.length);
          //console.log(profile);
    })
    .catch(e => console.log(e));

    //console.log(profile);

  },
  head: {
    title: 'Update Data',
  },
}
  </script>