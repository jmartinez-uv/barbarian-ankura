<template>

  <ais-instant-search :search-client="searchClient" index-name="test_profile">

    <ais-search-box placeholder="Search" autofocus />

    <ais-stats />

    <ais-refinement-list attribute="name" />

    <ais-hits>
      <template slot="item" slot-scope="{ item }">
        <p>
          <ais-highlight attribute="name" :hit="item" />&nbsp;
        </p>
        <nuxt-link :to="item.name">X</nuxt-link>
        <p>
          <ais-snippet attribute="exp" :hit="item" />
        </p>
       <!--  <p>
          <ais-snippet attribute="newsEvents" :hit="item" />
        </p>
        <p>
          <ais-snippet attribute="honorsAwards" :hit="item" />
        </p>
        <p>
          <ais-snippet attribute="insightInnovations" :hit="item" />
        </p> -->
      </template>
    </ais-hits>

      <nuxt-link to="/">Back to Blog home</nuxt-link>
    <ais-pagination />
  </ais-instant-search>

</template>


<script>
import {
  AisInstantSearch,
  AisRefinementList,
  AisHits,
  AisHighlight,
  AisSearchBox,
  AisStats,
  AisPagination,
  AisSnippet,
  createServerRootMixin,
} from 'vue-instantsearch';
import algoliasearch from 'algoliasearch/lite';

const searchClient = algoliasearch(
    '3CHTSI59UD',
    'cb1da53e638c75abf4b37d91c2e8aa5d'
);

export default {
  components: {
    AisInstantSearch,
    AisRefinementList,
    AisHits,
    AisHighlight,
    AisSearchBox,
    AisStats,
    AisSnippet,
    AisPagination,
  },
  data() {
    return {
      searchClient,
    };
  },
  head() {
    return {
      link: [
        {
          rel: 'stylesheet',
          href: 'https://cdn.jsdelivr.net/npm/instantsearch.css@7.3.1/themes/algolia-min.css',
        },
      ],
    };
  },
};



</script>
