```html
<script type="text/javascript">
            var myEditor;

            $(function() {
                myEditor = editormd("my-editormd", {
                    width   : "90%",
                    height  : 800,
                    syncScrolling : "single",
                    path    : "/smart-api/htdocs/mdeditor/lib/",
                    saveHTMLToTextarea : true,

                    emoji: true,//emoji表情，默认关闭
                    taskList: true,
                    tocm: true, // Using [TOCM]
                    tex: true,// 开启科学公式TeX语言支持，默认关闭

                    flowChart: true,//开启流程图支持，默认关闭
                    sequenceDiagram: true,//开启时序/序列图支持，默认关闭,

                    dialogLockScreen : false,//设置弹出层对话框不锁屏，全局通用，默认为true
                    dialogShowMask : false,//设置弹出层对话框显示透明遮罩层，全局通用，默认为true
                    dialogDraggable : false,//设置弹出层对话框不可拖动，全局通用，默认为true
                    dialogMaskOpacity : 0.4, //设置透明遮罩层的透明度，全局通用，默认值为0.1
                    dialogMaskBgColor : "#000",//设置透明遮罩层的背景颜色，全局通用，默认为#fff

                    codeFold: true,

                    imageUpload : true,
                    imageFormats : ["jpg", "jpeg", "gif", "png", "bmp", "webp"],
                    imageUploadURL : "/smart-api/upload/editormdPic/",

                    /*上传图片成功后可以做一些自己的处理*/
                    onload: function () {
                        //console.log('onload', this);
                        //this.fullscreen();
                        //this.unwatch();
                        //this.watch().fullscreen();
                        //this.width("100%");
                        //this.height(480);
                        //this.resize("100%", 640);
                    },

                    /**设置主题颜色*/
                    editorTheme: "pastel-on-dark",
                    theme: "gray",
                    previewTheme: "dark"
                });

            });
        </script>
```