<template>
  <b-container>
    <b-form-group>
      <label for="taxonid">Taxon Id</label>
      <b-input-group>
        <b-form-input
          v-on:keypress="NumbersOnly"
          required
          id="taxonid"
          v-model="form.taxonId"
          :state="noEmpty"
        >
        </b-form-input>

        <b-button @click="getTaxon()" :disabled="!Boolean(noEmpty)">
          Get Taxons
        </b-button>

        <b-button @click="insertTaxon()" :disabled="!Boolean(noEmpty)">
          Add Taxon
        </b-button>

        <b-form-invalid-feedback :state="noEmpty" id="input-live-feedback">
          Input can't be empty.
        </b-form-invalid-feedback>
      </b-input-group>
    </b-form-group>

    <ModalTaxons v-if="Boolean(taxon)" :taxon="taxon"></ModalTaxons>

    <!--  <router-link :to="{name: 'taxons', params: {id : item} }" 
      v-for="(listTaxons, index) of listTaxon" :key="index">
      <button> fotso {{this.listTaxon}}</button>
    </router-link>-->

    <b-card-header header-tag="header" class="p-1" role="tab">
      <b-button
        @click="insertInfo"
        v-b-toggle.taxonName
        variant="dark"
        id="taxonName"
      >

        {{ this.listTaxon }}
      </b-button>

      <b-collapse id="taxonName" accordion="taxonName" role="tabpanel">
        <b-card-body>
          <b-card-text>
            <div v-html="this.info">
              {{ this.info[0] }}
            </div>
          </b-card-text>
        </b-card-body>
      </b-collapse>

      <br />
    </b-card-header>

    <!--<pre>{{ this.listTaxon }}</pre>-->
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
    noEmpty() {
      return this.form.taxonId.length > 0 ? true : false;
    },
  },
  data() {
    return {
      toggle: false,
      listTaxon: [],
      info: "",
      taxon: null,
      taxonResponse: Object,
      form: {
        taxonId: "",
      },
      //template: '<v-btn color="error" v-on:click="listTaxon++"> {{listTaxon}}</v-btn>'
    };
  },
  mounted(){
    this.$root.$on('insert-taxon', this.insertTaxon)
  },
  methods: {
    NumbersOnly(evt) {
      evt = evt ? evt : window.event;
      var charCode = evt.which ? evt.which : evt.keyCode;
      if (
        charCode > 31 &&
        (charCode < 48 || charCode > 57) &&
        charCode !== 46
      ) {
        evt.preventDefault();
      } else {
        return true;
      }
    },
    getTaxon() {
      checkListService
        .getEnaRecord(this.form.taxonId)
        .then((response) => {
          var xml2js = require("xml2js");
          var parser = new xml2js.Parser();

          parser.parseStringPromise(response.data).then((result) => {
            //console.log(result.TAXON_SET);
            const taxonResponse = result.TAXON_SET.taxon[0];
            this.taxon = taxonResponse;
            console.log(taxonResponse);
            this.$nextTick(() => {
              this.$root.$emit("bv::show::modal", "modal-taxons");
            });
            //console.log(taxonResponse);
            //console.log(this.getTaxon);
          });
        })
        .catch((e) => {
          this.message = e;
        });
    },
    insertTaxon() {
      checkListService
        .getEnaRecord(this.form.taxonId)
        .then((response) => {
          var xml2js = require("xml2js");
          var parser = new xml2js.Parser();
          parser.parseStringPromise(response.data).then((result) => {
            const taxonResponse = result.TAXON_SET.taxon[0];
            console.log(taxonResponse);
            this.taxon = taxonResponse;
            //console.log(this.taxon);
            this.listTaxon.push(this.taxon.$.scientificName);
            //this.$emit("click");
            //this.$emit("hover")
            //console.log(this.listTaxon);

            //console.log(taxonResponse.$.scientificName);

            //this.listTaxon = "";
            /*for(const nombre in taxonResponse.$.scientificName){
              this.listTaxon+=nombre+"<br>";
            }*/
            //this.listTaxon = taxonResponse.$;
            //console.log(taxonResponse);
          });
        })
        .catch((e) => {
          this.message = e;
        });
      //this.listTaxon.push(this.taxonResponse);
    },
    insertInfo() {
      checkListService
        .getEnaRecord(this.form.taxonId)
        .then((response) => {
          var xml2js = require("xml2js");
          var parser = new xml2js.Parser();

          parser.parseStringPromise(response.data).then((result) => {
            //console.log("hola");
            for (const name in result.TAXON_SET.taxon[0].$) {
              this.info +=
                name + ": " + result.TAXON_SET.taxon[0].$[name] + "<br>";
            }
            //this.info = result.TAXON_SET.taxon[0].$;
            console.log(result.TAXON_SET.taxon[0].$);
          });
        })
        .catch((e) => {
          this.message = e;
        });
    },
  },
};
</script>
<style>
.buttons-form {
  display: block;
}
</style>