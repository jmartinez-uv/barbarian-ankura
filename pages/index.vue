<template>
  <div class="container">
    <h1 class="title is-2">{{ person.fields.name }}</h1>
    <!-- render blog posts -->
    <hr>
    <div v-html="experience"></div>
    <hr>
    <div v-html="richTextHtml"></div>
    <hr>
    <div v-html="newEvents"></div>
    <hr>
    <div v-html="insightsInnovations"></div>
  </div>
</template>

<script>
//library used to import the format from contentful speficy a "rich text"
  import { documentToHtmlString } from '@contentful/rich-text-html-renderer';
  import {createClient} from '~/plugins/contentful.js'
  import algoliasearch from 'algoliasearch/lite';
  // library used to remove tags from contentful
  import removeMd from 'remove-markdown';

  const client = createClient();

   export default {

    // `env` is available in the context object
    asyncData ({env}) {

//initialize the algolia plug in with the id from contentfull webhook
  const algoliaClient = algoliasearch(
    env.ALGOLIA_APP_ID,
    env.ALGOLIA_ADMIN_KEY
  );

// link to algolia index
  const algoliaIndex = algoliaClient.initIndex(env.ALGOLIA_INDEX);

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
        //console.log('Indexed Content:', indexedContent);

        // return data that should be available
        // in the template
        return {
          person: entries.items[0],
          experience: documentToHtmlString(entries.items[0].fields.ex),
          richTextHtml: documentToHtmlString(entries.items[0].fields.honorsAwards),
          newEvents: documentToHtmlString(entries.items[0].fields.newsEvents),
          insightsInnovations: documentToHtmlString(entries.items[0].fields.insightInnovations),   
        } 

      })
      .catch(console.error)

    }

  }  

</script>