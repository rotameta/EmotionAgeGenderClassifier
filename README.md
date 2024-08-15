Tabii! İşte metnin Türkçeye çevirisi:

---

# face-api.js

## Eğitimler

* **[face-api.js — Tarayıcıda tensorflow.js ile Yüz Tanıma İçin JavaScript API'si](https://itnext.io/face-api-js-javascript-api-for-face-recognition-in-the-browser-with-tensorflow-js-bcc2a6c4cf07)**
* **[Realtime JavaScript Yüz Takibi ve Yüz Tanıma: face-api.js'in MTCNN Yüz Dedektörü Kullanarak](https://itnext.io/realtime-javascript-face-tracking-and-face-recognition-using-face-api-js-mtcnn-face-detector-d924dd8b5740)**
* **[Gerçek Zamanlı Webcam Yüz Tespiti ve Duygu Tanıma - Video](https://youtu.be/CVClHLwv-4I)**
* **[JavaScript ile Kolay Yüz Tanıma Eğitimi - Video](https://youtu.be/AZ4PdALMqx0)**
* **[face-api.js'i Vue.js ve Electron ile Kullanma](https://medium.com/@andreas.schallwig/do-not-laugh-a-simple-ai-powered-game-3e22ad0f8166)**
* **[İnsanlara Maske Ekleme - Gant Laborde Jason ile Öğrenin](https://www.learnwithjason.dev/fun-with-machine-learning-pt-2)**

## İçindekiler

* **[Özellikler](#features)**
* **[Örnekleri Çalıştırma](#running-the-examples)**
* **[Tarayıcı için face-api.js](#face-api.js-for-the-browser)**
* **[Nodejs için face-api.js](#face-api.js-for-nodejs)**
* **[Kullanım](#getting-started)**
  * **[Modelleri Yükleme](#getting-started-loading-models)**
  * **[Yüksek Seviyeli API](#high-level-api)**
  * **[Tespit Sonuçlarını Gösterme](#getting-started-displaying-detection-results)**
  * **[Yüz Tespiti Seçenekleri](#getting-started-face-detection-options)**
  * **[Yardımcı Sınıflar](#getting-started-utility-classes)**
  * **[Diğer Yararlı Yardımcı](#other-useful-utility)**
* **[Mevcut Modeller](#models)**
  * **[Yüz Tespiti](#models-face-detection)**
  * **[Yüz Landmark Tespiti](#models-face-landmark-detection)**
  * **[Yüz Tanıma](#models-face-recognition)**
  * **[Yüz İfade Tanıma](#models-face-expression-recognition)**
  * **[Yaş Tahmini ve Cinsiyet Tanıma](#models-age-and-gender-recognition)**
* **[API Belgeleri](https://justadudewhohacks.github.io/face-api.js/docs/globals.html)**

# Çalıştırma

Depoyu klonlayın:

```bash
git clone https://github.com/justadudewhohacks/face-api.js.git
```

## Tarayıcı Örneklerini Çalıştırma

```bash
cd face-api.js/examples/examples-browser
npm i
npm start
```

http://localhost:3000/ adresine göz atın.

## Nodejs Örneklerini Çalıştırma

```bash
cd face-api.js/examples/examples-nodejs
npm i
```

Şimdi ts-node kullanarak örneklerden birini çalıştırın:

```bash
ts-node faceDetection.ts
```

Veya basitçe derleyin ve node ile çalıştırın:

```bash
tsc faceDetection.ts
node faceDetection.js
```

<a name="face-api.js-for-the-browser"></a>

# Tarayıcı için face-api.js

En son scripti [dist/face-api.js](https://github.com/justadudewhohacks/face-api.js/tree/master/dist) dosyasından dahil edin.

Veya npm ile yükleyin:

```bash
npm i face-api.js
```

<a name="face-api.js-for-nodejs"></a>

# Nodejs için face-api.js

Bazı tarayıcı spesifiklerini (HTMLImageElement, HTMLCanvasElement ve ImageData gibi) polyfill ederek nodejs ortamında eşdeğer API'yi kullanabiliriz. Bunu yapmanın en kolay yolu, node-canvas paketini yüklemektir.

Alternatif olarak, görüntü verilerinden kendi tensörlerinizi oluşturabilir ve API'ye girdiler olarak geçebilirsiniz.

Ayrıca, @tensorflow/tfjs-node paketini (gereklilik değil, ama şiddetle tavsiye edilir) yüklemek istersiniz; bu, şeyleri yerel Tensorflow C++ kütüphanesine derleyip bağlayarak büyük ölçüde hızlandırır:

```bash
npm i face-api.js canvas @tensorflow/tfjs-node
```

Şimdi çevreyi polyfill'lerle yamalayalım:

```javascript
// yerel tensorflow'a bağlanan nodejs eklemelerini içe aktarın,
// gerekli değil, ama şeyleri büyük ölçüde hızlandıracaktır (python gerekli)
import '@tensorflow/tfjs-node';

// HTMLCanvasElement, HTMLImageElement, ImageData için nodejs sarmalayıcılarını uygular
import * as canvas from 'canvas';

import * as faceapi from 'face-api.js';

// nodejs ortamını yamalayın, HTMLCanvasElement ve HTMLImageElement uygulaması sağlamamız gerekiyor
const { Canvas, Image, ImageData } = canvas;
faceapi.env.monkeyPatch({ Canvas, Image, ImageData });
```

<a name="getting-started"></a>

# Başlarken

<a name="getting-started-loading-models"></a>

## Modellerin Yüklenmesi

Tüm küresel sinir ağı örnekleri faceapi.nets aracılığıyla dışa aktarılır:

```javascript
console.log(faceapi.nets)
// ageGenderNet
// faceExpressionNet
// faceLandmark68Net
// faceLandmark68TinyNet
// faceRecognitionNet
// ssdMobilenetv1
// tinyFaceDetector
// tinyYolov2
```

Bir modeli yüklemek için, ilgili manifest.json dosyasını ve model ağırlık dosyalarını (parçaları) varlıklar olarak sağlamanız gerekir. Bunları basitçe public veya assets klasörünüze kopyalayın. Bir modelin manifest.json ve parça dosyaları aynı dizinde bulunmalı / aynı rota altında erişilebilir olmalıdır.

Varsayalım modeller **public/models** içinde bulunuyor:

```javascript
await faceapi.nets.ssdMobilenetv1.loadFromUri('/models')
// diğer modeller için de geçerli:
// await faceapi.nets.faceLandmark68Net.loadFromUri('/models')
// await faceapi.nets.faceRecognitionNet.loadFromUri('/models')
// ...
```

Bir nodejs ortamında modelleri doğrudan diskten yükleyebilirsiniz:

```javascript
await faceapi.nets.ssdMobilenetv1.loadFromDisk('./models')
```

Ayrıca modeli bir tf.NamedTensorMap'ten de yükleyebilirsiniz:

```javascript
await faceapi.nets.ssdMobilenetv1.loadFromWeightMap(weightMap)
```

Alternatif olarak, sinir ağlarının kendi örneklerini oluşturabilirsiniz:

```javascript
const net = new faceapi.SsdMobilenetv1();
await net.loadFromUri('/models');
```

Ağırlıkları bir Float32Array olarak da yükleyebilirsiniz (sıkıştırılmamış modelleri kullanmak isterseniz):

```javascript
// fetch kullanarak
net.load(await faceapi.fetchNetWeights('/models/face_detection_model.weights'))

// axios kullanarak
const res = await axios.get('/models/face_detection_model.weights', { responseType: 'arraybuffer' });
const weights = new Float32Array(res.data);
net.load(weights);
```

<a name="getting-high-level-api"></a>

## Yüksek Seviyeli API

Aşağıda **input** bir HTML img, video veya canvas elementi ya da bu elementin kimliği olabilir.

```html
<img id="myImg" src="images/example.png" />
<video id="myVideo" src="media/example.mp4" />
<canvas id="

myCanvas" />
```

```javascript
const input = document.getElementById('myImg');
// const input = document.getElementById('myVideo')
// const input = document.getElementById('myCanvas')
// veya basitçe:
// const input = 'myImg'
```

### Yüzleri Tespit Etme

Bir görüntüdeki tüm yüzleri tespit edin. **Array<[FaceDetection](#interface-face-detection)>** döndürür:

```javascript
const detections = await faceapi.detectAllFaces(input);
```

Bir görüntüde en yüksek güven puanına sahip yüzü tespit edin. **[FaceDetection](#interface-face-detection) | undefined** döndürür:

```javascript
const detection = await faceapi.detectSingleFace(input);
```

Varsayılan olarak **detectAllFaces** ve **detectSingleFace**, SSD Mobilenet V1 Yüz Dedektörünü kullanır. Yüz dedektörünü, ilgili seçenekler nesnesini geçirerek belirtebilirsiniz:

```javascript
const detections1 = await faceapi.detectAllFaces(input, new faceapi.SsdMobilenetv1Options());
const detections2 = await faceapi.detectAllFaces(input, new faceapi.TinyFaceDetectorOptions());
```

Her yüz dedektörünün seçeneklerini [burada](#getting-started-face-detection-options) gösterildiği gibi ayarlayabilirsiniz.

### 68 Yüz Landmark Noktasını Tespit Etme

**Yüz tespitinden sonra, her tespit edilen yüz için yüz landmark noktaları tahmin edilebilir:**

Bir görüntüdeki tüm yüzleri tespit edin + her tespit edilen yüz için 68 Noktalı Yüz Landmarklarını hesaplayın. **Array<[WithFaceLandmarks<WithFaceDetection<{}>>](#getting-started-utility-classes)>** döndürür:

```javascript
const detectionsWithLandmarks = await faceapi.detectAllFaces(input).withFaceLandmarks();
```

Bir görüntüde en yüksek güven puanına sahip yüzü tespit edin + bu yüz için 68 Noktalı Yüz Landmarklarını hesaplayın. **[WithFaceLandmarks<WithFaceDetection<{}>>](#getting-started-utility-classes) | undefined** döndürür:

```javascript
const detectionWithLandmarks = await faceapi.detectSingleFace(input).withFaceLandmarks();
```

Varsayılan model yerine küçük modeli kullanmayı da belirtebilirsiniz:

```javascript
const useTinyModel = true;
const detectionsWithLandmarks = await faceapi.detectAllFaces(input).withFaceLandmarks(useTinyModel);
```

### Yüz Tanımlayıcılarını Hesaplama

**Yüz tespiti ve yüz landmark tahmininden sonra, her yüz için yüz tanımlayıcıları şu şekilde hesaplanabilir:**

Bir görüntüdeki tüm yüzleri tespit edin + her tespit edilen yüz için 68 Noktalı Yüz Landmarklarını hesaplayın. **Array<[WithFaceDescriptor<WithFaceLandmarks<WithFaceDetection<{}>>>](#getting-started-utility-classes)>** döndürür:

```javascript
const results = await faceapi.detectAllFaces(input).withFaceLandmarks().withFaceDescriptors();
```

Bir görüntüde en yüksek güven puanına sahip yüzü tespit edin + bu yüz için 68 Noktalı Yüz Landmarklarını ve yüz tanımlayıcısını hesaplayın. **[WithFaceDescriptor<WithFaceLandmarks<WithFaceDetection<{}>>>](#getting-started-utility-classes) | undefined** döndürür:

```javascript
const result = await faceapi.detectSingleFace(input).withFaceLandmarks().withFaceDescriptor();
```

### Yüz İfadelerini Tanıma

**Yüz ifadelerini tanıma, tespit edilen yüzler için şu şekilde yapılabilir:**

Bir görüntüdeki tüm yüzleri tespit edin + her yüzün yüz ifadelerini tanıyın. **Array<[WithFaceExpressions<WithFaceLandmarks<WithFaceDetection<{}>>>](#getting-started-utility-classes)>** döndürür:

```javascript
const detectionsWithExpressions = await faceapi.detectAllFaces(input).withFaceLandmarks().withFaceExpressions();
```

Bir görüntüde en yüksek güven puanına sahip yüzü tespit edin + bu yüzün yüz ifadelerini tanıyın. **[WithFaceExpressions<WithFaceLandmarks<WithFaceDetection<{}>>>](#getting-started-utility-classes) | undefined** döndürür:

```javascript
const detectionWithExpressions = await faceapi.detectSingleFace(input).withFaceLandmarks().withFaceExpressions();
```

**.withFaceLandmarks() adımını atlayabilirsiniz, bu yüz hizalama adımını atlar (daha az kararlı doğruluk):**

Yüz hizalaması olmadan tüm yüzleri tespit edin + her yüzün yüz ifadelerini tanıyın. **Array<[WithFaceExpressions<WithFaceDetection<{}>>](#getting-started-utility-classes)>** döndürür:

```javascript
const detectionsWithExpressions = await faceapi.detectAllFaces(input).withFaceExpressions();
```

Yüz hizalaması olmadan en yüksek güven puanına sahip yüzü tespit edin + bu yüzün yüz ifadesini tanıyın. **[WithFaceExpressions<WithFaceDetection<{}>>](#getting-started-utility-classes) | undefined** döndürür:

```javascript
const detectionWithExpressions = await faceapi.detectSingleFace(input).withFaceExpressions();
```

### Yaş Tahmini ve Cinsiyet Tanıma

**Yüz tespitinden sonra yaş tahmini ve cinsiyet tanıma şu şekilde yapılabilir:**

Bir görüntüdeki tüm yüzleri tespit edin + her yüzün yaşını tahmin edin ve cinsiyetini tanıyın. **Array<[WithAge<WithGender<WithFaceLandmarks<WithFaceDetection<{}>>>>](#getting-started-utility-classes)>** döndürür:

```javascript
const detectionsWithAgeAndGender = await faceapi.detectAllFaces(input).withFaceLandmarks().withAgeAndGender();
```

Bir görüntüde en yüksek güven puanına sahip yüzü tespit edin + bu yüzün yaşını tahmin edin ve cinsiyetini tanıyın. **[WithAge<WithGender<WithFaceLandmarks<WithFaceDetection<{}>>>>](#getting-started-utility-classes) | undefined** döndürür:

```javascript
const detectionWithAgeAndGender = await faceapi.detectSingleFace(input).withFaceLandmarks().withAgeAndGender();
```

**.withFaceLandmarks() adımını atlayabilirsiniz, bu yüz hizalama adımını atlar (daha az kararlı doğruluk):**

Yüz hizalaması olmadan tüm yüzleri tespit edin + her yüzün yaşını tahmin edin ve cinsiyetini tanıyın. **Array<[WithAge<WithGender<WithFaceDetection<{}>>>](#getting-started-utility-classes)>** döndürür:

```javascript
const detectionsWithAgeAndGender = await faceapi.detectAllFaces(input).withAgeAndGender();
```

Yüz hizalaması olmadan en yüksek güven puanına sahip yüzü tespit edin + bu yüzün yaşını tahmin edin ve cinsiyetini tanıyın. **[WithAge<WithGender<WithFaceDetection<{}>>>](#getting-started-utility-classes) | undefined** döndürür:

```javascript
const detectionWithAgeAndGender = await faceapi.detectSingleFace(input).withAgeAndGender();
```

### Görevlerin Bileşimi

**Görevler şu şekilde birleştirilebilir:**

```javascript
// tüm yüzler
await faceapi.detectAllFaces(input);
await faceapi.detectAllFaces(input).withFaceExpressions();
await faceapi.detectAllFaces(input).withFaceLandmarks();
await faceapi.detectAllFaces(input).withFaceLandmarks().withFaceExpressions();
await faceapi.detectAllFaces(input).withFaceLandmarks().withFaceExpressions().withFaceDescriptors();
await faceapi.detectAllFaces(input).withFaceLandmarks().withAgeAndGender().withFaceDescriptors();
await faceapi.detectAllFaces(input).withFaceLandmarks().withFaceExpressions().withAgeAndGender().withFaceDescriptors();

// tek yüz
await faceapi.detectSingleFace(input);
await faceapi.detectSingleFace(input).withFaceExpressions();
await faceapi.detectSingleFace(input).withFaceLandmarks();
await faceapi.detectSingleFace(input).withFaceLandmarks().withFaceExpressions();
await faceapi.detectSingleFace(input).withFaceLandmarks().withFaceExpressions().withFaceDescriptor();
await faceapi.detectSingleFace(input).withFaceLandmarks().withAgeAndGender().withFaceDescriptor();
await faceapi.detectSingleFace(input).withFaceLandmarks().withFaceExpressions().withAgeAndGender().withFaceDescriptor();
```

### Tanımlayıcıları Eşleştirerek Yüz Tanıma

Yüz tanıma yapmak için, referans yüz tanımlayıcılarını sorgu yüz tanımlayıcılarıyla karşılaştırmak için faceapi.FaceMatcher'ı kullanabilirsiniz.

Öncelikle, referans verilerle FaceMatcher'ı başlatıyoruz. Örneğin, basitçe bir **referenceImage**'de yüzleri tespit edebilir ve referans görüntüsü için tespit sonuçlarından otomatik olarak atanmış etiketlerle oluşturulan FaceMatcher ile sonuçları eşleştirerek **queryImage1**'de gösterilen kişinin yüzünü tanıyabilirsiniz:

```javascript
const results = await faceapi
  .detectAllFaces(referenceImage

)
  .withFaceLandmarks()
  .withFaceDescriptors();

if (!results.length) {
  return;
}

// tespit sonuçlarından otomatik olarak atanmış etiketlerle FaceMatcher oluşturun
const faceMatcher = new faceapi.FaceMatcher(results);
```

Şimdi **queryImage1**'de gösterilen kişinin yüzünü tanıyabiliriz:

```javascript
const singleResult = await faceapi
  .detectSingleFace(queryImage1)
  .withFaceLandmarks()
  .withFaceDescriptor();

if (singleResult) {
  const bestMatch = faceMatcher.findBestMatch(singleResult.descriptor);
  console.log(bestMatch.toString());
}
```

Veya **queryImage2**'de gösterilen tüm yüzleri tanıyabiliriz:

```javascript
const results = await faceapi
  .detectAllFaces(queryImage2)
  .withFaceLandmarks()
  .withFaceDescriptors();

results.forEach(fd => {
  const bestMatch = faceMatcher.findBestMatch(fd.descriptor);
  console.log(bestMatch.toString());
});
```

Ayrıca şu şekilde etiketli referans tanımlayıcıları oluşturabilirsiniz:

```javascript
const labeledDescriptors = [
  new faceapi.LabeledFaceDescriptors(
    'obama',
    [descriptorObama1, descriptorObama2]
  ),
  new faceapi.LabeledFaceDescriptors(
    'trump',
    [descriptorTrump]
  )
];

const faceMatcher = new faceapi.FaceMatcher(labeledDescriptors);
```

<a name="getting-started-displaying-detection-results"></a>

## Tespit Sonuçlarını Gösterme

Kaplama kanvasını hazırlayın:

```javascript
const displaySize = { width: input.width, height: input.height };
// kaplama kanvasını giriş boyutlarına yeniden boyutlandırın
const canvas = document.getElementById('overlay');
faceapi.matchDimensions(canvas, displaySize);
```

face-api.js, kullanabileceğiniz bazı yüksek seviyeli çizim fonksiyonları tanımlar:

```javascript
/* Tespit edilen yüz sınırlayıcı kutuları göster */
const detections = await faceapi.detectAllFaces(input);
// görüntünüz orijinalden farklı bir boyuta sahipse, tespit edilen kutuları yeniden boyutlandırın
const resizedDetections = faceapi.resizeResults(detections, displaySize);
// tespitleri kanvasa çizin
faceapi.draw.drawDetections(canvas, resizedDetections);

/* Yüz landmarklarını göster */
const detectionsWithLandmarks = await faceapi
  .detectAllFaces(input)
  .withFaceLandmarks();
// görüntünüz orijinalden farklı bir boyuta sahipse, tespit edilen kutuları ve landmarkları yeniden boyutlandırın
const resizedResults = faceapi.resizeResults(detectionsWithLandmarks, displaySize);
// tespitleri kanvasa çizin
faceapi.draw.drawDetections(canvas, resizedResults);
// landmarkları kanvasa çizin
faceapi.draw.drawFaceLandmarks(canvas, resizedResults);

/* Yüz ifade sonuçlarını göster */
const detectionsWithExpressions = await faceapi
  .detectAllFaces(input)
  .withFaceLandmarks()
  .withFaceExpressions();
// görüntünüz orijinalden farklı bir boyuta sahipse, tespit edilen kutuları ve landmarkları yeniden boyutlandırın
const resizedResults = faceapi.resizeResults(detectionsWithExpressions, displaySize);
// tespitleri kanvasa çizin
faceapi.draw.drawDetections(canvas, resizedResults);
// minimum olasılıkla yüz ifadelerini gösteren bir metin kutusu çizin
const minProbability = 0.05;
faceapi.draw.drawFaceExpressions(canvas, resizedResults, minProbability);
```

Ayrıca özel metin içeren kutular çizebilirsiniz ([DrawBox](https://github.com/justadudewhohacks/tfjs-image-recognition-base/blob/master/src/draw/DrawBox.ts)):

```javascript
const box = { x: 50, y: 50, width: 100, height: 100 };
// Aşağıdaki DrawBoxOptions'a bakın
const drawOptions = {
  label: 'Merhaba, ben bir kutuyum!',
  lineWidth: 2
};
const drawBox = new faceapi.draw.DrawBox(box, drawOptions);
drawBox.draw(document.getElementById('myCanvas'));
```

DrawBox çizim seçenekleri:

```javascript
export interface IDrawBoxOptions {
  boxColor?: string;
  lineWidth?: number;
  drawLabelOptions?: IDrawTextFieldOptions;
  label?: string;
}
```

Son olarak, özel metin alanları çizebilirsiniz ([DrawTextField](https://github.com/justadudewhohacks/tfjs-image-recognition-base/blob/master/src/draw/DrawTextField.ts)):

```javascript
const text = [
  'Bu bir metin satırı!',
  'Bu başka bir metin satırı!'
];
const anchor = { x: 200, y: 200 };
// Aşağıdaki DrawTextField'a bakın
const drawOptions = {
  anchorPosition: 'TOP_LEFT',
  backgroundColor: 'rgba(0, 0, 0, 0.5)'
};
const drawBox = new faceapi.draw.DrawTextField(text, anchor, drawOptions);
drawBox.draw(document.getElementById('myCanvas'));
```

DrawTextField çizim seçenekleri:

```javascript
export interface IDrawTextFieldOptions {
  anchorPosition?: AnchorPosition;
  backgroundColor?: string;
  fontColor?: string;
  fontSize?: number;
  fontStyle?: string;
  padding?: number;
}

export enum AnchorPosition {
  TOP_LEFT = 'TOP_LEFT',
  TOP_RIGHT = 'TOP_RIGHT',
  BOTTOM_LEFT = 'BOTTOM_LEFT',
  BOTTOM_RIGHT = 'BOTTOM_RIGHT'
}
```

<a name="getting-started-face-detection-options"></a>

## Yüz Tespiti Seçenekleri

### SsdMobilenetv1Options

```javascript
export interface ISsdMobilenetv1Options {
  // minimum güven eşiği
  // varsayılan: 0.5
  minConfidence?: number;

  // döndürülecek maksimum yüz sayısı
  // varsayılan: 100
  maxResults?: number;
}

// örnek
const options = new faceapi.SsdMobilenetv1Options({ minConfidence: 0.8 });
```

### TinyFaceDetectorOptions

```javascript
export interface ITinyFaceDetectorOptions {
  // görüntünün işlendiği boyut, ne kadar küçükse o kadar hızlı,
  // ancak küçük yüzleri tespit etmekte daha az hassas, 32'nin katı olmalı,
  // yaygın boyutlar 128, 160, 224, 320, 416, 512, 608'dir,
  // webcam üzerinden yüz takibi için daha küçük boyutlar kullanmanızı tavsiye ederim,
  // örn. 128, 160, küçük yüzleri tespit etmek için daha büyük boyutlar kullanın, örn. 512, 608
  // varsayılan: 416
  inputSize?: number;

  // minimum güven eşiği
  // varsayılan: 0.5
  scoreThreshold?: number;
}

// örnek
const options = new faceapi.TinyFaceDetectorOptions({ inputSize: 320 });
```

<a name="getting-started-utility-classes"></a>

## Yardımcı Sınıflar

### IBox

```javascript
export interface IBox {
  x: number;
  y: number;
  width: number;
  height: number;
}
```

### IFaceDetection

```javascript
export interface IFaceDetection {
  score: number;
  box: Box;
}
```

### IFaceLandmarks

```javascript
export interface IFaceLandmarks {
  positions: Point[];
  shift: Point;
}
```

### WithFaceDetection

```javascript
export type WithFaceDetection<TSource> = TSource & {
  detection: FaceDetection;
};
```

### WithFaceLandmarks

```javascript
export type WithFaceLandmarks<TSource> = TSource & {
  unshiftedLandmarks: FaceLandmarks;
  landmarks: FaceLandmarks;
  alignedRect: FaceDetection;
};
```

### WithFaceDescriptor

```javascript
export type WithFaceDescriptor<TSource> = TSource & {
  descriptor: Float32Array;
};
```

### WithFaceExpressions

```javascript
export type WithFaceExpressions<TSource> = TSource & {
  expressions: FaceExpressions;
};
```

### WithAge

```javascript
export type WithAge<TSource> = TSource & {
  age: number;
};
```

### WithGender

```javascript
export type WithGender<TSource> = TSource & {
  gender: Gender;
  genderProbability: number;
};

export enum Gender {
  FEMALE = 'female',
  MALE = 'male'
}
```

<a name="getting-started-other-useful-utility"></a>

## Diğer Yararlı Yardımcı

### Düşük Seviyeli API Kullanımı

Yüksek seviyeli API'yi kullanmak yerine, her sinir ağının ileri yöntemlerini doğrudan kullanabilirsiniz:

```javascript
const detections1 = await faceapi.ssdMobilenetv1(input, options);
const detections2 = await faceapi.tinyFaceDetector(input, options);
const landmarks1 = await faceapi.detectFaceLandmarks(faceImage);
const landmarks2 = await faceapi.detectFaceLandmarksTiny(faceImage);
const descriptor = await faceapi.computeFaceDescriptor(alignedFaceImage);
```

### Bir Görüntü Bölgesi İçin Kanvas Çıkarma

```javascript


const regionsToExtract = [
  new faceapi.Rect(0, 0, 100, 100)
];
// aslında extractFaces yüz bölgelerini sınırlayıcı kutulardan çıkarmak için tasarlanmıştır,
// ancak bunu başka herhangi bir bölgeyi çıkarmak için de kullanabilirsiniz
const canvases = await faceapi.extractFaces(input, regionsToExtract);
```

### Öklid Mesafesi

```javascript
// iki yüz tanımlayıcısı arasındaki öklid mesafesini hesaplamak için kullanılmak üzere tasarlanmıştır
const dist = faceapi.euclideanDistance([0, 0], [0, 10]);
console.log(dist); // 10
```

### Yüz Landmark Noktalarını ve Konturlarını Alma

```javascript
const landmarkPositions = landmarks.positions;

// veya bireysel konturların pozisyonlarını alın,
// yalnızca 68 noktalı yüz landmarkları için mevcut (FaceLandmarks68)
const jawOutline = landmarks.getJawOutline();
const nose = landmarks.getNose();
const mouth = landmarks.getMouth();
const leftEye = landmarks.getLeftEye();
const rightEye = landmarks.getRightEye();
const leftEyeBbrow = landmarks.getLeftEyeBrow();
const rightEyeBrow = landmarks.getRightEyeBrow();
```

### URL'den Görüntü Alma ve Gösterme

```html
<img id="myImg" src="">
```

```javascript
const image = await faceapi.fetchImage('/images/example.png');

console.log(image instanceof HTMLImageElement); // true

// alınan görüntü içeriğini gösterme
const myImg = document.getElementById('myImg');
myImg.src = image.src;
```

### JSON Alma

```javascript
const json = await faceapi.fetchJson('/files/example.json');
```

### Bir Görüntü Seçici Oluşturma

```html
<img id="myImg" src="">
<input id="myFileUpload" type="file" onchange="uploadImage()" accept=".jpg, .jpeg, .png">
```

```javascript
async function uploadImage() {
  const imgFile = document.getElementById('myFileUpload').files[0];
  // Bir Blob'dan bir HTMLImageElement oluşturun
  const img = await faceapi.bufferToImage(imgFile);
  document.getElementById('myImg').src = img.src;
}
```

### Bir Görüntü veya Video Elementinden Kanvas Elemanı Oluşturma

```html
<img id="myImg" src="images/example.png" />
<video id="myVideo" src="media/example.mp4" />
```

```javascript
const canvas1 = faceapi.createCanvasFromMedia(document.getElementById('myImg'));
const canvas2 = faceapi.createCanvasFromMedia(document.getElementById('myVideo'));
```

<a name="models"></a>

# Mevcut Modeller

<a name="models-face-detection"></a>

## Yüz Tespiti Modelleri

### SSD Mobilenet V1

Yüz tespiti için bu proje, MobileNetV1 tabanlı bir SSD (Single Shot Multibox Detector) uygular. Sinir ağı, bir görüntüdeki her yüzün konumlarını hesaplayacak ve her yüz için sınırlayıcı kutularla birlikte olasılıkları döndürecektir. Bu yüz dedektörü, düşük çalışma süresi yerine yüz sınırlayıcı kutularını tespit etmede yüksek doğruluk elde etmeyi hedefler. Kuantize edilmiş modelin boyutu yaklaşık 5.4 MB'dir (**ssd_mobilenetv1_model**).

Yüz tespiti modeli [WIDERFACE veri seti](http://mmlab.ie.cuhk.edu.hk/projects/WIDERFace/) üzerinde eğitilmiştir ve ağırlıklar [yeephycho](https://github.com/yeephycho) tarafından [bu](https://github.com/yeephycho/tensorflow-face-detection) repo'da sağlanmıştır.

### Tiny Face Detector

Tiny Face Detector, SSD Mobilenet V1 yüz dedektörüne kıyasla çok daha hızlı, daha küçük ve daha az kaynak tüketen çok performanslı, gerçek zamanlı bir yüz dedektörüdür. Bu model çok mobil ve web dostudur, dolayısıyla mobil cihazlar ve kaynak sınırlı istemcilerde tercih edilmelidir. Kuantize edilmiş modelin boyutu yalnızca 190 KB'dir (**tiny_face_detector_model**).

Yüz dedektörü, sınırlayıcı kutularla etiketlenmiş ~14K görüntüden oluşan özel bir veri seti üzerinde eğitilmiştir. Ayrıca, model, genel olarak SSD Mobilenet V1'e kıyasla sonraki yüz landmark tespitiyle birlikte daha iyi sonuçlar üreten sınırlayıcı kutularla yüz özelliği noktalarını tamamen kapsayacak şekilde eğitilmiştir.

Bu model, Tiny Yolo V2'nin daha da küçük bir versiyonudur ve Yolo'nun düzenli konvolüsyonlarını derin ayrılabilir konvolüsyonlarla değiştirir. Yolo tamamen konvolüsyoneldir, dolayısıyla farklı giriş görüntü boyutlarına kolayca uyum sağlar ve performans (çalışma süresi) karşılığında doğruluğu değiştirir.

<a name="models-face-landmark-detection"></a>

## 68 Noktalı Yüz Landmark Tespiti Modelleri

Bu paket, çok hafif, hızlı ve doğru bir 68 noktalı yüz landmark dedektörü uygular. Varsayılan modelin boyutu yalnızca 350kb'dir (**face_landmark_68_model**) ve küçük model yalnızca 80kb'dir (**face_landmark_68_tiny_model**). Her iki model de derin ayrılabilir konvolüsyonlar ve yoğun bağlantılı blokların fikirlerini kullanır. Modeller, 68 yüz landmark noktalarıyla etiketlenmiş ~35k yüz görüntüsü içeren bir veri seti üzerinde eğitilmiştir.

<a name="models-face-recognition"></a>

## Yüz Tanıma Modeli

Yüz tanıma için, bir yüz görüntüsünden yüz tanımlayıcı (128 değer içeren bir özellik vektörü) hesaplamak için bir ResNet-34 benzeri mimari uygulanmıştır. Bu model, eğitimde kullanılan yüz setiyle sınırlı değildir, bu da onu herhangi bir kişinin yüz tanıması için kullanabileceğiniz anlamına gelir, örneğin kendinizinki. İki rastgele yüzün benzerliğini, örneğin öklid mesafesini hesaplayarak veya tercih ettiğiniz başka bir sınıflandırıcıyı kullanarak belirleyebilirsiniz.

Sinir ağı, [face-recognition.js](https://github.com/justadudewhohacks/face-recognition.js) kullanılan **FaceRecognizerNet** ile eşdeğerdir ve [dlib](https://github.com/davisking/dlib/blob/master/examples/dnn_face_recognition_ex.cpp) yüz tanıma örneğinde kullanılan sinir ağına eşdeğerdir. Ağırlıklar [davisking](https://github.com/davisking) tarafından eğitilmiştir ve model, yüz tanıma için LFW (Etiketli Yüzler Yaban Hayatta) referans puanında %99.38 doğruluk oranına sahiptir.

Kuantize edilmiş modelin boyutu yaklaşık 6.2 MB'dir (**face_recognition_model**).

<a name="models-face-expression-recognition"></a>

## Yüz İfade Tanıma Modeli

Yüz ifade tanıma modeli hafif, hızlı ve makul bir doğruluk sağlar. Modelin boyutu yaklaşık 310kb'dir ve derin ayrılabilir konvolüsyonlar ve yoğun bağlantılı bloklar kullanır. Model, halka açık veri setlerinden ve web'den kazınan görüntülerden çeşitli görüntüler üzerinde eğitilmiştir. Gözlük takmanın tahmin sonuçlarının doğruluğunu düşürebileceğini unutmayın.

<a name="models-age-and-gender-recognition"></a>

## Yaş ve Cinsiyet Tanıma Modeli

Yaş ve cinsiyet tanıma modeli, bir özellik çıkarma katmanı, bir yaş regresyon katmanı ve bir cinsiyet sınıflandırıcısı kullanan çoklu görev ağıdır. Modelin boyutu yaklaşık 420kb'dir ve özellik çıkarıcı, Xception'a çok benzer ancak daha küçük bir mimari kullanır.

Bu model, UTK, FGNET, Chalearn, Wiki, IMDB*, CACD*, MegaAge, MegaAge-Asian veritabanlarında %80/20'lik bir eğitim/test ayrımıyla eğitilmiş ve test edilmiştir. `*`, bu veritabanlarının algoritmik olarak temizlendiğini gösterir, çünkü başlangıç veritabanları oldukça gürültülüdür.

### Toplam Test Sonuçları

Toplam MAE (Ortalama Yaş Hatası): **4.54**

Toplam Cinsiyet Doğruluğu: **%95**

### Her Veritabanı İçin Test Sonuçları

Veritabanlarında cinsiyet etiketleri bulunmadığında `-` kullanılmıştır.

Veritabanı       | UTK    | FGNET | Chalearn | Wiki | IMDB* | CACD* | MegaAge | MegaAge-Asian |
----------------|-------:|------:|---------:|-----:|------:|------:|--------:|--------------:|
MAE             | 5.25   | 4.23  | 6.24     | 6.

54 | 3.63  | 3.20  | 6.23    | 4.21          |
Cinsiyet Doğruluğu | 0.93   | -     | 0.94     | 0.95 | -     | 0.97  | -       | -             |

### Farklı Yaş Kategorisi Grupları İçin Test Sonuçları

Yaş Aralığı      | 0 - 3  | 4 - 8 | 9 - 18 | 19 - 28 | 29 - 40 | 41 - 60 | 60 - 80 | 80+     |
----------------|-------:|------:|-------:|--------:|--------:|--------:|--------:|--------:|
MAE             | 1.52   | 3.06  | 4.82   | 4.99    | 5.43    | 4.94    | 6.17    | 9.91    |
Cinsiyet Doğruluğu | 0.69   | 0.80  | 0.88   | 0.96    | 0.97    | 0.97    | 0.96    | 0.9     |
