<template>
  <ais-instant-search :search-client="searchClient" index-name="test_profile">
    <ais-search-box />
    <ais-stats />
    <ais-refinement-list attribute="brand" />
    <ais-hits>
      <template slot="item" slot-scope="{ item }">
        <p >
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


<script>
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
