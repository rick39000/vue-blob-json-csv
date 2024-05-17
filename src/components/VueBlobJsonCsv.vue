<template>
  <component :is="tagName" ref="download" @click="handleDownload()">
    <template v-if="title === ''">
      <slot></slot>
    </template>
    <template>
      {{ title }}
    </template>
  </component>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "VueBlobJsonCsv",
  props: {
    tagName: {
      type: String,
      required: false,
      default: "span"
    },
    title: {
      type: String,
      required: false,
      default: ""
    },
    fileType: {
      type: String,
      required: true,
      default: ""
    },
    delimiter: {
      type: String,
      required: false,
      default: ","
    },
    fileName: {
      type: String,
      required: false,
      default: "data"
    },
    data: {
      type: Array,
      required: true,
      default: () => []
    },
    fields: {
      type: Array,
      required: false,
      default: () => []
    },
    confirm: {
      type: String,
      required: false,
      default: null
    }
  },
  computed: {
    createContent: function(): string | null {
      let content = null;

      if (this.fileType === "json") {
        content = JSON.stringify(this.data);
      } else if (this.fileType === "csv") {
        const keys =
          this.fields.length > 0
            ? this.fields
            : Object.keys(this.data[0] as string[]);
        let csv = `\ufeff${keys.join(this.delimiter)}\n`;

        for (let index = 0; index < this.data.length; index++) {
          const item: any = this.data[index];
          let line = keys
            .map((key: any) => {
              if (item[key] === null) {
                return null;
              } else if (typeof item[key] === "object") {
                let result = JSON.stringify([item[key]]);
                if (result.search(this.delimiter) >= 0) {
                  result = '"' + result.replace(/"/, '""') + '"';
                }
                return result;
              } else {
                let result = item[key];
                if (
                  typeof result === "string" &&
                  result.search(this.delimiter) >= 0
                ) {
                  result = '"' + result.replace(/"/, '""') + '"';
                }
                return [result];
              }
            })
            .join(this.delimiter);
          csv += `${line}\n`;
        }
        content = csv;
      }
      return content;
    }
  },
  methods: {
    handleDownload(): void {
      let content = this.createContent;

      if (content === null) {
        this.$emit("error");
        return;
      }

      if (this.confirm !== null) {
        const result = confirm(this.confirm);
        if (!result) return;
      }

      const blob = new Blob([content], {
        type: `application/${this.fileType}`
      });
      const link = document.createElement("a");
      link.href = window.URL.createObjectURL(blob);
      link.download = `${this.fileName}.${this.fileType}`;
      link.click();
      this.$emit("success");
    }
  }
});
</script>
