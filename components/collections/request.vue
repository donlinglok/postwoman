<template>
  <div class="flex-wrap">
    <div>
      <button class="icon" @click="selectRequest()" v-tooltip="'Use request'">
        <i class="material-icons">insert_drive_file</i>
        <span>{{ request.name }}</span>
      </button>
    </div>
    <div class="hide-on-small-screen">
      <button class="icon" @click="removeRequest" v-tooltip="'Delete request'">
        <i class="material-icons">delete</i>
      </button>
      <button
        class="icon"
        @click="$emit('edit-request')"
        v-tooltip="'Edit request'"
      >
        <i class="material-icons">edit</i>
      </button>
    </div>
    <v-popover class="hide-on-large-screen">
      <button class="tooltip-target icon" v-tooltip="'More'">
        <i class="material-icons">more_vert</i>
      </button>
      <template slot="popover">
        <div>
          <button
            class="icon"
            @click="removeRequest"
            v-tooltip="'Delete request'"
            v-close-popover
          >
            <i class="material-icons">delete</i>
            <span>Delete</span>
          </button>
        </div>
        <div>
          <button
            class="icon"
            @click="$emit('edit-request')"
            v-tooltip="'Edit request'"
            v-close-popover
          >
            <i class="material-icons">edit</i>
            <span>Edit</span>
          </button>
        </div>
      </template>
    </v-popover>
  </div>
</template>

<style scoped>
ul {
  display: flex;
  flex-direction: column;
}

ul li {
  display: flex;
  padding-left: 16px;
  border-left: 1px solid var(--brd-color);
}
</style>

<script>
export default {
  props: {
    request: Object,
    collectionIndex: Number,
    folderIndex: Number,
    requestIndex: Number
  },
  methods: {
    selectRequest() {
      this.$store.commit("postwoman/selectRequest", { request: this.request });
    },
    removeRequest() {
      if (!confirm("Are you sure you want to remove this request?")) return;
      this.$store.commit("postwoman/removeRequest", {
        collectionIndex: this.collectionIndex,
        folderIndex: this.folderIndex,
        requestIndex: this.requestIndex
      });
    }
  }
};
</script>
