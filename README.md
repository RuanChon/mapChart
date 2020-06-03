> ## vue中使用Echarts map图实现下钻至县级
>

因为项目需求需要实现 map 图下钻至县级，也找了很多博客，但是基本都是同一篇博客。
好在最终还是实现了
>
>
>基本思路都是一致的，获取鼠标点击的参数跳转至指定的 JSON
>
>

需要注意的是，因为我是直接从 vue-cli2 直接跳到 vue-cli4 ，还奇怪怎么读取不到JSON，查找后才知道 vue-cli3 往后的项目基础架构对比旧版本有些区别。

以前大家都习惯在根目录下的 static 文件夹下创建 JSON 文件，vue-cli3、4没有 static文件夹了，创一个 static 文件夹在下面也不行，结果会报错，请求 404找不到文件。

正确的路径是在 public 文件夹下引入

步骤如下：
> 一.    首先初始化全国 map。

      initEcharts("china")

      function initEcharts(map) {
        let option = {

          geo: {
            map: map,
            roam: false,
            scaleLimit: {
              min: 1.2,
              max: 3
            },
            zoom: 1.2,
            //图形上的文本标签，可用于说明图形的一些数据信息
            label: {
              normal: {
                show: true,
                fontSize: "10",
                color: "rgba(0,0,0,0.7)"
              }
            },
            //地图区域的多边形 图形样式，有 normal 和 emphasis 两个状态
            itemStyle: {
              //normal 是图形在默认状态下的样式；
              normal: {
                borderColor: "rgba(0, 0, 0, 0.2)"
              },
              //emphasis 是图形在高亮状态下的样式，比如在鼠标悬浮或者图例联动高亮时。
              emphasis: {
                areaColor: "#F3B329",
                shadowOffsetX: 0,
                shadowOffsetY: 0,
                shadowBlur: 20,
                borderWidth: 0,
                shadowColor: "rgba(0, 0, 0, 0.5)"
              }
            }
          },
          series: [
            {
              name: "信息量",
              type: "map",
              mapType: map,
              geoIndex: 0
              // data: dataList
            }
          ]
        };

        myChart.setOption(option);
      }

> 二.   定义省份及市区数组，由于太长了我就不放了
>
>
> 三.   点击事件
   
      // 点击触发
      myChart.on("click", param => {
        if (param.name in provinces) {
          // 处理省模块
          let names = param.name;
          for (let key in provinces) {
            if (names == key) {
              showProvince(provinces[key], key);
              break;
            }
          }

        } else if (param.name in cityMap) {
          // 处理市模块
          let names = param.name;
          for (let key in cityMap) {
            if (names == key) {
              showCitys(cityMap[key], key);
              break;
            }
          }
        }
      });


> 四. 渲染 map

      //展示对应的省
      function showProvince(eName,param) {
        console.log(eName, param)
        $.getJSON(`/map/province/${eName}.json`, data=>{
          that.$echarts.registerMap(param, data);
          alert("省")
          initEcharts(param);
        })
      }


      //展示对应市
      function showCitys(cName, param) {
        console.log(cName, param)
        // 显示县级地图
        $.getJSON(`/map/city/${cName}.json`, data=>{
          that.$echarts.registerMap(param, data);
          alert("县")
          initEcharts(param);
        })
      }

