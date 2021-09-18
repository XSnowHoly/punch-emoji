<template>
  <div id="app">
    <div class="box">
      <canvas id="c" width="300" height="300"></canvas>
      <div class="img-templete">
        <img
          id="img"
          width="200"
          height="200"
          :src="templateImg"
          crossOrigin
          alt=""
        />
      </div>
    </div>
    <input type="file" accept="image/*" multiple @change="uploadImg" />
    <button @click="play()">开始</button>
    <button @click="gifRender()">下载</button>
    <button @click="stop()">停止</button>
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
      font: '48px Arial',
      rect: null,
      canvas: null,
      index: 0,
      opinion: true,
      fistLeft: null,
      fistRgiht: null,
      fistLeftDone: false,
      fistRightDone: false,
      templateImg: defaultImg,
      gifData: undefined,
      cap: null,
      publicPath: process.env.BASE_URL
    };
  },
  mounted() {
    // fabric.Object.prototype.originX = fabric.Object.prototype.originY = 'center';
    this.canvas = new fabric.Canvas('c');
    this.templateImgInit();
    fabric.Image.fromURL(fist, (oImg) => {
      oImg.scale(0.4); //图片缩小一半
      oImg.left = 60;
      oImg.top = 220;
      oImg.angle = -45;
      oImg.selectable = false;
      oImg.originX = oImg.originY = 'center';
      this.canvas.bringToFront(oImg);
      this.canvas.add(oImg);
      this.fistLeft = oImg;
    });
    fabric.Image.fromURL(fist, (oImg) => {
      oImg.scale(0.4); //图片缩小一半
      oImg.flipX = true;
      oImg.left = 240;
      oImg.top = 220;
      oImg.angle = 45;
      oImg.selectable = false;
      oImg.originX = oImg.originY = 'center';
      this.canvas.bringToFront(oImg);
      this.canvas.add(oImg);
      this.fistRgiht = oImg;
    });
  },
  methods: {
    templateImgInit() {
      fabric.Image.fromURL(this.templateImg, (img) => {
        img.set({
          left: 40,
          top: 40,
          selectable: false,
          scaleX: 200 / img.width,
          scaleY: 200 / img.height,
        });
        this.rect = img;
        this.canvas.add(this.rect);
        this.rect.sendToBack();
      });
    },
    animate() {
      const arr = [
        { left: 30, top: 40 },
        { left: 50, top: 20 },
        { left: 70, top: 40 },
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
        duration: 200,
        onChange: this.canvas.renderAll.bind(this.canvas),
        onComplete: this.animate,
      });
    },
    fistAnimate() {
      const arr = [
        { scaleX: 0.4, scaleY: 0.4, angle: -45 },
        { scaleX: 0.65, scaleY: 0.65, angle: -10 },
      ];

      this.fistLeft.animate(this.fistLeftDone ? arr[0] : arr[1], {
        duration: 100,
        onChange: this.canvas.renderAll.bind(this.canvas),
        onComplete: this.fistRightAnimate,
        easing: fabric.util.ease.easeOutCubic,
      });
      this.fistLeftDone = !this.fistLeftDone;
    },
    fistRightAnimate() {
      const arr = [
        { scaleX: 0.4, scaleY: 0.4, angle: 45 },
        { scaleX: 0.65, scaleY: 0.65, angle: 10 },
      ];

      this.fistRgiht.animate(this.fistRightDone ? arr[0] : arr[1], {
        duration: 100,
        onChange: this.canvas.renderAll.bind(this.canvas),
        onComplete: this.fistAnimate,
      });
      this.fistRightDone = !this.fistRightDone;
    },
    play() {
      this.animate();
      this.fistAnimate();
    },
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
        this.canvas.setCoords();
      });

      //Its a 106KB size image

      this.canvas.renderAll();
      this.canvas.calcOffset();
    },
    uploadImg(event) {
      console.log('开始上传');
      const e = window.event || event;
      const oFile = e.target.files[0];
      const imgMaxSize = 1024 * 1024 * 4;
      console.log(oFile.type.split('/')[1]);
      if (['jpeg', 'png', 'gif', 'jpg'].indexOf(oFile.type.split('/')[1]) < 0) {
        alert('仅可以上传图片格式文件');
        return;
      }

      if (oFile.size > imgMaxSize) {
        alert('文件最大为4MB');
        return;
      }

      const reads = new FileReader();
      reads.readAsDataURL(oFile);
      reads.onload = () => {
        console.log('输出图片');
        this.templateImg = reads.result;
        this.changeImage(reads.result);
      };
    },

    gifRender() {
      let canvas = document.querySelector('canvas');
      const cap = new window.CCapture({
        format : 'gif',
        workersPath: this.publicPath,
      });
      cap.start();
      function animate() {
        requestAnimationFrame(animate);
        if (cap) {
          cap.capture(canvas);
        }
      }
      animate();

      setTimeout(() => {
        cap.stop();
        cap.save();
      }, 5000);

      // fabric.util.requestAnimFrame(function render() {
      //   fabric.util.requestAnimFrame(render);
      //   if(cap) {
      //     cap.capture(canvas);
      //   }
      // });
    },

    // 使用fileSaver.js保存
    stop() {
      this.cap.stop();
      this.cap.save();
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
    height: 600px;
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
  #c {
    background: #ccc;
  }
}
</style>
