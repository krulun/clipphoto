# clipphoto
头像裁剪上传
做移动端的时候，各位可能会遇到用户上传头像，而前端的input：file，如果上传正方形图片还好，但是如果是矩形的话，在头像的显示这块，就会显的很不和谐。

前端根据矩形的图片，实现裁剪成正方形，并把裁剪后的文件上传给后台。

1.初始页面如下，点击中间默认头像触发input:file点击，弹出选择文件框.





2.开始选择照片，如下，为选择的照片，和选择照片后的效果图：





3.此时图片的高度大于宽度，因此可以上下滚动，选择想要裁剪的范围，然后点击选取（下图从地铁导航处开始裁剪）。





此时，已经裁剪到正方形图片，可以重新拍摄或者上传给后台。

下面是代码


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


第一步：
 //用户上传图片后触发file:onchange
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
   checkClip(e) {//滑动图片适合位置后点击选取触发该方法，获取滚动的x和Y的距离传给setPhotoCanvas，以便canvas裁剪。
      let sTop =  this.$refs['clip_img'].scrollTop;
      let sLeft =  this.$refs['clip_img'].scrollLeft;
      this.setPhotoCanvas(sTop, sLeft);
    },
   setPhotoCanvas(sTop, sLeft) {//跟进图片x和Y方向滚动的距离，从而裁剪相应的图片  
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
    dataURItoBlob: function(dataURI) {/把base64 的图片uri转换成二进制大对象，方便传给后台 
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

这就已经大功告成啦，难点主要在获取图片滚动的距离后怎么进行裁剪，

ctx.drawImage(img, sx, sy, sw, sh, 0, 0, short, short);

这行代码，起着决定作用，img代表图片文件，sx代表souce-x，既源图片X轴从哪里开始裁剪，sy代表souce-y，既源图片Y轴从哪里开始裁剪。

sw和sh分别代表,souce-width和souce-height。即原图片x和Y方向裁剪的宽度和长度。

0,0代表重新画的image从canvas的0,0坐标开始。既左上角。

short,short，short来源于图片长度和宽度的比较，获取较短的那个边的长度。

然后就可以根据这几个参数canvas绘画出对应的正方形图片了。

最后由于上传base64较大，就改为二进制上传,转换为bolb对象。

