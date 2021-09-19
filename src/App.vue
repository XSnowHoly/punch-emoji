<template>
  <div id="app">
    <div class="box">
      <canvas id="c" width="240" height="240"></canvas>
    </div>
    <input type="file" accept="image/*" multiple @change="uploadImg" />
    <div class="action-box">
      <cube-button :disabled="disablePlay" @click="play()">预览</cube-button>
      <cube-button primary @click="donwload()">下载</cube-button>
    </div>
  </div>
</template>

<script>
import { fabric } from 'fabric';
import fist from '@/assets/fist.png';
import defaultImg from '@/assets/hehe.png';

export default {
  name: 'App',
  components: {},
  data() {
    return {
      publicPath: process.env.BASE_URL,
      templateImg: defaultImg,
      templateMoveCoords: [
        { left: 5, top: 20 },
        { left: 20, top: 5 },
        { left: 35, top: 20 },
      ],
      disablePlay: false,
      rect: null,
      canvas: null,
      index: 0,
      opinion: true,
      fistLeft: null,
      fistRgiht: null,
      fistLeftDone: false,
      fistRightDone: false,
      gifData: undefined,
      cap: null,
    };
  },
  mounted() {
    this.canvas = new fabric.StaticCanvas('c', {
      width: 240,
      height: 240,
      enableRetinaScaling: false,
      backgroundColor: '#fff',
    });
    this.templateImgInit();
    this.fistLeftInit();
    this.fistightInit();
  },
  methods: {
    // 模板图片初始化
    templateImgInit() {
      const { left, top } = this.templateMoveCoords[0];
      fabric.Image.fromURL(this.templateImg, (img) => {
        img.set({
          left,
          top,
          selectable: false,
          scaleX: 200 / img.width,
          scaleY: 200 / img.height,
        });
        this.rect = img;
        this.canvas.add(this.rect);
        this.rect.sendToBack();
      });
    },
    // 左拳初始化
    fistLeftInit() {
      fabric.Image.fromURL(fist, (oImg) => {
        oImg.scale(0.2);
        oImg.left = 30;
        oImg.top = 200;
        oImg.angle = -45;
        oImg.selectable = false;
        oImg.originX = oImg.originY = 'center';
        this.canvas.bringToFront(oImg);
        this.canvas.add(oImg);
        this.fistLeft = oImg;
      });
    },
    // 右拳初始化
    fistightInit() {
      fabric.Image.fromURL(fist, (oImg) => {
        oImg.scale(0.2);
        oImg.flipX = true; // 镜像翻转
        oImg.left = 210;
        oImg.top = 200;
        oImg.angle = 45;
        oImg.selectable = false;
        oImg.originX = oImg.originY = 'center';
        this.canvas.bringToFront(oImg);
        this.canvas.add(oImg);
        this.fistRgiht = oImg;
      });
    },
    // 模板图片动画
    animate() {
      const arr = [
        { left: 5, top: 20 },
        { left: 20, top: 5 },
        { left: 35, top: 20 },
      ];
      if (this.opinion) {
        this.index += 1;
        this.index === 2 ? (this.opinion = false) : null;
      } else {
        this.index -= 1;
        this.index === 0 ? (this.opinion = true) : null;
      }
      const postion = arr[this.index];
      this.rect.animate(postion, {
        duration: 120,
        onChange: this.canvas.renderAll.bind(this.canvas),
        onComplete: this.animate,
      });
    },
    // 左拳头动画
    fistAnimate() {
      const arr = [
        { scaleX: 0.2, scaleY: 0.2, angle: -45, left: 30, top: 200 },
        { scaleX: 0.4, scaleY: 0.4, angle: -10, left: 60, top: 180 },
      ];

      this.fistLeft.animate(this.fistLeftDone ? arr[0] : arr[1], {
        duration: 100,
        onChange: this.canvas.renderAll.bind(this.canvas),
        onComplete: this.fistRightAnimate,
        easing: fabric.util.ease.easeOutCubic,
      });
      this.fistLeftDone = !this.fistLeftDone;
    },
    // 右拳头动画
    fistRightAnimate() {
      const arr = [
        { scaleX: 0.2, scaleY: 0.2, angle: 45, left: 210, top: 200 },
        { scaleX: 0.4, scaleY: 0.4, angle: 10, left: 180, top: 180 },
      ];

      this.fistRgiht.animate(this.fistRightDone ? arr[0] : arr[1], {
        duration: 100,
        onChange: this.canvas.renderAll.bind(this.canvas),
        onComplete: this.fistAnimate,
      });
      this.fistRightDone = !this.fistRightDone;
    },
    // 播放动画
    play() {
      this.disablePlay = true;
      this.animate();
      this.fistAnimate();
    },
    // 替换图片素材
    changeImage(src) {
      this.rect.setSrc(src, (img) => {
        this.rect.set({
          left: 40,
          top: 40,
          selectable: false,
          scaleX: 200 / img.width,
          scaleY: 200 / img.height,
        });
        this.canvas.renderAll();
        // this.canvas.setCoords();
      });

      //Its a 106KB size image

      this.canvas.renderAll();
      this.canvas.calcOffset();
    },
    // 图片上传
    uploadImg(event) {
      console.log('开始上传');
      const e = window.event || event;
      const oFile = e.target.files[0];
      const imgMaxSize = 1024 * 1024 * 4;
      console.log(oFile.type.split('/')[1]);
      if (['jpeg', 'png', 'gif', 'jpg'].indexOf(oFile.type.split('/')[1]) < 0) {
        const toast = this.$createToast({
          time: 1000,
          type: 'warn',
          txt: '仅可以上传图片格式文件',
        });
        toast.show();
        return;
      }

      if (oFile.size > imgMaxSize) {
        const toast = this.$createToast({
          time: 1000,
          type: 'warn',
          txt: '文件最大为4MB',
        });
        toast.show();
        return;
      }

      const reads = new FileReader();
      reads.readAsDataURL(oFile);
      reads.onload = () => {
        this.templateImg = reads.result;
        this.changeImage(reads.result);
      };
    },

    // gif 下载
    donwload() {
      if (!this.disablePlay) {
        const toast = this.$createToast({
          txt: '请先点击预览再下载',
          type: 'warn',
        });
        toast.show();
        return;
      }
      let canvas = document.querySelector('canvas');
      const cap = new window.CCapture({
        framerate: 30,
        quality: 1,
        timeLimit: 1.2,
        name: 'punch',
        format: 'gif',
        workersPath: this.publicPath,
      });
      const toast = this.$createToast({
        time: 1000,
        txt: '生成gif图片中...',
        type: 'loading',
      });
      toast.show();
      cap.start();
      function _animate() {
        requestAnimationFrame(_animate);
        if (cap) {
          cap.capture(canvas);
        }
      }
      _animate();

      // setTimeout(() => {
      //   cap.stop();
      //   cap.save();
      // }, 1000);

      // fabric.util.requestAnimFrame(function render() {
      //   fabric.util.requestAnimFrame(render);
      //   if(cap) {
      //     cap.capture(canvas);
      //   }
      // });
    },
  },
};
</script>
<style lang="less" scoped>
html,
body {
  height: 100%;
  margin: 0;
  padding: 0;
}
#app {
  height: 100%;
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  .box {
    height: 400px;
    display: flex;
    justify-content: center;
    align-items: center;
    .img-templete {
      position: absolute;
      width: 200px;
      top: 100px;
      left: 20px;
      img {
        width: 100%;
      }
    }
  }
  .action-box {
    margin-top: 10px;
    display: flex;
    justify-content: center;
  }
  // #c {
  //   background: #ccc;
  // }
}
</style>
