<template>
  <div class="goodListBox">
    <ul class="but-upload">
      <li v-for="img in imageOriginalUrl">
        <img :src="img" alt="" />
        <i class="delimg" @click="delupImg(img)"></i>
      </li>
      <li class="li-upfile">
        <input type="file" class="upfile" @change="upfile"/>
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    data() {
      return {
        //上传图片集合——原图
        imageOriginalUrl:[],
        //上传图片集合——缩略图
        imageNarrowUrl:[],
      }
    },
    methods: {
      //上传图片
      upfile:function(){
        if(this.imageOriginalUrl && this.imageOriginalUrl.length>2){
          alert("最多上传3张图");
          return false;
        }
        var file = $(".upfile")[0].files[0];
        //这里我们判断下类型如果不是图片就返回 去掉就可以上传任意文件
        if(!/image\/\w+/.test(file.type)){
          alert("请确保文件为图像类型");
          return false;
        }
        var _this = this;
        var reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = function(e){
          var result = this.result;
          result = result.substring(result.indexOf(",")+1,result.length-1);
          axios.post(contextPath.contextPath+'waevaluate/evaluateUpImg', $.param({
            from: 'WX',
            imgData:result,
            imgName: file.name
          }))
            .then((data) => {
              _this.imageOriginalUrl.push(data.data.imageIconOriginal);
              _this.imageNarrowUrl.push(data.data.imageIconDispose);
              var msg = data.data.resmsg
              TipFunction.loadingMessage(msg);
              setTimeout(function(){
                TipFunction.hideLoading();
              },2000)
            })
            .catch((response) => {
              alert("上传出错了");
              alert(response);
              console.log(response);
            });
        }
      },
      //删除图片
      delupImg:function(o){
        if(o){
          var that = this;
          this.imageOriginalUrl.forEach(function(d,i){
            if(d==o){
              //创建删除图片参数
              var parms = {
                from: 'WX',
                imageIconOriginal: that.imageOriginalUrl[i],
                imageIconDispose: that.imageNarrowUrl[i]
              }
              //从数组中删除图片
              that.imageOriginalUrl.splice(i,1);
              that.imageNarrowUrl.splice(i,1);
              //调服务删除图片
              axios.get(contextPath.contextPath+'waevaluate/delEvaluateUpImg', {
                params: parms
              })
                .then((res) => {
                  console.log(res);
                  var oData = res.data;
                  if(oData.rescode === '00000'){
                    TipFunction.loadingMessage(oData.resmsg);
                    setTimeout(function(){
                      TipFunction.hideLoading();
                    },2000)
                  }
                })
                .catch((response) => {

                });
            }
          })
        }
      }
    }
  };
</script>

<style>
</style>
