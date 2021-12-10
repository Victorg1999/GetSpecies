<template>
  <div>
    <b-container>
      <b-row>
        <b-col>
          <b-form enctype="multipart/form-data">
            <div>
              <div>
                <b-form-group>
                  <label for="taxonid">Taxeron Id</label>
                  <b-input-group>
                    <b-form-input
                      required
                      id="taxonid"
                      v-model="form.taxonId"
                      :state="onlyNumbers"
                    >
                    </b-form-input>

                    <b-button
                      @click="getTaxon"
                      :disabled="!Boolean(onlyNumbers)"
                    >
                      Get Taxons
                      <ModalTaxons :taxon="taxon"></ModalTaxons>
                    </b-button>

                    <b-button @click="insertTaxon"> Add Taxon </b-button>

                    <b-form-invalid-feedback
                      :state="onlyNumbers"
                      id="input-live-feedback"
                    >
                      You need put only numbers.
                    </b-form-invalid-feedback>
                  </b-input-group>
                </b-form-group>
              </div>
            </div>
          </b-form>
          <!--<pre>{{ JSON.stringify(this.listTaxon, null, 2) }}</pre>-->
        </b-col>
      </b-row>
    </b-container>
  </div>
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
    onlyNumbers() {
      return this.form.taxonId.length > 0 ? true : false;
    },
  },
  data() {
    return {
      //listTaxon: [],
      taxon: Object,
      form: {
        taxonId: "",
      },
    };
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
            this.$root.$emit("bv::show::modal", "modal-taxons");
            //console.log(taxonResponse);
          });
        })
        .catch((e) => {
          this.message = e;
        });
    },
    /*insertTaxon() {
      this.listTaxon.push(this.taxon);
    },*/
  },
};
</script>
<style>
.buttons-form {
  display: block;
}
</style>