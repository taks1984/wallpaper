<template>
  <v-app>
    <v-main>
      <v-bottom-sheet
        v-model="sheet"
        inset
        max-width="800"
      >
        <v-sheet class="pa-4">
          <h2 class="mb-4 text-center">Wallpaper Configurations</h2>
          <v-expansion-panels accordion>
            <v-expansion-panel>
              <v-expansion-panel-header>Canvas Settings</v-expansion-panel-header>
              <v-expansion-panel-content>
                <v-row>
                  <v-col cols="4">
                    <v-text-field
                      type="number"
                      v-model="canvas.width"
                      label="Width (under 3072px)"
                      @change="drawImages"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      type="number"
                      v-model="canvas.height"
                      label="Height (under 2048px)"
                      @change="drawImages"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-select
                      v-model="canvas.background"
                      :items="['Wood','Transparent']"
                      label="Background"
                      @change="drawImages"
                    ></v-select>
                  </v-col>
                </v-row>
              </v-expansion-panel-content>
            </v-expansion-panel>

            <v-expansion-panel
              v-for="(image, index) in images"
              :key="index"
            >
              <v-expansion-panel-header>Image #{{ index + 1 }}<span v-if="image.filename" class="ml-4">{{image.filename}}</span></v-expansion-panel-header>
              <v-expansion-panel-content>
                <v-row dense>
                  <v-col cols="8">
                    <v-file-input
                      label="Upload your file"
                      truncate-length="80"
                      prepend-icon=""
                      outlined
                      dense
                      @change="onImagePicked(index, $event)"
                    ></v-file-input>
                  </v-col>
                  <v-col cols="4">
                    <img
                      class="thumbnail"
                      :src="image.src"
                    />
                  </v-col>
                </v-row>
                <v-row dense>
                  <v-col cols="4">
                    <v-text-field
                      type="number"
                      v-model.number="images[index].left"
                      label="X"
                      dense
                      @change="drawImages"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      type="number"
                      v-model.number="images[index].top"
                      label="Y"
                      dense
                      @change="drawImages"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      type="number"
                      v-model.number="images[index].depth"
                      label="Z"
                      dense
                      @change="drawImages"
                    ></v-text-field>
                  </v-col>
                </v-row>
                <v-row dense>
                  <v-col cols="4">
                    <v-text-field
                      type="number"
                      v-model.number="images[index].scale"
                      label="Scale (0 ~ 1)"
                      dense
                      @change="drawImages"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      type="number"
                      v-model.number="images[index].rotate"
                      label="Rotation (0 ~ 360)"
                      dense
                      @change="drawImages"
                    ></v-text-field>
                  </v-col>
                  <v-col cols="4">
                    <v-text-field
                      type="number"
                      v-model.number="images[index].borderWidth"
                      label="Border Width"
                      dense
                      :disabled="images[index].transparent"
                      @change="drawImages"
                    ></v-text-field>
                  </v-col>
                </v-row>
                <v-row dense>
                  <v-col cols="4">
                    <v-checkbox
                      v-model="images[index].shadow"
                      label="Shadow"
                      dense
                      hide-details
                      @change="drawImages"
                    ></v-checkbox>
                    <v-checkbox
                      v-model="images[index].transparent"
                      label="Transparent"
                      dense
                      hide-details
                      @change="drawImages"
                    ></v-checkbox>
                  </v-col>
                </v-row>
              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>

          <v-btn
            class="mt-4"
            @click="addImage"
          >+</v-btn>
        </v-sheet>
      </v-bottom-sheet>

      <canvas
        id="wp"
        class="ma-4"
        @click="sheet = true"
      ></canvas>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: "App",

  data: () => ({
    sheet: true,
    canvas: {
      width: 1920,
      height: 1080,
      background: "Wood",
    },
    images: [
      {
        src: "",
        left: 100,
        top: 100,
        scale: 1,
        rotate: 0,
        depth: 0,
        borderWidth: 40,
        transparent: false,
        shadow: true,
      },
      {
        src: "",
        left: 100,
        top: 100,
        scale: 1,
        rotate: 0,
        depth: 0,
        borderWidth: 40,
        transparent: false,
        shadow: true,
      },
      {
        src: "",
        left: 100,
        top: 100,
        scale: 1,
        rotate: 0,
        depth: 0,
        borderWidth: 40,
        transparent: false,
        shadow: true,
      },
    ],
  }),
  created: function () {
    window.addEventListener("beforeunload", this.beforeUnloadWarning);
  },
  mounted: async function () {
    await this.drawImages();
  },
  destroyed() {
    window.removeEventListener("beforeunload", this.beforeUnloadWarning);
  },
  methods: {
    beforeUnloadWarning: function (e) {
      e.returnValue = "変更内容は保存されません";
    },
    addImage: function () {
      this.images.push({
        src: "",
        left: 100,
        top: 100,
        scale: 1,
        rotate: 0,
        depth: 0,
        borderWidth: 40,
        transparent: false,
        shadow: true,
      });
    },
    drawImages: async function () {
      var canvas = document.getElementById("wp");
      canvas.width = this.canvas.width;
      canvas.height = this.canvas.height;
      canvas.imageSmoothingEnabled = false;
      canvas.imageSmoothingQuality = "high";

      if (canvas.getContext) {
        var context = canvas.getContext("2d");

        (async () => {
          // 背景を描画
          if (this.canvas.background === "Wood") {
            await this.drawImage(context, {
              src: "./images/wood-texture_00005.jpg",
              left: 0,
              top: 0,
              scale: 1,
              borderWidth: 0,
            });
          }

          // 全ての画像を描画
          const images = [...this.images];
          images.sort((x, y) => y.depth - x.depth);

          for (let i = 0; i < images.length; i++) {
            if (images[i].src) {
              await this.drawImage(context, images[i]);
            }
          }
        })();
      }
    },
    drawImage: function (ctx, config) {
      console.log("drawImage");
      console.log(config);
      const borderColor = "#FFF";

      return new Promise((resolve) => {
        var image = new Image();
        image.addEventListener(
          "load",
          function () {
            console.log("loaded");
            const left = config.left;
            const top = config.top;
            const width = Math.floor(image.naturalWidth * config.scale);
            const height = Math.floor(image.naturalHeight * config.scale);

            console.log([left, top, width, height]);

            ctx.save();

            ctx.translate(left + width / 2, top + height / 2);
            ctx.rotate((config.rotate * Math.PI) / 180);
            ctx.translate(-(left + width / 2), -(top + height / 2));
            ctx.fillStyle = borderColor;
            ctx.shadowColor = "#000";

            if (!config.transparent) {
              // 枠線を描画
              ctx.shadowBlur = config.shadow ? 10 : 0;
              ctx.fillRect(
                left,
                top,
                width + config.borderWidth * 2,
                height + config.borderWidth * 2
              );
            }

            // 画像を描画
            ctx.shadowBlur = config.shadow && config.transparent ? 10 : 0;
            ctx.drawImage(
              image,
              left + config.borderWidth,
              top + config.borderWidth,
              width,
              height
            );

            ctx.restore();
            resolve();
          },
          false
        );

        image.src = config.src;
      });
    },
    onImagePicked: function (target, file) {
      if (file !== undefined && file !== null) {
        if (file.name.lastIndexOf(".") <= 0) {
          return;
        }
        const fr = new FileReader();
        fr.addEventListener("load", () => {
          this.images[target].filename = file.name;
          this.images[target].src = fr.result;
          this.drawImages();
        });

        fr.readAsDataURL(file);
      }
    },
  },
};
</script>

<style scoped>
#wp {
  filter: drop-shadow(0px 0px 10px #888);
  background: url("/images/74820679_p0.png") repeat;
  margin: 0px auto;
}

.v-main {
  overflow: auto;
}
.v-sheet {
  overflow: auto;
}
.thumbnail {
  height: 60px;
}
</style>
