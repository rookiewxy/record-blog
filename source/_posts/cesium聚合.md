## 聚合效果

基于`cesium`实现聚合效果，(官网案例)[https://sandcastle.cesium.com/index.html?src=Clustering.html],可以看出来，其实是非常简单和单调的，实际开发过程中肯定比这个更复杂，但是我们现在以及非常明确的知道，`DataSource`对象中的`cluster`就是聚合的关键，也就是实现聚合效果的核心对象是`EntityCluster`，因为我们的数据源不是json格式也不是官网案例的那种格式，而是普通的对象，所以使用自定义数据`CustomDataSource`实现，代码如下:

```tsx
const dataSourceForCluster = new Xt.Cesium.CustomDataSource('cluster');
    for (const d of data) {
      const data = {
        position: new Xt.Cesium.Cartesian3.fromDegrees(d.y, d.x, 0),
        billboard: {
          image: IconImage,
        },
        id: d.id,
        text: d.name
      }
      dataSourceForCluster.entities.add(data);
    }
```

创建了数据源，就要对聚合做样式做修改了，网上有使用cavnas绘制图片和文字的但是这种方式是获取不到cesium实体类中的点击方式的，所以我们直接对`cluster`做修改，修改其中的`label`和`billboard`,当然需要通过事件获取`cluster`对象
```ts
 this.cluserLayerResource.clustering.clusterEvent.addEventListener(
            async (clusteredEntities: any[], cluster: any) => {
                if (clusteredEntities.length > this.minimumClusterSize) {
                    this.rebuildCluster(cluster, {
                        width: 44,
                        height: 42,
                        image: IconMark,
                        text: clusteredEntities.length.toString(),
                        padding:[6,6]
                    });
                } else {
                    this.rebuildCluster(cluster, {
                        width: 212,
                        height: 84,
                        offsetY: -14,
                        text: clusteredEntities[0]?.text,
                        image: IconPoint,
                        padding:[55,6]
                    });
                }

            }
        )


 rebuildCluster(cluster: any, config: any) {
        cluster.label.show = true;
        cluster.billboard.show = true;
        cluster.label.font = "20px AlimamaDongFangDaKai";
        cluster.label.text = config.text;
        cluster.label.verticalOrigin = Xt.Cesium.VerticalOrigin.CENTER;
        cluster.label.horizontalOrigin = Xt.Cesium.HorizontalOrigin.CENTER;
        cluster.label.pixelOffset = new Xt.Cesium.Cartesian2(config.offsetX, config.offsetY)
        cluster.billboard.image = config.image;
    }
```


但是这样的问题是会造成图表把文字遮住了，需要设置`showBackground`为`true`,在设置背景色，如果直接设置为`transparent`字体又会有一圈边框，并且去不掉，实在不知道解决方法，不想用canvas来绘制，因为cluster点击事件就没了，于是我将背景色设置为和图标一样的底色，label也不能设置宽度，使用`backgroundPadding`解决
```ts
    cluster.label.showBackground = true;
    cluster.label.backgroundColor = new Xt.Cesium.Color(71 / 255, 68 / 255, 61 / 255, 0.8);
    cluster.label.backgroundPadding = new Xt.Cesium.Cartesian2(config.padding[0], config.padding[1]);
```