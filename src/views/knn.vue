<template>
  <div>
    knn分类器
    <div @click="run" style="display:flex;flex-wrap:wrap">
      <img id='class0' src='../assets/cj.png'/>
      <img id='class1' src='../assets/ct.png'/>
      <img id='test' src='../assets/ks.png'/>
    </div>
    <button @click="clean">清空模型</button>
  </div>
</template>

<script>
import * as tf from "@tensorflow/tfjs";
import * as knnClassifier from "@tensorflow-models/knn-classifier";
import * as mobilenetModule from "@tensorflow-models/mobilenet";

export default {
  name: "Knn",
  mounted() {
    this.run()
  },
  methods: {
    clean () {
      const classifier = knnClassifier.create();
      classifier.clearAllClasses()
      console.log('清空完成');
    },

    async run () {
      console.log('开始');
      const classifier = knnClassifier.create();

      const mobilenet = await mobilenetModule.load({
        version: 2,
        alpha: 1,
      });

      const dog = await this.imgP("./ks.png");
      const n = await this.imgP("./cj.png");
      const z = await this.imgP("./close.png");
      console.log(tf.browser.fromPixels(dog));
      const logits0 = mobilenet.infer(
        tf.browser.fromPixels(dog),
        "conv_preds"
      );
      for (let i = 0; i < 100; i++) {
        classifier.addExample(logits0, "dog");
      }
      classifier.addExample(logits0, "dog");

      const logits1 = mobilenet.infer(
        tf.browser.fromPixels(n),
        "conv_preds"
      );
      for (let i = 0; i < 100; i++) {
        classifier.addExample(logits1, "n");
      }
      classifier.addExample(logits1, "n");


      console.log("识别结果:");
      const logits = mobilenet.infer(
        tf.browser.fromPixels(z),
        "conv_preds"
      );
      const result = await classifier.predictClass(logits);
      console.log("dog", result);
    },

    imgP (url) {
      return new Promise((r) => {
        const img = new Image();
        img.src = url;
        img.onload = () => {
          r(img);
        };
      });
    }


  },
};
</script>
