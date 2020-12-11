<template>
  <div class="longTake" ref="longTake" @touchstart="touch()">
    <!-- <div>{{ devicint }}</div> -->
    <div class="loading" v-show="loadingShow">
      <p>载入中：{{ loading }}%</p>
    </div>
  </div>
</template>

<script>
import * as PIXI from "pixi.js";
import { TweenMax, TimelineMax } from "gsap";
import AlloyTouch from "alloytouch";
// 环境判断
var browser = {
  versions: (function () {
    var u = navigator.userAgent,
      app = navigator.appVersion;
    return {
      trident: u.indexOf("Trident") > -1, //IE内核
      presto: u.indexOf("Presto") > -1, //opera内核
      webKit: u.indexOf("AppleWebKit") > -1, //苹果、谷歌内核
      gecko: u.indexOf("Gecko") > -1 && u.indexOf("KHTML") == -1, //火狐内核
      mobile: !!u.match(/AppleWebKit.*Mobile.*/), //是否为移动终端
      ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), //ios终端
      android: u.indexOf("Android") > -1 || u.indexOf("Adr") > -1, //android终端
      iPhone: u.indexOf("iPhone") > -1, //是否为iPhone或者QQHD浏览器
      iPad: u.indexOf("iPad") > -1, //是否iPad
      webApp: u.indexOf("Safari") == -1, //是否web应该程序，没有头部与底部
      weixin: u.indexOf("MicroMessenger") > -1, //是否微信 （2015-01-22新增）
      qq: u.indexOf("QQ") > -1, //是否QQ
    };
  })(),
  language: (navigator.browserLanguage || navigator.language).toLowerCase(),
};
// -->
export default {
  name: "longTake",
  data() {
    const _this = this;
    return {
      clientWidth: window.innerWidth,
      clientHeight: window.innerHeight,
      stage: null,
      loader: null,
      timeline: null,
      AlloyTouch: null,
      // 容器处理
      sence: [
        {
          name: "sence-1",
          x: 0,
          y: 0,
        },
        {
          name: "sence-2",
          x: 0,
          y: 0,
        },
      ],
      // 容器处理
      scenesItem: {},
      // 精灵处理
      sprData: [
        {
          sence: "sence-1",
          name: "bg",
          width: 1080,
          height: 2339,
          x: window.innerWidth / 2,
          y: window.innerHeight / 2,
          img: {
            name: "item-bg-1",
            url: require("@/assets/image/common/bg.jpg"),
          },
          anchorX: 0.5,
          anchorY: 0.5,
          mode: "fitWidth",
        },
        // 添加一堆随机尺寸随机大小的烟花。
        ...Array.from(new Array(_this.random(20, 50)).keys())
          .slice(0)
          .map((v, i) => {
            let scale = _this.random(1, 10) * 0.1;
            let delay = _this.random(1, 10);
            return {
              sence: "sence-1",
              name: "fireworks",
              width: 152 * scale,
              height: 149.5 * scale,
              x: window.innerWidth * Math.random().toFixed(1),
              y: window.innerHeight * Math.random().toFixed(1),
              img: {
                name: `item-fireworks-1`,
                url: require("@/assets/image/common/fireworks.png"),
              },
              anchorX: 0.5,
              anchorY: 0.5,
              mode: "",
              ani: {
                duration: 2,
                delay: delay,
                autoPlay: true,
                from: {
                  alpha: 1,
                  width: 0,
                  height: 0,
                  delay: delay,
                  repeat: -1,
                  repeatDelay: _this.random(1, 20) * 0.1,
                },
                to: {
                  alpha: 0,
                  width: 152 * 1.2 * scale,
                  height: 149.5 * 1.2 * scale,
                  repeat: -1,
                  delay: delay,
                  repeatDelay: _this.random(1, 20) * 0.1,
                },
              },
            };
          }),
        // -->
        {
          sence: "sence-1",
          name: "cover",
          img: _this.getFrame("cover", 193),
          width: 752,
          height: 1080,
          x: window.innerWidth / 2,
          y: window.innerHeight / 2,
          anchorX: 0.5,
          anchorY: 0.5,
          frame: true,
          infinite: true,
          setIntervalTime: 30,
          mode: "fitHeight",
          ani: {
            duration: 0.4,
            delay: 0.1,
            from: {
              alpha: 1,
            },
            to: {
              alpha: 0,
              width: 752 * 2,
              height: 1080 * 2,
            },
            // props: "scale",
            // // propsFrom: {
            // //   x: 1,
            // //   y: 1,
            // // },
            // propsTo: {
            //   x: 0,
            //   y: 0,
            // },
          },
        },
        {
          sence: "sence-2",
          name: "pic1",
          img: _this.getFrame("pic1", 68),
          width: window.innerWidth / 2,
          height: ((1080 / 1920) * window.innerWidth) / 2,
          x: 0,
          y: 0,
          anchorX: 0,
          anchorY: 0,
          frame: true,
          mode: "",
          ani: {
            duration: 0.3,
            delay: 0.1,
            from: {
              alpha: 0,
              y: 100,
            },
            to: {
              alpha: 1,
              y: 0,
            },
          },
        },
        {
          sence: "sence-2",
          name: "pic2",
          img: _this.getFrame("pic2", 68),
          width: window.innerWidth / 2,
          height: ((1080 / 1920) * window.innerWidth) / 2,
          x: window.innerWidth / 2,
          y: ((1080 / 1920) * window.innerWidth) / 2,
          anchorX: 0,
          anchorY: 0,
          frame: true,
          mode: "",
          ani: {
            duration: 0.3,
            delay: 0.4,
            from: {
              alpha: 0,
              y: ((1080 / 1920) * window.innerWidth) / 2 + 100,
            },
            to: {
              alpha: 1,
              y: ((1080 / 1920) * window.innerWidth) / 2,
            },
          },
        },
        {
          sence: "sence-2",
          name: "pic3",
          img: _this.getFrame("pic3", 68),
          width: window.innerWidth / 2,
          height: ((1080 / 1920) * window.innerWidth) / 2,
          x: 0,
          y: (((1080 / 1920) * window.innerWidth) / 2) * 2,
          anchorX: 0,
          anchorY: 0,
          frame: true,
          mode: "",
          ani: {
            duration: 0.3,
            delay: 0.7,
            from: {
              alpha: 0,
              y: (((window.innerWidth / 1920) * 1080) / 2) * 2 + 100,
            },
            to: {
              alpha: 1,
              y: (((window.innerWidth / 1920) * 1080) / 2) * 2,
            },
          },
        },
      ],
      // 精灵处理
      sprItem: {},
      // 文字处理
      textData: [
        ..._this.getText(
          "美丽的烟花只在一瞬，美丽的烟花只在一瞬，美丽的烟花只在一瞬",
          10,
          20,
          {
            sence: "sence-2",
            width: window.innerWidth / 2,
            height: ((1080 / 1920) * window.innerWidth) / 2,
            x: (window.innerWidth / 4) * 2 + 20,
            y: 20,
            anchorX: 0,
            anchorY: 0,
            opts: {
              fontFamily: "Arial",
              fontSize: _this.getFontSize(14),
              fill: 0xff0000,
              align: "center",
            },
            ani: {
              duration: 0.3,
              delay: 0.1,
              from: {
                alpha: 0,
              },
              to: {
                alpha: 1,
              },
            },
          }
        ),
        ..._this.getText("美丽的烟花只在一瞬，美丽的烟花只在一瞬", 10, 20, {
          sence: "sence-2",
          width: window.innerWidth / 2,
          height: ((1080 / 1920) * window.innerWidth) / 2,
          x: 20,
          y: ((1080 / 1920) * window.innerWidth) / 2 + 20,
          anchorX: 0,
          anchorY: 0,
          opts: {
            fontFamily: "Arial",
            fontSize: _this.getFontSize(14),
            fill: 0x000000,
            align: "center",
          },
          ani: {
            duration: 0.3,
            delay: 0.4,
            from: {
              alpha: 0,
            },
            to: {
              alpha: 1,
            },
          },
        }),
        ..._this.getText("美丽的烟花只在一瞬，美丽的烟花只在一瞬", 10, 20, {
          sence: "sence-2",
          width: window.innerWidth / 2,
          height: ((1080 / 1920) * window.innerWidth) / 2,
          x: (window.innerWidth / 4) * 2 + 20,
          y: (((1080 / 1920) * window.innerWidth) / 2) * 2 + 20,
          anchorX: 0,
          anchorY: 0,
          opts: {
            fontFamily: "Arial",
            fontSize: _this.getFontSize(14),
            fill: 0x000000,
            align: "center",
          },
          ani: {
            duration: 0.3,
            delay: 0.7,
            from: {
              alpha: 0,
            },
            to: {
              alpha: 1,
            },
          },
        }),
      ],
      // 文字处理
      textItem: {},
      // setIntervalItem: [],
      progress: 0,
      loading: 0,
      loadingShow: true,
      devicint: {},
      isTouch: false
    };
  },
  mounted() {
    this.initStage();
  },
  watch: {
    progress(val) {
      // 不循环
      this.sprData.map((v, i) => {
        const { frame, infinite, ani, img, name } = v;
        if (frame) {
          if (!infinite) {
            if (ani) {
              let frameProgress = (val - ani.duration) / ani.delay;
              let index = Math.floor(frameProgress * img.length);
              if (index < img.length && index >= 0) {
                this.sprItem[`sprite${i + 1}`].texture = this.loader.resources[
                  `item-${name}-${index + 1}`
                ].texture;
              }
            }
          }
        }
      });
    },
  },
  destroyed() {
    // this.clearIntervalItem();
    window.removeEventListener('deviceorientation', this.updateGravity. false)
  },
  methods: {
    async touch() {
      if(this.isTouch){
        return false
      }
      console.log(1111)
      this.isTouch = true;
      if (browser.versions.iPhone || browser.versions.iPad) {
        var str = navigator.userAgent.toLowerCase();
        var ver = str
          .match(/cpu iphone os (.*?) like mac os/)[1]
          .replace(/_/g, ".");
        try {
          if (ver.split(".")[0] >= "13") {
            const res = await this.getDeviceMotionEvent();
            if (res) {
              window.addEventListener(
                "deviceorientation",
                this.updateGravity,
                false
              );
            }
          } else {
            window.addEventListener(
              "deviceorientation",
              this.updateGravity,
              false
            );
          }
        } catch (error) {
          console.log(error);
        }
      } else {
        console.log(2222)
        window.addEventListener("deviceorientation", this.updateGravity, false);
      }
    },
    getDeviceMotionEvent() {
      return new Promise((resolve, reject) => {
        DeviceMotionEvent.requestPermission()
          .then((res) => {
            console.log(res);
            if (res == "granted") {
              resolve(res);
            } else {
              reject(res);
              alert("你拒绝了授权");
              return false;
            }
          })
          .catch((err) => {
            console.log(err);
            reject(err);
            alert("操作错误");
            return false;
          });
      });
    },
    updateGravity(event) {
      var absolute = event.absolute;
      var alpha = event.alpha;
      var beta = event.beta;
      var gamma = event.gamma;
      this.devicint = {
        absolute: absolute,
        alpha: alpha,
        beta: beta,
        gamma: gamma,
      };
      console.log(this.devicint)
    },
    initStage() {
      this.stage = new PIXI.Application({
        width: this.clientWidth,
        height: this.clientHeight,
        transparent: true,
        resolution: window.devicePixelRatio,
      });
      const $longTake = this.$refs.longTake;
      $longTake.appendChild(this.stage.view);

      let sprArr = [];

      this.loader = new PIXI.Loader();
      this.sprData.map(({ img }, i) => {
        // 性能优化处理，存在相同名字的纹理时不再次添加。
        if (img?.name) {
          if (this.loader.resources[img.name]) {
            return false;
          }
        }
        this.loader.add(img);
      });

      this.loader.load(async (loader, res) => {
        PIXI.utils.clearTextureCache();
        // await this.initBg();
        await this.initSence();
        await this.initTimeLine();
        await this.initSpr();
        await this.initText();
        // if (!this.getPhoneWindow()) {
        // this.stage.stage.rotation = Math.PI / 2
        // this.stage.stage.pivot.set(0.5, 0.5)
        //   await this.initTouch(false);
        // } else {
        await this.initTouch(true);
        // }
      });

      // this.loader 加载中的回调事件。
      this.loader.onProgress.add((e) => {
        this.loading = Math.floor(e.progress);
      });

      // this.loader 加载完成的回调事件。
      this.loader.onComplete.add((e) => {
        setTimeout(() => {
          this.loadingShow = false;
        }, 300);
      });
    },
    initBg() {
      let bg = new PIXI.Graphics();
      bg.beginFill(0x000000);
      bg.drawRect(0, 0, this.stage.screen.width, this.stage.screen.height);
      bg.endFill();
      bg.x = 0;
      bg.y = 0;
      this.stage.stage.addChild(bg);
    },
    initSence() {
      this.sence.map((v, i) => {
        this.scenesItem[v.name] = new PIXI.Container();
        this.scenesItem[v.name].x = v.x;
        this.scenesItem[v.name].y = v.y;
        this.stage.stage.addChild(this.scenesItem[v.name]);
      });
    },
    initSpr() {
      this.sprData.map(async (v, i) => {
        const {
          frame,
          img,
          infinite,
          mode,
          name,
          sence,
          setIntervalTime,
          width,
          height,
          x,
          y,
          anchorX,
          anchorY,
          ani,
        } = v;
        // 帧动画
        if (frame) {
          // 循环
          if (infinite) {
            // 用定时器的方法做帧动画。
            // this.sprItem[`sprite${i + 1}`] = new PIXI.Sprite(
            //   this.loader.resources[img[0].name].texture
            // );
            // let _i = 0;
            // let timer = setInterval(() => {
            //   if (_i >= img.length - 1) {
            //     _i = 0;
            //   } else {
            //     _i++;
            //   }
            //   this.sprItem[`sprite${i + 1}`].texture = this.loader.resources[
            //     img[_i].name
            //   ].texture;
            // }, setIntervalTime);
            // this.setIntervalItem.push(timer);

            // 用PIXI.AnimatedSprite 的方法做帧动画。
            let frameArr = img.map((v, i) => {
              return this.loader.resources[v.name].texture;
            });

            this.sprItem[`sprite${i + 1}`] = new PIXI.AnimatedSprite(frameArr);
            this.sprItem[`sprite${i + 1}`].animationSpeed = 0.7;
            this.sprItem[`sprite${i + 1}`].play();
          } else {
            // 不循环
            this.sprItem[`sprite${i + 1}`] = new PIXI.Sprite(
              this.loader.resources[img[0].name].texture
            );
            // 不循环用watch 移步watch函数集
          }
        } else {
          // 不是帧动画
          this.sprItem[`sprite${i + 1}`] = new PIXI.Sprite(
            this.loader.resources[img.name].texture
          );
        }

        let size = this.setSize(mode, width, height);
        this.sprItem[`sprite${i + 1}`].width = size.width;
        this.sprItem[`sprite${i + 1}`].height = size.height;
        // 设置精灵位置 -->

        // BUG(已定位): 设置xy 动画里涉及宽高会闪屏， 因为AlloyTouch value min设置异常导致。
        // 已解决
        this.sprItem[`sprite${i + 1}`].position.x = x;
        this.sprItem[`sprite${i + 1}`].position.y = y;
        this.sprItem[`sprite${i + 1}`].anchor.x = anchorX;
        this.sprItem[`sprite${i + 1}`].anchor.y = anchorY;
        // 设置精灵位置 -->

        // await 添加精灵
        await this.scenesItem[sence].addChild(this.sprItem[`sprite${i + 1}`]);

        // 动画 -->
        this.setAniTimeLine(ani, this.sprItem[`sprite${i + 1}`]);
        // 动画 -->
      });
    },
    initText() {
      this.textData.map((v, i) => {
        const {
          text,
          opts,
          mode,
          width,
          height,
          x,
          y,
          anchorX,
          anchorY,
          ani,
        } = v;
        const style = new PIXI.TextStyle(opts);
        this.textItem[`text${i + 1}`] = new PIXI.Text(text, style);

        // 设置位置 -->
        // const size = this.setSize(mode, width, height);
        // this.textItem[`text${i + 1}`].width = size.width;
        // this.textItem[`text${i + 1}`].height = size.height;
        this.textItem[`text${i + 1}`].position.x = x;
        this.textItem[`text${i + 1}`].position.y = y;
        this.textItem[`text${i + 1}`].anchor.x = anchorX;
        this.textItem[`text${i + 1}`].anchor.y = anchorY;
        // 设置位置 -->
        this.scenesItem[v.sence].addChild(this.textItem[`text${i + 1}`]);

        // 动画 -->
        this.setAniTimeLine(ani, this.textItem[`text${i + 1}`]);
        // 动画 -->
      });
    },
    initTimeLine() {
      this.timeline = new TimelineMax({
        paused: true,
      });
    },
    initTouch(type) {
      const _this = this;
      // let min = -Math.floor(this.stage.stage.height - window.innerHeight);
      let min = type ? -(this.clientWidth * 2) : -(this.clientWidth * 2);
      this.AlloyTouch = new AlloyTouch({
        touch: "body",
        vertical: type,
        min: min,
        max: 0,
        maxSpeed: 1,
        sensitivity: 0.6,
        bindSelf: false,
        value: 0,
        change: (value) => {
          //  滚动时舞台Y轴一起滚动
          // this.stage.stage.position.y =
          //   value >= 0 ? 0 : value <= min ? min : value;
          // 总播放进度
          // this.progress =
          //   -value / (this.stage.stage.height - window.innerHeight);
          this.progress = -value / -min;
          this.progress = this.progress < 0 ? 0 : this.progress;
          this.progress = this.progress > 1 ? 1 : this.progress;
          // console.log(this.progress)
          // 控制进度条
          this.timeline.seek(this.progress);
        },
      });
    },
    getFrame(name, num) {
      return Array.from(new Array(num).keys())
        .slice(0)
        .map((v, i) => {
          let pathName = `00000${i + 1}`.slice(-num.toString().length);
          return {
            name: `item-${name}-${i + 1}`,
            url: require(`@/assets/image/${name}/${name} ${pathName}.jpg`),
          };
        });
    },
    setSize(mode, w, h) {
      if (mode == "fitWidth") {
        return {
          width: this.stage.screen.width,
          height: (this.stage.screen.width / w) * h,
        };
      } else if (mode == "fitHeight") {
        return {
          width: (this.stage.screen.height / h) * w,
          height: this.stage.screen.height,
        };
      } else {
        return {
          width: w,
          height: h,
        };
      }
    },
    getFontSize(size) {
      return (window.innerWidth / 375) * size;
    },
    getText(text, line, setY, opts) {
      // 格式化文本精灵
      if (text.length < line) {
        return false;
      }
      let textArr = [];
      for (let i = 0, len = text.length; i < len / line; i++) {
        let str = text.slice(line * i, line * (i + 1));
        textArr.push(str);
      }
      let Y = opts.y;
      return textArr.map((v, i) => {
        let inlineHeight = Y + setY * i;
        opts.y = inlineHeight;
        return {
          text: v,
          ...opts,
        };
      });
    },
    setAniTimeLine(ani, item) {
      // 设置动画 -->
      if (ani) {
        let tw;
        if (ani.from && ani.to) {
          tw = new TweenMax.fromTo(item, ani.duration, ani.from, ani.to);
        } else if (ani.from) {
          tw = new TweenMax.from(item, ani.duration, ani.from);
        } else if (ani.to) {
          tw = new TweenMax.to(item, ani.duration, ani.to);
        }
        // 设置每个精灵的时间轴 添加到主时间轴里
        // delay：延时
        if (!ani.autoPlay) {
          let twTimeline = new TimelineMax({
            delay: ani.delay,
          });
          twTimeline.add(tw, 0);
          twTimeline.play();
          this.timeline.add(twTimeline, 0);
        }
        // 特殊动画 -->
        if (ani.props) {
          let propsTw;
          if (ani.propsFrom && ani.propsTo) {
            propsTw = new TweenMax.fromTo(
              item[ani.props],
              ani.duration,
              ani.propsFrom,
              ani.propsTo
            );
          } else if (ani.propsFrom) {
            propsTw = new TweenMax.from(
              item[ani.props],
              ani.duration,
              ani.propsFrom
            );
          } else if (ani.propsTo) {
            propsTw = new TweenMax.to(
              item[ani.props],
              ani.duration,
              ani.propsTo
            );
          }
          if (!ani.autoPlay) {
            let twPropsTimeline = new TimelineMax({ delay: ani.delay || 0 });
            twPropsTimeline.add(propsTw, 0);
            twPropsTimeline.play();
            this.timeline.add(twPropsTimeline, 0);
          }
        }
        // 特殊动画 -->
      }
      // 设置动画 -->
    },
    // clearIntervalItem() {
    //   this.setIntervalItem.map((v, i) => {
    //     clearInterval(v);
    //   });
    // },
    random(min, max) {
      if (max == null) {
        max = min;
        min = 0;
      }
      return min + Math.floor(Math.random() * (max - min + 1));
    },
    getPhoneWindow() {
      if (window.orientation === 180 || window.orientation === 0) {
        console.log("竖屏");
        return true;
      }
      if (window.orientation === 90 || window.orientation === -90) {
        console.log("横屏");
        return false;
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
.longTake {
  width: 100vw;
  height: 100vh;
  /deep/ canvas {
    width: 100%;
    height: 100%;
    display: block;
  }
  .loading {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: #000;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 2em;
    color: #fff;
  }
}
</style>
