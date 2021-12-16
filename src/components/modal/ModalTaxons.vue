<template>
  <b-modal
    :title="taxon.$.scientificName"
    id="modal-taxons"
    ok-only
    @ok="buttonClickHandler()"
  >
    <b-col>
      <b-card>
        <b-card-body>
          <b-card-text>
            <strong>Scientific Name </strong> {{ taxon.$.scientificName }}
          </b-card-text>
          <b-card-text>
            <strong>taxID </strong> {{ taxon.$.taxId }}
          </b-card-text>
          <b-card-text> <strong>rank </strong> {{ taxon.$.rank }} </b-card-text>
          <b-card-text>
            <strong>hidden </strong> {{ taxon.$.hidden }}
          </b-card-text>
        </b-card-body>
      </b-card>
    </b-col>

    <b-card no-body class="mb-1">
      <b-card-header header-tag="header" class="p-1" role="tab">
        <b-button block v-b-toggle.lineage variant="dark" id="lineage"
          >Lineage</b-button
        >
      </b-card-header>
      <b-collapse id="lineage" accordion="lineage" role="tabpanel">
        <b-card-body>
          <b-card-text
            v-for="taxon in taxon.lineage[0].taxon.slice().reverse()"
            v-bind:key="taxon.$.taxId"
          >
            <p v-if="ranki.includes(taxon.$.rank)">
              <strong> {{ taxon.$.rank }} </strong>
              {{ taxon.$.scientificName }}
            </p>
          </b-card-text>
        </b-card-body>
      </b-collapse>
    </b-card>
  </b-modal>
</template>

<script>
export default {
  name: "modal-taxons",
  props: ["taxon"],
  data() {
    return {
      ranki: [
        "root",
        "superkingdom",
        "kingdom",
        "phylum",
        "class",
        "order",
        "family",
        "genus",
        "species",
        "subspecies",
      ],
    };
  },

  computed: {
    filteredRanks: function (taxon) {
      this.taxon.lineage[0].taxon.filter((taxon) => {
        this.taxon.$.rank.includes(taxon.ranki);
      });
      return taxon.ranki;
    },
  },
  mounted() {},
  methods: {
    buttonClickHandler() {
      const taxon = JSON.parse(JSON.stringify(this.taxon));
      this.$emit("insert-taxon", taxon);
    },
  },
};
</script>