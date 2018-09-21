# Lottie动画的入门和实践（4）Lottie动画的优化

## Lottie Json 数据简单解析

json数据中包含一个动画所有信息，包括起始帧，结束帧，高度，宽度，资源，图层等信息。

json数据是图层和帧这两个维度对动画进行记录。

```json
{
    "v":"5.1.16",
    "fr":60,  //每秒播放帧数
    "ip":0,  //起始帧
    "op":509,  //结束帧
    "w":400,  //宽
    "h":520,  //高
    "nm":'',  //动画名称
    "assets":[  //资源
        {
            "id":"image_0",  //资源id，作为动画中使用资源的索引
            "w":233,  //宽度
            "h":165,  //高度
            "u":"images/",  //路径
            "p":"img_0.png",  //文件名称（路径）
            "e":0  //资源类型（待确定），0，表示图片，1表示base64
        }，{
        	"id":"comp_0",  //资源id
        	"layers":[{
        		"ddd":0,
        		"ind":1,
        		"ty":2,
        		"nm":"2苗.png",
        		"cl":"png",
        		"refId":"image_0",  //表示引用了的资源
        		"sr":1,
        		"ks":{
        			"o":{"a":0,"k":100,"ix":11},
        			"r":{"a":0,"k":0,"ix":10},
        			"p":{"a":0,"k":[96,88,0],"ix":2},
					"a":{"a":0,"k":[116.5,82.5,0],"ix":1},
					"s":{"a":0,"k":68.181,68.181,100],"ix":6}},
					"ao":0,
					"ip":0,
					"op":549,
					"st":0,
					"bm":0}]}
		}
    ],
    "layers":[]  //图层
}
```

##  JSON数据及资源优化

一般动效老师将json的动画导出来时会包含json数据，html测试页面和images图片资源。

当动画应用到页面上时，我们更加注重动画的加载时间，缩短加载时间最直接的手段就是降低资源体积和资源数量。

资源体积和数量的降低，首先要做的就是合并重复使用的资源，其次是整合小体积的资源。

通过以上的思路，保证动效的最优使用。

1. 我们首先检查images中重复资源，将重复资源记录下来，记录`config.json`
2. 处理json数据中的`asstes`属性。
   1. 将`asstes`中的元素拆分成`assetImgs`和`assetComs`两个不同的数组。
   2. 对`assetImgs`数组进行处理
      1. 对`assetImgs`数组中的元素根据`config.json`提供的信息进行去重。
      2. 对`assetImgs`数组中定位的图片资源体积小于10k的转为base64格式，体积大于10k的导出到新位置。
   3. 对`assetComs`数组进行处理
      1. 遍历`assetComs`数组，获得`assetComs`数组元素中的`layers`对象
      2. 根据`config.json`提供的信息替换`assetComs[index].layers[index].refId`的值。
3. 处理json数据中的`layers`属性。
   1. 遍历`layers`对象。
   2. 根据`config.json`提供的信息替换`layers[index].refId`的值。
4. 导出json数据。
   1. 将`assetImgs`和`assetComs`两个数组合并为`asstes`对象。
   2. 更新json数据中的`asstes`和`layers`对象。
   3. 输出新的json文件及使用的图片资源。

