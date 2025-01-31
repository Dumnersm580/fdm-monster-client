<template>
  <v-container>
    <v-row>
      <v-col v-if="formData" cols="12" md="6">
        <validation-provider v-slot="{ errors }" :rules="printerFloorNameRules" name="Name">
          <v-text-field
            v-model="formData.name"
            :error-messages="errors"
            autofocus
            label="Floor name*"
            required
          />
        </validation-provider>

        <validation-provider v-slot="{ errors }" :rules="floorNumberRules" name="FloorNumber">
          <v-text-field
            v-model="formData.floor"
            :error-messages="errors"
            label="Floor number"
            required
            type="number"
          />
        </validation-provider>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { defineComponent, inject } from "vue";
import { ValidationProvider } from "vee-validate";
import { AppConstants } from "@/constants/app.constants";
import {
  getDefaultCreatePrinterFloor,
  PreCreateFloor,
} from "@/models/printer-floor/printer-floor.model";
import { FloorService } from "../../../backend/floor.service";
import { usePrintersStore } from "@/store/printers.store";

const watchedId = "printerFloorId";

interface Data {
  formData: PreCreateFloor;
}

export default defineComponent({
  name: "PrinterFloorCrudForm",
  components: {
    ValidationProvider,
  },
  setup: () => {
    return {
      printersStore: usePrintersStore(),
      appConstants: inject("appConstants") as AppConstants,
    };
  },
  async created() {
    if (this.printerFloorId) {
      const crudeData = this.printersStore.floor(this.printerFloorId);
      this.formData = FloorService.convertPrinterFloorToCreateForm(crudeData);
    } else if (this.printersStore.floors?.length) {
      const maxIndex = Math.max(...this.printersStore.floors.map((pf) => pf.floor)) + 1;
      this.formData.floor = maxIndex.toString();
    }
  },
  async mounted() {},
  props: {
    printerFloorId: String,
  },
  data: (): Data => ({
    formData: getDefaultCreatePrinterFloor(),
  }),
  computed: {
    printerFloorNameRules() {
      return { required: true, min: this.appConstants.minPrinterFloorNameLength };
    },
    floorNumberRules() {
      return {
        required: true,
        integer: true,
      };
    },
  },
  methods: {},
  watch: {
    [watchedId](val?: string) {
      if (!val) return;
      const printerFloor = this.printersStore.floor(val);
      this.formData = FloorService.convertPrinterFloorToCreateForm(printerFloor);
    },
  },
});
</script>
