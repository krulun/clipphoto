<template>
  <div class="photo-panel">
    <back-btn/>
    <h3 class="tc">头像</h3>

    <div class="photo-box bg-w">
      <div class="license-box">
        <div class="bus-license img-box">
          <img ref="avatar_img" src="@/assets/avater.png" class="avatar" alt="头像" @click="upload('avatar')">
          <input ref="avatar" :accept="accept" type="file" name="" class="hide" @change="setPhoto">
        </div>
        <p v-show="!avatar" class="lh-3 tc ">点击拍摄头像</p>
        <p v-show="avatar" class="lh-3 tc"><span class="f-blue" @click="$refs.avatar.click();currentType='avatar'">重新拍摄 </span> 头像 </p>
      </div>
    </div>

    <div v-show="show_clip_img" class="clip_img_box">
      <div class="shade top_shade"/>
      <div ref="clip_img" class="clip_img">
        <img ref="clip_avatar" :class="avater_width_larger_height? 'w_l_h' : ''" src="@/assets/license2.png" class="clip_avatar" alt="">
      </div>
      <div class="shade bottom_shade">
        <div>
          <div class="cancel_clip clip_btn fl" @click="cancelClip">取消</div>
          <div class="check_clip clip_btn fr" @click="checkClip">选取</div>
        </div>
      </div>
    </div>
    <van-button type="info" size="large" class="upload_btn pdbox mgbox bottom-btn" @click="submit">上传照片</van-button>

  </div>
</template>

<script>
import { ImagePreview } from 'vant'//想要图片点击全屏查看的效果，就引进
import Lrz from 'lrz'//这里使用Lrz来防止图片选中后出现旋转情况
import { browserType } from '@/utils/index'//判断浏览器类型

export default {
  name: 'Idphoto',
  data() {
    return {
      loading: false,
      avatar: '',
      maxSize: 1024 * 1024,
      ratio: 1,
      accept: 'image/png,image/gif,image/jpeg,image/jpg',
      maxWidth: 1000,
      show_clip_img: false,
      avater_width_larger_height: false
    }
  },
  created() {
    document.title = '上传头像'
    this.judgeBrowser();
  },
  mounted() {
    this.$refs['avatar_img'].src = JSON.parse(localStorage.getItem('info')).avatar;
  },
  methods: {
    judgeBrowser() {
      let browser = browserType();
      if (browser.isAndroid && browser.isWechat) {
        this.accept = 'image/*'//兼容安卓微信端的情况
      }
    },
    loadend() {
      if (this.human_photo) {
        this.loading = false
      }
    },
    upload(type) {
      if (this[type]) {
        ImagePreview([
          this.$refs['avatar_img'].src
        ])
      } else {
        this.$refs[type].click();
      }
      this.currentType = type;
    },
    setPhoto(e) {
      let _this = this;
      let file = e.target.files[0];
      if (!file) return ''
      //大于6兆，不允许上传
      if (file.size > this.maxSize * 6) {
        this.$toast('图片过大，不能大于6M')
        return;
      }
      if (file.size > this.maxSize * 3) {
        this.ratio = 0.5;
      } else if (file.size > this.maxSize * 4) {
        this.ratio = 0.4;
      }
      Lrz(file, { width: 1024, quality: this.ratio })
        .then((rst) => {
          // 把处理的好的图片给用户看看呗
          // _this.$refs['avatar_img'].src = rst.base64;
          _this.$refs['clip_avatar'].src = rst.base64;
          _this.$nextTick(() => {
            _this.avater_width_larger_height = _this.$refs['clip_avatar'].height < _this.$refs['clip_avatar'].width
          })
          _this.show_clip_img = true;
          return rst;
        }).then((rst) => {
          this[this.currentType] = rst.file;
        })
    },
    dataURItoBlob: function(dataURI) {
    // convert base64/URLEncoded data component to raw binary data held in a string
      let byteString;
      if (dataURI.split(',')[0].indexOf('base64') >= 0) {
        byteString = atob(dataURI.split(',')[1]);
      } else {
        byteString = unescape(dataURI.split(',')[1]);
      }

      // separate out the mime component
      let mimeString = dataURI.split(',')[0].split(':')[1].split(';')[0];

      // write the bytes of the string to a typed array
      let ia = new Uint8Array(byteString.length);
      for (let i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i);
      }
      return new Blob([ia], { type: mimeString });
    },
    setPhotoCanvas(sTop, sLeft) {
      let reader = new FileReader();
      reader.onloadend = (e) => {
        let img = new Image();
        img.onload = () => {
          let w = Math.min(this.maxWidth, img.width);
          let h = img.height * (w / img.width);
          let [sw, sh, short, sx, sy] = [img.width, img.height, 0, 0, 0, 0]
          console.log(document.body.clientWidth)
          if (w < h) {
            short = w;
            sy = (sTop / document.body.clientWidth) * img.width
            sh = sw;
          } else { //w>h
            short = h;
            sx = (sLeft / document.body.clientWidth) * img.height
            sw = sh;
          }
          let canvas = document.createElement('canvas');
          let ctx1 = canvas.getContext('2d');
          // 设置 canvas 的宽度和高度
          canvas.width = short;
          canvas.height = short;
          ctx1.drawImage(img, sx, sy, sw, sh, 0, 0, short, short);
          let base64 = canvas.toDataURL('image/png', this.ratio);
          // 插入到预览区
          debugger;
          console.log(base64);
          this.$refs[this.currentType + '_img'].src  =  base64;
          this.show_clip_img = false;
          this[this.currentType] = this.dataURItoBlob(base64);
          console.log('size-------------------' + this[this.currentType].size)
        };
        img.src = e.target.result;
      }
      reader.readAsDataURL(this[this.currentType]);
    },
    submit() {
      let formData = new FormData();
      if (!this.avatar) {
        return this.$toast('请上传头像');
      }
      formData.append('upload', this.avatar)

      this.$req({
        url: '/api/user/uploadavatar',
        method: 'post',
        data: formData
      }).then((res) => {
        if (res.code === 1) {
          this.$router.push({ path: '/index' });
        }
      }).catch((err) => {
        console.log(err)
      })
    },
    checkClip(e) {
      let sTop =  this.$refs['clip_img'].scrollTop;
      let sLeft =  this.$refs['clip_img'].scrollLeft;
      this.setPhotoCanvas(sTop, sLeft);
    },
    cancelClip(e) {
      this.show_clip_img = false;
      this[this.currentType] = '';
      this.$refs['avatar'].value = '';
    }
  }
}
</script>

