<template>
  <b-container>
    <b-form-group>
      <label for="taxonid">Taxon Id</label>
      <b-input-group>
        <b-form-input
          required
          id="taxonid"
          v-model="form.taxonId"
          :state="!correctInput"
          type="number"
        >
        </b-form-input>

        <b-button @click="getTaxon()" :disabled="correctInput">
          Get Taxons
        </b-button>

        <b-form-invalid-feedback
          :state="!correctInput"
          id="input-live-feedback"
        >
          Input can't be empty and only accept numbers.
        </b-form-invalid-feedback>
      </b-input-group>
    </b-form-group>

    <ModalTaxons
      v-on:insert-taxon="insertTaxon"
      v-if="Boolean(taxon)"
      :taxon="taxon"
    ></ModalTaxons>

    <b-dropdown
      v-if="listTaxon.length > 0"
      text="Species List"
      block
      variant="primary"
      class="n-2"
    >
      <b-dropdown-item
        @click="infoOfModal(tax)"
        v-for="tax in listTaxon"
        v-bind:key="tax.$.scientificName"
        >{{ tax.$.scientificName }}</b-dropdown-item
      >
    </b-dropdown>
  </b-container>
</template>
<script>
import ModalTaxons from "../components/modal/ModalTaxons.vue";
import checkListService from "../services/CheckListService";
export default {
  name: "sample-input-form",
  components: {
    ModalTaxons,
  },
  computed: {
    correctInput() {
      return (
        this.form.taxonId.length === 0 || typeof this.form.taxonId === "number"
      );
    },
  },
  data() {
    return {
      toggle: false,
      listTaxon: [],
      taxon: null,
      taxonResponse: Object,
      form: {
        taxonId: "",
      },
    };
  },
  mounted() {
    this.$root.$on("insert-taxon", this.insertTaxon);
  },
  methods: {
    getTaxon() {
      checkListService
        .getEnaRecord(this.form.taxonId)
        .then((response) => {
          var xml2js = require("xml2js");
          var parser = new xml2js.Parser();

          parser.parseStringPromise(response.data).then((result) => {
            const taxonResponse = result.TAXON_SET.taxon[0];
            this.taxon = taxonResponse;
            console.log(taxonResponse);
            this.$nextTick(() => {
              this.$root.$emit("bv::show::modal", "modal-taxons");
            });
          });
        })
        .catch((e) => {
          this.message = e;
        });
    },
    insertTaxon(value) {
      const taxon = value;
      if (
        this.listTaxon.length === 0 ||
        this.listTaxon.filter(
          (txn) => txn.$.scientificName === taxon.$.scientificName
        ).length === 0
      ) {
        this.listTaxon.push(taxon);
      }
    },
    infoOfModal(taxon) {
      this.taxon = taxon;
      this.$root.$emit("bv::show::modal", "modal-taxons");
    },
  },
};
</script>
<style>
.buttons-form {
  display: block;
}
</style>