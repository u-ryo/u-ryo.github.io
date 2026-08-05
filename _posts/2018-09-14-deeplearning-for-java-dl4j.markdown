---
layout: post
title: "DeepLearning for Java DL4J"
date: "2018-09-14 00:13"
author: 'u-ryo'
categories: [Java, Machine Learning, AI]
comments: true
published: true
---
会社はケチな(本気でMLやる気はない)のでマシンもGPUもなく、
やむなく[Google Colaboratory](https://colab.research.google.com/)上で[keras](https://keras.io/ja/)でVGG16 with ImageNetをfine tuningしたmodelを、
GPUのないNotePCでpredictしようとしました。
NotePCはDisk/Memory容量が小さいためpythonも本気では入れてないので、
[Groovy](http://groovy-lang.org/)+[DeepLearning4J](https://deeplearning4j.org/)で何とかならないかなー、
とあがきました。
単純にGrapeでの指定だけだと失敗するんです。

```
$ groovy dl4j.groovy
Caught: java.lang.NoClassDefFoundError: org/bytedeco/javacpp/hdf5$Group
java.lang.NoClassDefFoundError: org/bytedeco/javacpp/hdf5$Group
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasModelBuilder.modelHdf5Filename(KerasModelBuilder.java:226)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport.importKerasSequentialModelAndWeights(KerasModelImport.java:194)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport$importKerasSequentialModelAndWeights.call(Unknown Source)
        at dl4j.run(dl4j.groovy:17)
Caused by: java.lang.ClassNotFoundException: org.bytedeco.javacpp.hdf5$Group
        ... 4 more
```

これは、何も指定しないと`~/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2-windows-x86_64.jar`が落ちてきて、Windows用native libraryで動こうとするからっぽい?んですね。
なので、`hdf5-1.10.2-1.4.2-linux-x86_64.jar`を落としてきて、`-cp`で明示的に指定しないとダメでした。

```
$ groovy -cp ~/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2-linux-x86_64.jar dl4j.groovy
[main] INFO org.deeplearning4j.nn.modelimport.keras.Hdf5Archive - Unexpected end-of-input: was expecting closing quote for a string value at [Source: {"config": {"name": "model_1", "input_layers": ......, "name; line: 1, column: 20001]
Caught: org.deeplearning4j.nn.modelimport.keras.exceptions.InvalidKerasConfigurationException: Model class name must be Sequential (found Model). For more information, see http://deeplearning4j.org/model-import-keras.
org.deeplearning4j.nn.modelimport.keras.exceptions.InvalidKerasConfigurationException: Model class name must be Sequential (found Model). For more information, see http://deeplearning4j.org/model-import-keras.
        at org.deeplearning4j.nn.modelimport.keras.KerasSequentialModel.<init>(KerasSequentialModel.java:94)
        at org.deeplearning4j.nn.modelimport.keras.KerasSequentialModel.<init>(KerasSequentialModel.java:61)
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasModelBuilder.buildSequential(KerasModelBuilder.java:320)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport.importKerasSequentialModelAndWeights(KerasModelImport.java:195)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport$importKerasSequentialModelAndWeights.call(Unknown Source)
        at dl4j.run(dl4j.groovy:18)
```

VGG16はSequential Modelじゃないからダメでした。
...ってこれ、`importKerasSequentialModelAndWeights`でやってるからですよね。
そりゃそうですか。`importKerasModelAndWeights`に変えて試してみます。

```
$ groovy -cp ~/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2-linux-x86_64.jar dl4j.groovy
[main] INFO org.deeplearning4j.nn.modelimport.keras.Hdf5Archive - Unexpected end-of-input: was expecting closing quote for a string value at [Source: {"config": {"name": "model_1", ......, "name; line: 1, column: 20001]
Caught: java.lang.ExceptionInInitializerError
java.lang.ExceptionInInitializerError
        at org.deeplearning4j.nn.conf.dropout.Dropout.initializeHelper(Dropout.java:107)
        at org.deeplearning4j.nn.conf.dropout.Dropout.<init>(Dropout.java:100)
        at org.deeplearning4j.nn.conf.dropout.Dropout.<init>(Dropout.java:80)
        at org.deeplearning4j.nn.conf.layers.Layer$Builder.dropOut(Layer.java:257)
        at org.deeplearning4j.nn.modelimport.keras.layers.convolutional.KerasConvolution2D.<init>(KerasConvolution2D.java:101)
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasLayerUtils.getKerasLayerFromConfig(KerasLayerUtils.java:226)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.prepareLayers(KerasModel.java:220)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.<init>(KerasModel.java:166)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.<init>(KerasModel.java:98)
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasModelBuilder.buildModel(KerasModelBuilder.java:305)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport.importKerasModelAndWeights(KerasModelImport.java:144)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport$importKerasModelAndWeights.call(Unknown Source)
        at dl4j.run(dl4j.groovy:18)
Caused by: java.lang.RuntimeException: org.nd4j.linalg.factory.Nd4jBackend$NoAvailableBackendException: Please ensure that you have an nd4j backend on your classpath. Please see: http://nd4j.org/getstarted.html
        at org.nd4j.linalg.factory.Nd4j.initContext(Nd4j.java:5449)
        at org.nd4j.linalg.factory.Nd4j.<clinit>(Nd4j.java:213)
        ... 13 more
Caused by: org.nd4j.linalg.factory.Nd4jBackend$NoAvailableBackendException: Please ensure that you have an nd4j backend on your classpath. Please see: http://nd4j.org/getstarted.html
        at org.nd4j.linalg.factory.Nd4jBackend.load(Nd4jBackend.java:213)
        at org.nd4j.linalg.factory.Nd4j.initContext(Nd4j.java:5446)
        ... 14 more
```

何でしょう。今度は[Nd4j](https://deeplearning4j.org/docs/latest/nd4j-overview)のようなので、同じように`~/.groovy/grapes/org.nd4j/nd4j-native/jars/nd4j-native-1.0.0-beta2-linux-x86_64.jar`を落としてきて入れて明示的に指定して試してみても同じだったので、色々試行錯誤の結果、Grapeで指定するのは`nd4j-native-platform`ではなく`nd4j-native`の方だった、ということで一歩進みました。

```
$ groovy -cp ~/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-native/jars/nd4j-native-1.0.0-beta2-linux-x86_64.jar dl4j.groovy
[main] INFO org.deeplearning4j.nn.modelimport.keras.Hdf5Archive - Unexpected end-of-input: was expecting closing quote for a string value at [Source: {"config": {"name": "model_1", ......, "name; line: 1, column: 20001]
[main] INFO org.nd4j.linalg.factory.Nd4jBackend - Loaded [CpuBackend] backend
[main] INFO org.nd4j.nativeblas.NativeOpsHolder - Number of threads used for NativeOps: 2
Caught: java.lang.UnsatisfiedLinkError: no jniopenblas_nolapack in java.library.path
java.lang.UnsatisfiedLinkError: no jniopenblas_nolapack in java.library.path
        at org.bytedeco.javacpp.Loader.loadLibrary(Loader.java:1225)
        at org.bytedeco.javacpp.Loader.load(Loader.java:983)
        at org.bytedeco.javacpp.Loader.load(Loader.java:882)
        at org.bytedeco.javacpp.openblas_nolapack.<clinit>(openblas_nolapack.java:10)
        at org.bytedeco.javacpp.Loader.load(Loader.java:941)
        at org.bytedeco.javacpp.Loader.load(Loader.java:898)
        at org.bytedeco.javacpp.presets.openblas_nolapack.blas_set_num_threads(openblas_nolapack.java:189)
        at org.nd4j.linalg.cpu.nativecpu.blas.CpuBlas.setMaxThreads(CpuBlas.java:136)
        at org.nd4j.nativeblas.Nd4jBlas.<init>(Nd4jBlas.java:52)
        at org.nd4j.linalg.cpu.nativecpu.blas.CpuBlas.<init>(CpuBlas.java:31)
        at org.nd4j.linalg.cpu.nativecpu.CpuNDArrayFactory.createBlas(CpuNDArrayFactory.java:91)
        at org.nd4j.linalg.factory.BaseNDArrayFactory.blas(BaseNDArrayFactory.java:72)
        at org.nd4j.linalg.cpu.nativecpu.ops.NativeOpExecutioner.getEnvironmentInformation(NativeOpExecutioner.java:1236)
        at org.nd4j.linalg.api.ops.executioner.DefaultOpExecutioner.printEnvironmentInformation(DefaultOpExecutioner.java:649)
        at org.nd4j.linalg.factory.Nd4j.initWithBackend(Nd4j.java:5575)
        at org.nd4j.linalg.factory.Nd4j.initContext(Nd4j.java:5447)
        at org.nd4j.linalg.factory.Nd4j.<clinit>(Nd4j.java:213)
        at org.deeplearning4j.nn.conf.dropout.Dropout.initializeHelper(Dropout.java:107)
        at org.deeplearning4j.nn.conf.dropout.Dropout.<init>(Dropout.java:100)
        at org.deeplearning4j.nn.conf.dropout.Dropout.<init>(Dropout.java:80)
        at org.deeplearning4j.nn.conf.layers.Layer$Builder.dropOut(Layer.java:257)
        at org.deeplearning4j.nn.modelimport.keras.layers.convolutional.KerasConvolution2D.<init>(KerasConvolution2D.java:101)
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasLayerUtils.getKerasLayerFromConfig(KerasLayerUtils.java:226)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.prepareLayers(KerasModel.java:220)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.<init>(KerasModel.java:166)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.<init>(KerasModel.java:98)
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasModelBuilder.buildModel(KerasModelBuilder.java:305)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport.importKerasModelAndWeights(KerasModelImport.java:144)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport$importKerasModelAndWeights.call(Unknown Source)
        at dl4j.run(dl4j.groovy:19)
Caused by: java.lang.UnsatisfiedLinkError: no openblas_nolapack in java.library.path
        at org.bytedeco.javacpp.Loader.loadLibrary(Loader.java:1225)
        at org.bytedeco.javacpp.Loader.load(Loader.java:968)
        ... 28 more
```

openblasがない?
`sudo apt install libopenblas-base libopenblas-dev`しても変わりません。
うーん。
色々探すと、[libopenblas_nolapack.so.0: cannot open shared object file: No such file or directory #6132](https://github.com/deeplearning4j/deeplearning4j/issues/6132)という記事があり、
「nd4j-backends」から「mkl-dnn」をexcludeしろ、とあります。
えー! nd4j-backendsはpomだけでjarないんですけど。
`org.bytedeco.javacpp-presets:mkl-dnn`を見てみると、
windowsのjar`mkl-dnn-0.15-1.4.2-windows-x86_64.jar`が落ちてきていました。
それならっていうんで、linux-x86 64版を持ってきて`-cp`で指定します。
ついでに、`org.bytedeco.javacpp-presets:openblas`というのもあって
こっちにもwindows-x86 64版のjarだったので、これについてもlinux-x86 64版を
持ってきてClassPathに入れます。
するとどうでしょう、漸く動き出しました。
けれども、ヌルポで落ちます。

```
$ groovy -cp ~/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-native/jars/nd4j-native-1.0.0-beta2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/mkl-dnn/jars/mkl-dnn-0.15-1.4.2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/openblas/jars/openblas-0.3.0-1.4.2-linux-x86_64.jar dl4j.groovy
[main] INFO org.deeplearning4j.nn.modelimport.keras.Hdf5Archive - Unexpected end-of-input: was expecting closing quote for a string value at [Source: {"config": {"name": "model_1",......, "name; line: 1, column: 20001]
[main] INFO org.nd4j.linalg.factory.Nd4jBackend - Loaded [CpuBackend] backend
[main] INFO org.nd4j.nativeblas.NativeOpsHolder - Number of threads used for NativeOps: 2
[main] INFO org.nd4j.nativeblas.Nd4jBlas - Number of threads used for BLAS: 2
[main] INFO org.nd4j.linalg.api.ops.executioner.DefaultOpExecutioner - Backend used: [CPU]; OS: [Linux]
[main] INFO org.nd4j.linalg.api.ops.executioner.DefaultOpExecutioner - Cores: [4]; Memory: [4.3GB];
[main] INFO org.nd4j.linalg.api.ops.executioner.DefaultOpExecutioner - Blas vendor: [MKL]
{config={lr=9.999999747378752E-5, decay=0.0, momentum=0.8999999761581421, nesterov=false}, class_name=SGD}
Caught: java.lang.NullPointerException
java.lang.NullPointerException
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasOptimizerUtils.mapOptimizer(KerasOptimizerUtils.java:119)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.importTrainingConfiguration(KerasModel.java:253)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.<init>(KerasModel.java:173)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.<init>(KerasModel.java:98)
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasModelBuilder.buildModel(KerasModelBuilder.java:305)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport.importKerasModelAndWeights(KerasModelImport.java:144)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport$importKerasModelAndWeights.call(Unknown Source)
        at dl4j.run(dl4j.groovy:19)
```

これ、119行目って、`KerasOptimizerUtils.java`の[この修正](https://github.com/deeplearning4j/deeplearning4j/commit/2fbc5b757a49c183a7cb7912b5b5e16644c9b798#diff-603a0324b05908c959af3f5cfd04bf44)の前のこれですね。

```
119: double momentum = (double) optimizerParameters.get("epsilon");
```

`epsilon`じゃねーよ`momentum`だよー今は。
ここは8.17.2018に`try-catch`で囲まれて、現在のversionには反映されてないので、
うー、[commentしました](https://github.com/deeplearning4j/deeplearning4j/issues/6385#issuecomment-421356713)。
でもまー、keras変わっちゃうと、DL4Jがついていくのも大変なのかもですね。
けどこれで現状、kerasのmodelのimportは、
optimizerにSGDを使ったものについては全滅です。
実に下らない理由(=parameter nameの間違い)で。

あまりにも悔しいので、[Java Bytecode Editor: JByteMod](https://grax.info/)を使ってbytecodeを書き換えてみました(正確には[Col-E/Recaf](https://github.com/Col-E/Recaf)、[JBE - Java Bytecode Editor](http://set.ee/jbe/)と3つ試してみて、JByteModだけでうまく書き換えられました)。
`deeplearning4j-modelimport-1.0.0-beta2.jar`をjarごと読み込めて、
`org.deeplearning4j.nn.modelimport.keras.utils.KerasOptimizerUtils`に
降りていって、
EditorのDecompilerで見事にsource codeが出て来ます。
こちらで変えたくなりますが、ここはview onlyの模様。
EditorのCodeで見られるbytecodeならいじれるので、
`line 119`近辺の`ldc String "epsilon"`に当たりをつけ、
それを`momentum`に変えて保存すると...
うまく行きました!→次のエラーに進みました。orz

```
[main] INFO org.nd4j.linalg.factory.Nd4jBackend - Loaded [CpuBackend] backend
[main] INFO org.nd4j.nativeblas.NativeOpsHolder - Number of threads used for NativeOps: 2
[main] INFO org.nd4j.nativeblas.Nd4jBlas - Number of threads used for BLAS: 2
[main] INFO org.nd4j.linalg.api.ops.executioner.DefaultOpExecutioner - Backend used: [CPU]; OS: [Linux]
[main] INFO org.nd4j.linalg.api.ops.executioner.DefaultOpExecutioner - Cores: [4]; Memory: [4.3GB];
[main] INFO org.nd4j.linalg.api.ops.executioner.DefaultOpExecutioner - Blas vendor: [MKL]
{config={lr=9.999999747378752E-5, decay=0.0, momentum=0.8999999761581421, nesterov=false}, class_name=SGD}
Caught: org.deeplearning4j.exception.DL4JInvalidConfigException: Invalid configuration for layer (idx=-1, name=block2_conv1, type=ConvolutionLayer) for width dimension:  Invalid input configuration for kernel width. Require 0 < kW <= inWidth + 2*padW; got (kW=3, inWidth=1, padW=0)
Input type = InputTypeConvolutional(h=112,w=1,c=64), kernel = [3, 3], strides = [1, 1], padding = [0, 0], layer size (output channels) = 128, convolution mode = Same
org.deeplearning4j.exception.DL4JInvalidConfigException: Invalid configuration for layer (idx=-1, name=block2_conv1, type=ConvolutionLayer) for width dimension:  Invalid input configuration for kernel width. Require 0 < kW <= inWidth + 2*padW; got (kW=3, inWidth=1, padW=0)
Input type = InputTypeConvolutional(h=112,w=1,c=64), kernel = [3, 3], strides = [1, 1], padding = [0, 0], layer size (output channels) = 128, convolution mode = Same
        at org.deeplearning4j.nn.conf.layers.InputTypeUtil.getOutputTypeCnnLayers(InputTypeUtil.java:349)
        at org.deeplearning4j.nn.conf.layers.ConvolutionLayer.getOutputType(ConvolutionLayer.java:181)
        at org.deeplearning4j.nn.modelimport.keras.layers.convolutional.KerasConvolution2D.getOutputType(KerasConvolution2D.java:150)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.inferOutputTypes(KerasModel.java:306)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.<init>(KerasModel.java:181)
        at org.deeplearning4j.nn.modelimport.keras.KerasModel.<init>(KerasModel.java:98)
        at org.deeplearning4j.nn.modelimport.keras.utils.KerasModelBuilder.buildModel(KerasModelBuilder.java:305)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport.importKerasModelAndWeights(KerasModelImport.java:144)
        at org.deeplearning4j.nn.modelimport.keras.KerasModelImport$importKerasModelAndWeights.call(Unknown Source)
        at tt.run(tt.groovy:19)
```

`Require 0 < kW <= inWidth + 2*padW; got (kW=3, inWidth=1, padW=0)`ということからすると、多分keras modelの方の`block2_conv1`の`padW`が1でないとダメ?
何をどうすればいいんでしょう?

心が折れそうになりましたが、もう少し頑張ってみます。

まずVGG16の`block2_conv1`のpadding(`padW`って恐らくこれでしょう?)は`"same"`のようです([VGG16をkerasで実装した](http://blog.neko-ni-naritai.com/entry/2018/04/07/115504))。
実際、途中のlogでは下のように`"padding": "same"`とあります。

```
{"name": "block2_conv1", "config": {"kernel_regularizer": null, "activation": "relu", "dilation_rate": [1, 1], "name": "block2_conv1", "data_format": "channels_last", "kernel_size": [3, 3], "bias_constraint": null, "filters": 128, "kernel_initializer": {"config": {"scale": 1.0, "mode": "fan_avg", "distribution": "uniform", "seed": null}, "class_name": "VarianceScaling"}, "use_bias": true, "kernel_constraint": null, "activity_regularizer": null, "trainable": false, "padding": "same", ...
```

色々見ると、VGG16では`padding=[1,1]`みたいです([事前学習済み VGG-16 畳み込みニューラル ネットワーク](https://jp.mathworks.com/help/deeplearning/ref/vgg16.html))
([Build VGG16 from scratch : part II](http://agnesmustar.com/2017/05/25/build-vgg16-scratch-part-ii/))
([Reading the VGG Network Paper and Implementing It From Scratch with Keras](https://hackernoon.com/learning-keras-by-implementing-vgg16-from-scratch-d036733f2d5))。
`"same"`が0と解釈された??
いやいや、DL4Jのsourceを見ると、`"same"`はきちんと扱っているようです。
あー、`SAME`ってこういう意味だったんですね([【Python】 KerasのConv2Dの引数paddingについて](http://ni4muraano.hatenablog.com/entry/2017/02/02/195505))。
今回のExceptionは、具体的には
`deeplearning4j/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/InputTypeUtil.java`の以下で出ていることがわかりました。

```
if (kW <= 0 || kW > inWidth + 2 * padW) {
    throw new DL4JInvalidConfigException(getConfigErrorCommonLine(layerIdx, layerName, layerClass, false)
                    + " Invalid input configuration for kernel width. Require 0 < kW <= inWidth + 2*padW; got (kW="
                    + kW + ", inWidth=" + inWidth + ", padW=" + padW + ")\n" + getConfigErrorCommonLastLine(
                                    inputType, kernelSize, stride, padding, outputDepth, convolutionMode));
}
```

`padW`って何?

```
int padH = (padding == null ? 0 : padding[0]); //May be null for ConvolutionMode.Same
int padW = (padding == null ? 0 : padding[1]);
```

`padding`は`getOutputTypeCnnLayers`の引数です。
その元は`deeplearning4j/deeplearning4j/deeplearning4j-nn/src/main/java/org/deeplearning4j/nn/conf/layers/ConvolutionLayer.java`の↓から来ていて、

```
@Override
public InputType getOutputType(int layerIndex, InputType inputType) {
    if (inputType == null || inputType.getType() != InputType.Type.CNN) {
        throw new IllegalStateException("Invalid input for Convolution layer (layer name=\"" + getLayerName()
                + "\"): Expected CNN input, got " + inputType);
    }

    return InputTypeUtil.getOutputTypeCnnLayers(inputType, kernelSize, stride, padding, dilation,
            convolutionMode, nOut, layerIndex, getLayerName(), ConvolutionLayer.class);
}
```

この`padding`はconstructorから来ていて。

```
this.padding = builder.padding;
```

`builder`はinner classでその`padding`は外から来ているか
又はdefaultなら`new int[]{0, 0}`と。
うーーー。
似ている[issue](https://github.com/deeplearning4j/deeplearning4j/issues/6218)もありましたが、input shapeの明示的指定では解決しなかったので、
もう[issue](https://github.com/deeplearning4j/deeplearning4j/issues/6444)上げました。

試すのは、Groovyだと↓で済むんですけど、

```
$ cat dl4j.groovy
@Grab('org.deeplearning4j:deeplearning4j-core:1.0.0-beta2')
@Grab('org.deeplearning4j:deeplearning4j-modelimport:1.0.0-beta2')
@Grab('org.nd4j:nd4j-native:1.0.0-beta2')
@Grab('org.slf4j:slf4j-simple')
@Grab('org.bytedeco.javacpp-presets:hdf5:1.10.2-1.4.2')
@Grab('org.bytedeco:javacpp:1.4.2')

import org.deeplearning4j.nn.modelimport.keras.KerasModelImport

// model = KerasModelImport.importKerasModelAndWeights('my_model.hdf5', [224, 224, 3] as int[], false)
model = KerasModelImport.importKerasModelAndWeights('my_model.hdf5')

$ groovy -cp ~/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-native/jars/nd4j-native-1.0.0-beta2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/mkl-dnn/jars/mkl-dnn-0.15-1.4.2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/openblas/jars/openblas-0.3.2-1.4.2-linux-x86_64.jar dl4j.groovy
```

[本家のissue](https://github.com/deeplearning4j/deeplearning4j/issues)で聞くからには、
Groovyだからダメなんじゃないの? とか言われそうだったので、
Javaにしました。
classpathの指定が大変でした。
不要なjarも入っているかもです。

```
$ cat DL4J.java
class DL4J {
    public static void main(String... args) throws Exception {
        org.deeplearning4j.nn.modelimport.keras.KerasModelImport
            .importKerasModelAndWeights("my_model.hdf5",
                                        new int[]{224, 224, 3}, false);
            // .importKerasModelAndWeights("my_model.hdf5"); // failed too
    }
}
$ javac -cp ~/.groovy/grapes/org.deeplearning4j/deeplearning4j-modelimport/jars/deeplearning4j-modelimport-1.0.0-beta2.jar:/usr/share/java/slf4j-simple.jar:/home/u-ryo/.groovy/grapes/org.deeplearning4j/deeplearning4j-nn/jars/deeplearning4j-nn-1.0.0-beta2.jar DL4J.java
$ java -cp ~/.groovy/grapes/org.deeplearning4j/deeplearning4j-modelimport/jars/deeplearning4j-modelimport-1.0.0-beta2.jar:/usr/share/java/slf4j-simple.jar:/home/u-ryo/.groovy/grapes/org.deeplearning4j/deeplearning4j-nn/jars/deeplearning4j-nn-1.0.0-beta2.jar:/usr/share/java/slf4j-api.jar:.:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2.jar:/home/u-ryo/.groovy/grapes/org.bytedeco/javacpp/jars/javacpp-1.4.2.jar:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-jackson/jars/nd4j-jackson-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.nd4j/jackson/jars/jackson-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-common/jars/nd4j-common-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-api/jars/nd4j-api-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-context/jars/nd4j-context-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-buffer/jars/nd4j-buffer-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-native/jars/nd4j-native-1.0.0-beta2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-native-platform/jars/nd4j-native-platform-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-native/jars/nd4j-native-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.nd4j/nd4j-native-api/jars/nd4j-native-api-1.0.0-beta2.jar:/home/u-ryo/.groovy/grapes/org.apache.commons/commons-math3/jars/commons-math3-3.5.jar:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/openblas/jars/openblas-0.3.0-1.4.2-linux-x86_64.jar:/home/u-ryo/.groovy/grapes/org.bytedeco/javacpp/jars/javacpp-1.4.2.jar:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/hdf5/jars/hdf5-1.10.2-1.4.2.jar:/home/u-ryo/.groovy/grapes/org.bytedeco.javacpp-presets/openblas/jars/openblas-0.3.0-1.4.2.jar DL4J
```
