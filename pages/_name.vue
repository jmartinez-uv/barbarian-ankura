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

<script>
  import { documentToHtmlString } from '@contentful/rich-text-html-renderer';
  import {createClient} from '~/plugins/contentful.js'
  import removeMd from 'remove-markdown';

    const client = createClient();

export default {
  asyncData({ params, error, payload }) {
    if (payload) return { post: payload };
    return client
      .getEntries({
        content_type: 'profile',
        'fields.name': params.name
      })
      .then(entries => {
        return { 
          post: entries.items[0],
          experience: documentToHtmlString(entries.items[0].fields.ex),
          richTextHtml: documentToHtmlString(entries.items[0].fields.honorsAwards),
          newEvents: documentToHtmlString(entries.items[0].fields.newsEvents),
          insightsInnovations: documentToHtmlString(entries.items[0].fields.insightInnovations),   
        };
      })
      .catch(e => console.log(e));
  },
  head() {
    return {
      title: this.post.fields.name,
    };
  },
};
</script>