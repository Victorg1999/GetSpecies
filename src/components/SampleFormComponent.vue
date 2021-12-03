<template>
  <div>
    <b-container>
      <b-row>
        <b-col>
          <b-form enctype="multipart/form-data">
            <div>
              <div>
                <b-form-group>
                  <label for="taxonid">Taxon Id</label>
                  <b-button
                    v-b-tooltip.hover
                    title="must insert a valid taxon id"
                    style="border: none"
                    variant="outline-info"
                    class="mb-2"
                  >
                  </b-button>
                  <b-input-group>
                    <b-form-input
                      required
                      id="taxonid"
                      v-model="form.taxonId"
                      :state="onlyNumbers"
                      @keypress="validateNumber"
                    >
                    </b-form-input>

                    <b-button
                      @click="getTaxon"
                      :disabled="!Boolean(onlyNumbers)"
                    >
                      Get Taxons
                      <ModalTaxons :taxon="taxon"></ModalTaxons>
                    </b-button>

                    <b-form-invalid-feedback
                      :state="onlyNumbers"
                      id="input-live-feedback"
                      @keypress="validateNumber"
                    >
                      You need put only numbers.
                    </b-form-invalid-feedback>
                  </b-input-group>
                </b-form-group>
              </div>
            </div>
          </b-form>
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
      taxon: Object,
      fieldGroups: [],
      message: "",
      checklistResponse: "",
      checkListState: false,
      form: {
        taxonId: "",
        taxon: null,
        checklist: "",
        fields: {},
      },
    };
  },
  methods: {
    validateNumber: (event) => {
      let keyCode = event.keyCode;
      if (keyCode < 48 || keyCode > 57) {
        event.preventDefault();
      }
    },
    getEnaChecklist() {
      checkListService
        .getEnaRecord(this.form.checklist)
        .then((response) => {
          var xml2js = require("xml2js");
          var parser = new xml2js.Parser();
          parser.parseStringPromise(response.data).then((result) => {
            this.joinFields(
              result.CHECKLIST_SET.CHECKLIST[0].DESCRIPTOR[0].FIELD_GROUP
            );
            this.fieldGroups =
              result.CHECKLIST_SET.CHECKLIST[0].DESCRIPTOR[0].FIELD_GROUP;
            this.checkListState = true;
            this.checklistResponse =
              result.CHECKLIST_SET.CHECKLIST[0].DESCRIPTOR[0].DESCRIPTION[0];
          });
        })
        .catch((e) => {
          this.checklistResponse = e;
        });
    },
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
          });
        })
        .catch((e) => {
          this.message = e;
        });
    },
    joinFields(fields) {
      var mergedList = [];
      fields.forEach((element) => {
        mergedList = mergedList.concat(element.FIELD);
      });
      mergedList.forEach((field) => {
        var name = field.NAME[0];
        var units = field.UNITS;
        this.$set(
          this.form.fields,
          name,
          units ? { value: null, unit: units[0].UNIT } : { value: null }
        );
      });
      return mergedList;
    },
    isTextInput(field) {
      return field.FIELD_TYPE && field.FIELD_TYPE[0].TEXT_FIELD;
    },
    isMultipleChoice(field) {
      return field.FIELD_TYPE && field.FIELD_TYPE[0].TEXT_CHOICE_FIELD;
    },
    testRegex(field) {
      if (
        field.FIELD_TYPE[0].TEXT_FIELD[0] &&
        field.FIELD_TYPE[0].TEXT_FIELD[0].REGEX_VALUE
      ) {
        const regex = new RegExp(
          field.FIELD_TYPE[0].TEXT_FIELD[0].REGEX_VALUE[0]
        );
        return regex.test(this.form.fields[field.NAME[0]].value);
      } else {
        return null;
      }
    },
    makeToast(variant = null, message = null, title = null) {
      this.$bvToast.toast(`${message || "Failed"}`, {
        title: `${title || "Failed"}`,
        variant: variant,
        autoHideDelay: 5000,

        appendToast: true,
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