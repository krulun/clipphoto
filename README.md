# clipphoto
头像裁剪上传
做移动端的时候，各位可能会遇到用户上传头像，而前端的input：file，如果上传正方形图片还好，但是如果是矩形的话，在头像的显示这块，就会显的很不和谐。

前端根据矩形的图片，实现裁剪成正方形，并把裁剪后的文件上传给后台。

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


难点主要在获取图片滚动的距离后怎么进行裁剪，

ctx.drawImage(img, sx, sy, sw, sh, 0, 0, short, short);

这行代码，起着决定作用，img代表图片文件，sx代表souce-x，既源图片X轴从哪里开始裁剪，sy代表souce-y，既源图片Y轴从哪里开始裁剪。

sw和sh分别代表,souce-width和souce-height。即原图片x和Y方向裁剪的宽度和长度。

0,0代表重新画的image从canvas的0,0坐标开始。既左上角。

short,short，short来源于图片长度和宽度的比较，获取较短的那个边的长度。

然后就可以根据这几个参数canvas绘画出对应的正方形图片了。

最后由于上传base64较大，就改为二进制上传,转换为bolb对象。

