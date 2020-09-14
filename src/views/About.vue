<template>
  <div class="about">
    <h1>This is an about page</h1>
  </div>
</template>


<script>
import * as tf from '@tensorflow/tfjs'
import * as tfvis from '@tensorflow/tfjs-vis'
import { getValue, convertToTensor } from '@/common/getData'
export default {
  name: 'About',
  mounted () {
    this.init()
  },
  methods: {
    init () {
      console.log(tf, tfvis);
      this.run()
    },

    createModel() {
      // 定义模型架构
      // Create a sequential model 实例化模型
      const model = tf.sequential();
      
      // Add a single input layer 添加图层
      /*
      这将为我们的网络添加一个输入层，该输入层会自动连接到具有一个隐藏单元的“密集”层。
      dense图层是一种图层类型，将其输入乘以一个矩阵（称为weights），然后将一个数字（称为bias）相加。
      由于这是网络的第一层，因此我们需要定义inputShape。
      这inputShape是[1]因为我们有1数字作为输入（给定汽车的马力）。
      units 设置权重矩阵在图层中的大小。
      通过在此处将其设置为1，我们可以说数据的每个输入要素的权重为1。
      注意：默认情况下，密集层带有一个偏差项，因此我们不需要将useBias设置为true，我们将在以后的调用中省略tf.layers.dense
      */
      model.add(tf.layers.dense({inputShape: [1], units: 50, activation: 'sigmoid'}));
      // model.add(tf.layers.dense({inputShape: [1], units: 1, useBias: true}));
      
      // Add an output layer
      model.add(tf.layers.dense({units: 1, useBias: true}));
      // model.add(tf.layers.dense({units: 50, activation: 'sigmoid'}));

      return model;
    },

    async run() {
      // Load and plot the original input data that we are going to train on.
      const data = await getValue();
      const values = data.map(d => ({
        x: d.horsepower,
        y: d.mpg,
      }));

      tfvis.render.scatterplot( // 渲染马力和效率二维坐标图
        {name: 'Horsepower v MPG'},
        {values},
        {
          xLabel: 'Horsepower',
          yLabel: 'MPG',
          height: 300
        }
      );

      const model = this.createModel(); // 创建Model
      tfvis.show.modelSummary({name: 'Model Summary'}, model); // 把图层信息显示出来

      // Convert the data to a form we can use for training.
      const tensorData = convertToTensor(data); // 准备训练数据
      const {inputs, labels} = tensorData;
          
      // Train the model
      await this.trainModel(model, inputs, labels); // 训练模型
      this.testModel(model, data, tensorData); // 测试模型
      console.log('Done Training');
    },

    /**
     * Convert the input data to tensors that we can use for machine 
     * learning. We will also do the important best practices of _shuffling_
     * the data and _normalizing_ the data
     * MPG on the y-axis.
     */
    
    async trainModel(model, inputs, labels) {
      // 训练模型
      // Prepare the model for training.
      model.compile({
        optimizer: tf.train.adam(),
        loss: tf.losses.meanSquaredError,
        metrics: ['mse'],
      });
      
      const batchSize = 32;
      const epochs = 50;
      
      return await model.fit(inputs, labels, {
        batchSize,
        epochs,
        shuffle: true,
        callbacks: tfvis.show.fitCallbacks(
          { name: 'Training Performance' },
          ['loss', 'mse'], 
          { height: 200, callbacks: ['onEpochEnd'] }
        )
      });
    },

    testModel(model, inputData, normalizationData) {
      // 做出预测
      const {inputMax, inputMin, labelMin, labelMax} = normalizationData;  
      
      // Generate predictions for a uniform range of numbers between 0 and 1;
      // We un-normalize the data by doing the inverse of the min-max scaling 
      // that we did earlier.
      const [xs, preds] = tf.tidy(() => {
        
        const xs = tf.linspace(0, 1, 100);      
        const preds = model.predict(xs.reshape([100, 1]));
        
        const unNormXs = xs
          .mul(inputMax.sub(inputMin))
          .add(inputMin);
        
        const unNormPreds = preds
          .mul(labelMax.sub(labelMin))
          .add(labelMin);
        
        // Un-normalize the data
        return [unNormXs.dataSync(), unNormPreds.dataSync()];
      });
      
    
      const predictedPoints = Array.from(xs).map((val, i) => {
        return {x: val, y: preds[i]}
      });
      
      const originalPoints = inputData.map(d => ({
        x: d.horsepower, y: d.mpg,
      }));
      
      
      tfvis.render.scatterplot(
        {name: 'Model Predictions vs Original Data'}, 
        {values: [originalPoints, predictedPoints], series: ['original', 'predicted']}, 
        {
          xLabel: 'Horsepower',
          yLabel: 'MPG',
          height: 300
        }
      );
    }
  }
}
</script>