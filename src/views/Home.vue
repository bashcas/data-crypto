<template>
  <div>
    <pulse-loader :loading="isLoading" :color="'#68d391'" :size="100" />
    <PxAssetsTable v-if="!isLoading" :assets="assets" />
  </div>
</template>

<script>
import { getAssets } from "@/api"
import PxAssetsTable from "@/components/PxAssetsTable"

export default {
  name: "Home",
  components: { PxAssetsTable },

  data() {
    return {
      isLoading: false,
      assets: []
    }
  },

  created() {
    this.isLoading = true
    getAssets()
      .then(assets => (this.assets = assets))
      .finally(() => (this.isLoading = false))
  }
}
</script>
