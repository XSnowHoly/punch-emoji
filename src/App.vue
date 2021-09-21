<template>
  <div id="app">
    <div class="punch-descript">👊😅👊 打拳就完事了~</div>
    <div class="punch">
      <canvas id="c" width="240" height="240"></canvas>
    </div>
    <div class="punch-list">
      <h2>拳头类型：</h2>
      <cube-checker
        v-model="checkPunchStyleValue"
        :options="punchStyleList"
        type="radio"
      />
    </div>
    <Notice />
    <input type="file" accept="image/*" multiple @change="uploadImg" />
    <div class="action-box">
      <cube-button :disabled="disablePlay" @click="play()">预览</cube-button>
      <cube-button :disabled="downlaodLock" primary @click="donwload()"
        >下载</cube-button
      >
    </div>
    <cube-popup type="my-popup" ref="myPopup">
      <div class="cropper-wrap">
        <vue-cropper
          autoCrop
          :outputSize="cropperOptions.outputSize"
          :img="cropperImg"
          ref="cropper"
          centerBox
        />
        <cube-button @click="getCropData()" style="margin:10px 0" type="primary"
          >裁剪确定</cube-button
        >
      </div>
    </cube-popup>
  </div>
</template>

<script>
import { fabric } from 'fabric';
import Notice from './components/Notice.vue';
import fistImg from '@/assets/fist.png';
import punchImg from '@/assets/punch.png';
import defaultImg from '@/assets/hehe.png';

export default {
  name: 'App',
  components: {
    Notice,
  },
  data() {
    return {
      publicPath: process.env.BASE_URL,
      checkPunchStyleValue: fistImg,
      punchStyleList: [
        {
          value: fistImg,
          text: '普通拳',
        },
        {
          value: punchImg,
          text: '火焰拳',
        },
      ],
      cropperImg: null,
      templateImg: defaultImg,
      templateMoveCoords: [
        { left: 5, top: 20 },
        { left: 20, top: 5 },
        { left: 35, top: 20 },
      ],
      puncLeftMoveCoords: [
        { scaleX: 0.2, scaleY: 0.2, angle: -45, left: 30, top: 170 },
        { scaleX: 0.35, scaleY: 0.35, angle: -10, left: 60, top: 150 },
      ],
      puncRightMoveCoords: [
        { scaleX: 0.2, scaleY: 0.2, angle: 45, left: 170, top: 170 },
        { scaleX: 0.35, scaleY: 0.35, angle: 10, left: 140, top: 150 },
      ],
      cropperOptions: {
        outputSize: 1,
      },
      disablePlay: false,
      rect: null,
      canvas: null,
      index: 0,
      opinion: true,
      fistLeft: null,
      fistRight: null,
      fistLeftDone: false,
      fistRightDone: false,
      gifData: undefined,
      cap: null,
      downlaodLock: false,
    };
  },
  mounted() {
    this.canvas = new fabric.StaticCanvas('c', {
      width: 200,
      height: 200,
      enableRetinaScaling: false,
      backgroundColor: '#fff',
    });
    this.templateImgInit();
    this.fistLeftInit();
    this.fistightInit();
  },
  watch: {
    checkPunchStyleValue(val) {
      this.changePunchImage(val);
    },
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
          scaleX: 160 / img.width,
          scaleY: 160 / img.height,
        });
        this.rect = img;
        this.canvas.add(this.rect);
        this.rect.sendToBack();
      });
    },
    // 左拳初始化
    fistLeftInit() {
      fabric.Image.fromURL(fistImg, (oImg) => {
        oImg.scale(0.2);
        oImg.left = 30;
        oImg.top = 170;
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
      fabric.Image.fromURL(fistImg, (oImg) => {
        oImg.scale(0.2);
        oImg.flipX = true; // 镜像翻转
        oImg.left = 170;
        oImg.top = 170;
        oImg.angle = 45;
        oImg.selectable = false;
        oImg.originX = oImg.originY = 'center';
        this.canvas.bringToFront(oImg);
        this.canvas.add(oImg);
        this.fistRight = oImg;
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
      this.fistLeft.animate(
        this.fistLeftDone
          ? this.puncLeftMoveCoords[0]
          : this.puncLeftMoveCoords[1],
        {
          duration: 100,
          onChange: this.canvas.renderAll.bind(this.canvas),
          onComplete: this.fistRightAnimate,
          easing: fabric.util.ease.easeOutCubic,
        },
      );
      this.fistLeftDone = !this.fistLeftDone;
    },
    // 右拳头动画
    fistRightAnimate() {
      this.fistRight.animate(
        this.fistRightDone
          ? this.puncRightMoveCoords[0]
          : this.puncRightMoveCoords[1],
        {
          duration: 100,
          onChange: this.canvas.renderAll.bind(this.canvas),
          onComplete: this.fistAnimate,
        },
      );
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
      const { left, top } = this.templateMoveCoords[0];
      this.rect.setSrc(src, (img) => {
        this.rect.set({
          left,
          top,
          selectable: false,
          scaleX: 160 / img.width,
          scaleY: 160 / img.height,
        });
        this.canvas.renderAll();
      });
    },
    // 替换拳头图片素材
    changePunchImage(src) {
      this.fistLeft.setSrc(src, () => {
        this.fistLeft.set({
          ...this.puncLeftMoveCoords[0],
          selectable: false,
          originX: 'center',
          originY: 'center',
        });
        this.canvas.renderAll();
      });
      this.fistRight.setSrc(src, () => {
        this.fistRight.set({
          ...this.puncRightMoveCoords[0],
          selectable: false,
          originX: 'center',
          originY: 'center',
        });
        this.canvas.renderAll();
      });
    },
    // 图片上传
    uploadImg(event) {
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
      const toast = this.$createToast({
        time: 0,
        txt: '图片上传中',
      });
      toast.show();

      try {
        const reads = new FileReader();
        reads.readAsDataURL(oFile);
        reads.onload = () => {
          toast.hide();
          this.cropperImg = reads.result;
          const component = this.$refs['myPopup'];
          component.show();
        };
      } catch (err) {
        const toast = this.$createToast({
          time: 1000,
          type: 'warn',
          txt: '图片上传失败，请重试',
        });
        toast.show();
      }
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
      this.downlaodLock = true;
      setTimeout(() => {
        this.downlaodLock = false;
      }, 5000);
      let canvas = document.querySelector('canvas');
      const cap = new window.CCapture({
        framerate: 40,
        timeLimit: 1.5,
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
    getCropData() {
      this.$refs.cropper.getCropData((data) => {
        this.templateImg = data;
        this.changeImage(data);
        this.$refs['myPopup'].hide();
      });
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
  .cube-extend-popup {
    .cube-extend-popup-content {
      padding: 10px;
    }
  }
  .punch-descript {
    font-size: 30px;
    margin-top: 20px;
  }
  .punch-list {
    padding: 0 20px;
    h2 {
      font-weight: 700;
      text-align: left;
      margin-bottom: 10px;
    }
    margin-bottom: 20px;
  }
  .punch {
    height: 300px;
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
  .cropper-wrap {
    width: 100%;
    min-width: 150px;
    height: 300px;
  }
  // #c {
  //   background: #ccc;
  // }
}
</style>