```javascript
const Path = require('path')
const Fs = require('fs')
const ASSETS = 'assets'
const LAYERS = 'layers'
const basePath = Path.resolve(__dirname, './data/')
const distPath = Path.resolve(__dirname, './dist/')
/**
 * assets处理，拆分为img和com
 */
function resolveAssets(list) {
  let imgs = [],
    coms = []
  list.forEach(_asset => {
    if (_asset.hasOwnProperty(LAYERS)) {
      //处理layers
      coms.push(_asset)
    } else {
      //处理非layers
      imgs.push(_asset)
    }
  })
  return [imgs, coms]
}

//layers中refId替换
function layersRefIdReplace(layer, map) {
  layer.forEach(l => {
    let refId = l.refId
    if (!refId) return
    if (refId.match(/image_/)) {
      let n = refId.split('_')[1]
      if (map.hasOwnProperty(n)) {
        refId = refId.replace(n, map[n])
        l.refId = refId
      }
    }
  })
  return layer
}
//image资源去重
function duplicateImage(list, map) {
  let temp = []
  list.forEach(item => {
    let id = item.id
    if (id.match(/image_/)) {
      let n = id.split('_')[1]
      if (!map.hasOwnProperty(n)) {
        temp.push(item)
      }
    }
  })
  return temp
}

function assetsImageToBase64(image, subPath = '', size = 10000) {
  let path = Path.resolve(basePath, subPath, image.u + image.p)

  let stat = Fs.statSync(path)
  if (stat.size < size) {
    let originBuffer = Fs.readFileSync(path)
    let str = originBuffer.toString('base64')
    image.p = 'data:image/png;base64,' + str
    image.e = 1
    console.log(`${path} is conver to base64`)
    delete image.u
  } else {
    let outPath = Path.resolve(distPath, subPath, image.u + image.p)
    if (!Fs.existsSync(Path.resolve(distPath, subPath))) {
      Fs.mkdirSync(Path.resolve(distPath, subPath))
    }
    if (!Fs.existsSync(Path.resolve(distPath, subPath, image.u))) {
      Fs.mkdirSync(Path.resolve(distPath, subPath, image.u))
    }
    Fs.copyFileSync(path, outPath)
    console.log(`${path} is copy to ${outPath}`)
  }
  return image
}

//config map获取
function getDuplicationMap(data) {
  let result = {}
  Object.keys(data).forEach(prop => {
    data[prop].forEach(i => {
      result[i] = prop
    })
  })
  return result
}

function resolveAnimationJson(medalType) {
  let jsonPath = Path.resolve(basePath, `${medalType}/data.json`)
  let mapPath = Path.resolve(basePath, `${medalType}/config.json`)
  let data = require(jsonPath),
    map = require(mapPath)
  let assets = data[ASSETS],
    layers = data[LAYERS]

  let assetsMap = getDuplicationMap(map)
  let [assetImgs, assetComs] = resolveAssets(assets)

  let _assetImgs = duplicateImage(assetImgs, assetsMap)

  _assetImgs = _assetImgs.map(i => {
    return assetsImageToBase64(i, medalType)
  })

  let _assetComs = assetComs.map(com => {
    let layers = layersRefIdReplace(com.layers, assetsMap)
    com.layers = layers
    return com
  })
  let _layers = layersRefIdReplace(layers, assetsMap)

  let _data = data
  _data[ASSETS] = _assetImgs.concat(_assetComs)
  _data[LAYERS] = _layers

  let outPath = Path.resolve(distPath, `${medalType}/_data.json`)

  if (!Fs.existsSync(Path.resolve(distPath, medalType))) {
    Fs.mkdirSync(Path.resolve(distPath, medalType))
  }

  Fs.writeFile(outPath, JSON.stringify(_data), () => {
    console.log(`${medalType} convertion success!! Path: ${outPath}`)
  })
}

const medalTypes = ['blue', 'green', 'orange', 'red', 'yellow']

// resolveAnimationJson(medalTypes[0])
// resolveAnimationJson(medalTypes[1])
// resolveAnimationJson(medalTypes[2])
// resolveAnimationJson(medalTypes[3])
resolveAnimationJson(medalTypes[4])

```

## 优化效果对比及分析

| 资源类型        | 体积(MB) | 压缩体积(MB) | 请求数 | base64请求 | 实际请求 | 加载时间-pc(s) | 加载时间-ipad(s) |
| :-------------- | :------: | :----------: | :----: | :--------: | :------: | :------------: | :--------------: |
| 原始动画        |   6.4    |      *       |  559   |     1      |   558    |      4.69      |        *         |
| 图片去重        |   4.4    |      *       |  204   |     1      |   203    |      4.12      |        *         |
| base64          |   5.7    |     4.9      |  171   |     76     |    85    |      3.94      |       4-6        |
| 图片去重-base64 |   4.2    |     2.8      |  169   |     74     |    95    |      3.53      |       3-4        |

通过以上表格能够表示如下几点：

1. 降低**资源体积**和**资源数量**都能缩短加载时间。
2. 使用base64和图片去重都能降低资源的**体积**。
3. 图片去重降低**资源体积**效果好于base64格式。
4. 使用base64和图片去重都能降低资源的**数量**。
5. base64降低**资源数量**效果好于base64格式。
6. 结合两种方式能够最大优化资源体积和资源数量。