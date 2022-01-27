<template>
  <div>
    <div id="gjs"></div>
  </div>
</template>



<script>
import grapesjs from "grapesjs";
import axios from "axios";
import bootstrap4Blocks from "grapesjs-blocks-bootstrap4";
import gjsPresetWebpage from "grapesjs-preset-webpage";
import CustomCode from "grapesjs-custom-code"

export default {
  props: ["apiEndPoint"],
  data: () => {
    return {
      name: "Ghani",
      editor: {},
    };
  },
  mounted() {
    this.init();
    this.getAssets();
  },
  methods: {
    getAssets() {
      axios
        .get(this.apiEndPoint + "/get-images")
        .then(({ data }) => {
          const am = this.editor.AssetManager;

          for (let image of data) {
            am.add(image.image);
          }
        })
        .catch((err) => {
          console.log(err.response.data.message);
        });
    },
    addAssets(images) {
      const am = this.editor.AssetManager;

      for (let image in images) {
        am.add(images[image]);
      }
    },
    saveTemplate(template, style) {
      axios
        .post(this.apiEndPoint + "/save-template", {
          name: "Sample",
          template: `${template}`,
          style: `${style}`,
        })
        .then((res) => {
          alert(res.data.message);
        })
        .catch((err) => {
          alert(err.response.data.message);
        });
    },
    init() {
      grapesjs.plugins.add("grapesjs-blocks-bootstrap4", bootstrap4Blocks);
      grapesjs.plugins.add("gjs-preset-webpage", gjsPresetWebpage);
      grapesjs.plugins.add("gjs-custom-code", CustomCode);

      const editor = grapesjs.init({
        // Indicate where to init the editor. You can also pass an HTMLElement
        container: "#gjs",
        plugins: ["gjs-custom-code", "grapesjs-blocks-bootstrap4", "gjs-preset-webpage"],
        pluginsOpts: {
          "gjs-preset-webpage": {
            modalImportTitle: "Import Template",
            modalImportLabel:
              '<div style="margin-bottom: 10px; font-size: 13px;">Paste here your HTML/CSS and click Import</div>',
            modalImportContent: function (editor) {
              return editor.getHtml();
            },
          },
        },
        assetManager: {
          assets: [],
          autoAdd: false,
          uploadFile: (e) => {
            // var files = e.dataTransfer ? e.dataTransfer.files : e.target.files;
            var files = e.target.files;

            var formData = new FormData();
            for (var i in files) {
              formData.append("files[]", files[i]);
            }

            axios
              .post(`${this.apiEndPoint}/save-images`, formData, {
                headers: {
                  "Content-Type": "multipart/form-data",
                },
              })
              .then(({ data }) => {
                this.addAssets(data.images);
              })
              .catch((err) => {
                alert(err.response.data.message);
              });
          },
        },
        canvas: {
          styles: [
            "https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css",
          ],
          scripts: [
            "https://code.jquery.com/jquery-3.3.1.slim.min.js",
            "https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js",
            "https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js",
          ],
        },

        // Get the content for the canvas directly from the element
        // As an alternative we could use: `components: '<h1>Hello World Component!</h1>'`,
        fromElement: true,
        // Size of the editor
        height: window.innerHeight + "px",
        width: "auto",
        // Disable the storage manager for the moment
        storageManager: false,
        // Avoid any default panel
        // We define a default panel as a sidebar to contain layers
      });

      editor.Panels.addButton("options", {
        id: "publish-btn",
        className: "publish-btn",
        command: "save-template",
        attributes: { title: "Publish Page" },
        active: false,
        label : '<span><button style="cursor:pointer">Publish Page</button></span>'
      });

      editor.Commands.add("save-template", {
        run: (editor) => {
          this.saveTemplate(editor.getHtml(), editor.getCss());
          return editor;
        },
      });
      this.editor = editor;
    },
  },
};
</script>

<style scoped>
@import "~grapesjs/dist/css/grapes.min.css";
/* Let's highlight canvas boundaries */
body {
  margin: 0;
}
</style>