<style lang="scss" scoped>
    .photo-panel{
        overflow: hidden;
        h3{
            margin: 1rem auto 0.1rem;
        }
        .photo-box{
            margin-top: 0.5rem;
            color: #333333;
            &>div{
                border-bottom: 1px solid #CCCCCC;
                height: 6.28rem;
                overflow: hidden;
            }
            &>div:last-child{
                border-bottom: none;
            }
            .img-box{
								width: 5rem;
								height: 3.68rem;
								overflow: hidden;
								text-align: center;
                margin: 0.8rem auto 0;
                img{
                    height:100%;
                }
            }
        }
        .upload_btn{
            margin-top: 0.8rem;
        }
    }
    .clip_img_box{
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: #000;
      display: flex;
      flex-direction: column;
      z-index: 1;
      .shade{
        background: rgba(0,0,0,0.7);
        z-index: 10;
      }
      .top_shade{
        height: 10%;
        border-bottom: 1px solid #fff;
      }
      .bottom_shade{
        flex: 1;
        border-top: 1px solid #fff;
        position: relative;
        &>div{
          position: absolute;
          bottom: 0;
          left: 0;
          width: 100%;
          height: 1rem;
          background: rgba(86, 86, 86, 0.4);
          line-height: 1rem;
          padding: 0 0.4rem;
        }
        .clip_btn{
          // position: absolute;
          // bottom: 0.8rem;
          color: #fff;
          // &.cancel_clip{
          //   left: 0.4rem;;
          // }
          // &.check_clip{
          //   right: 0.4rem;;
          // }
        }
      }
      .clip_img_faker{
        border: 1px solid #fff;
        width: 100%;
        height: 7.5rem;
        position: absolute;
        top: 10%;
        z-index: 10;
      }
      .clip_img{
        width: 100%;
        height: 7.5rem;
        overflow-y:scroll;
        img{
          width:100%;
          height: auto;
          position: relative;
          display: block;
          // transition: 1s; 这里因为宽度和高度不是固定写死，因此transition不支持!anmation也不支持
          &.w_l_h{
            width: initial;
            height: 100%;
            // transition: 1s;
            // animation: changeLarge 3s;
            // animation-iteration-count: 1;
          }
        }
      }
    }
</style>

