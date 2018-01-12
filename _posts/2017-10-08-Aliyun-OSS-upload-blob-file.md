---
layout: post
title: aliyun OSS 上传 blob 文件
category : [学习问题记录]
tagline: "Supporting tagline"
tags : [aliyun, OSS, blob]
---

# aliyun OSS 上传 blob 文件 
使用 cropper 图片裁剪插件上传时，前段将裁剪后图片转成了 blob 文件。现在要将该 blob 文件上传到阿里云的 OSS 上。  

<!--break-->

页面(使用 cropper 插件需要在页面中引入 cropper.css 和 copper.js):    

``` 
<form action="">
    <input type="hidden" name="url" id="url"/>
    <div class="row">
      <div class="col-md-6">
        <label title="Upload image file" for="inputImage" class="btn btn-primary">
        <input type="file" accept="content/images/*" name="file" id="inputImage"
                                                     class="hide">选择图片
        </label>
        <button type="button" id="uploadLogo" class="btn btn-primary">上传</button>
        <div class="image-crop m-t-xs" style="max-height: 300px;">
            <img width="300" height="300">
        </div>
      </div>
    </div>
</form>    
```

js:   
```  
// 从本地选择裁剪图片
var $inputImage = $('#inputImage');
if (window.FileReader) {
    $inputImage.change(function () {
        var fileReader = new FileReader(),
            files = this.files,
            file;

        if (!files.length) {
            return;
        }

        file = files[0];

        if (/^image\/\w+$/.test(file.type)) {
            fileReader.readAsDataURL(file);
            fileReader.onload = function () {
                $inputImage.val('');
                $image.cropper('reset', true).cropper('replace', this.result);
            };
            $('#cropperModal').modal('show');
        } else {
            showMessage('请选择图片.');
        }
    });
} else {
    $inputImage.addClass('hide');
}
       
// 初始化裁剪插件
var $image = $('.image-crop > img');  
$($image).cropper({
	viewMode: 1,
    aspectRatio: 1,
    preview: '.img-preview',
    done: function (data) {
    // Output the result data for cropping image.
    }
}); 
              
// 裁剪图片并上传
$('#uploadLogo').on('click', function () {
    var dataUrl = $('.image-crop > img');
    // canvas 对象
    var cropper = dataUrl.cropper('getCroppedCanvas');
    // canvas 转 base64
    // var imgUrl = dataUrl.toDataURL("image/png");
    // var img = document.createElement("inputImage");
    // img.src = imgUrl;
    // document.getElementById("logoImg").appendChild(img);

    //可以直接保存 DataURL,这里保存了base64文件的文件名
    cropper.toBlob(function (blob) {
        var formData = new FormData();
        formData.append('file', blob);
        $.ajax('/imgCut', {
            method: "POST",
            data: formData,
            processData: false,
            contentType: false,
            success: function (res) {
                $('#url').val(res);
                toastr.success("图片上传成功");
            },
            error: function () {
                toastr.success("图片上传失败");
            }
        });
    });
}); 
```

Java 后台代码:   
``` 
public class AliyunOssUtil {

    private Log log = LogFactory.getLog(AliyunOssUtil.class);

    private OSSClient ossClient;

    private String bucket;

    private String key;

    /**
     * 初始化
     */
    public AliyunOssUtil(String endpoint, String accessKeyId, String accessKeySecret, String bucket, String key) {
        ossClient = new OSSClient(endpoint, accessKeyId, accessKeySecret);
        this.bucket = bucket;
        this.key = key;
    }

    /**
     * 销毁OssClient
     */
    public void destory() {
        ossClient.shutdown();
    }

    /**
     * 上传图片到阿里云 OSS 上
     *
     * @param file
     * @return
     */
    public String uploadImgToOss(MultipartFile file, Boolean compressed) {
        String originalFileName = file.getOriginalFilename();
        String fileExtName = getExtName(originalFileName);
        String savedFileName = UUID.randomUUID() + fileExtName;
        try {
            InputStream inputStream = file.getInputStream();
            this.uploadImageToOSS(inputStream, savedFileName, compressed);
            return savedFileName;
        } catch (Exception e) {
            log.error(e.getMessage());
            return "";
        }
    }

    /**
     * 上传文件到阿里云 OSS 上
     *
     * @param file
     * @return
     */
    public String uploadFileToOss(MultipartFile file) {
        String originalFileName = file.getOriginalFilename();
        String fileExtName = getExtName(originalFileName);
        String savedFileName = UUID.randomUUID() + "." + fileExtName;
        try {
            InputStream inputStream = file.getInputStream();
            this.uploadFileToOSS(inputStream, savedFileName);
            return savedFileName;
        } catch (Exception e) {
            log.error(e.getMessage());
            return "";
        }
    }

    /**
     * 上传文件到阿里云 OSS 上， 如果同名文件会覆盖服务器上的
     *
     * @param file
     * @param fileName
     * @return
     */
    public String uploadFileToOSS(MultipartFile file, String fileName) {
        InputStream inputStream = null;
        try {
            inputStream = file.getInputStream();
            return uploadFileToOSS(inputStream, fileName);
        } catch (IOException e) {
            e.printStackTrace();
            return "";
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 上传文件到阿里云 OSS 上， 如果同名文件会覆盖服务器上的
     *
     * @param inputStream 文件流
     * @param fileName    文件名称 包括后缀名
     * @return 出错返回"" ,唯一MD5数字签名
     */
    public String uploadFileToOSS(InputStream inputStream, String fileName) {
        String ret = "";
        try {
            //创建上传Object的Metadata
            ObjectMetadata objectMetadata = new ObjectMetadata();
            objectMetadata.setContentLength(inputStream.available());
            objectMetadata.setCacheControl("no-cache");
            objectMetadata.setHeader("Pragma", "no-cache");
            objectMetadata.setContentType(getContentType(getExtName(fileName)));
            objectMetadata.setContentDisposition("inline;filename=" + fileName);
            //上传文件
            PutObjectResult putResult = ossClient.putObject(bucket, key + fileName, inputStream, objectMetadata);
            ret = putResult.getETag();
        } catch (IOException e) {
            log.error(e.getMessage(), e);
        } finally {
            try {
                if (inputStream != null) {
                    inputStream.close();
                }
            } catch (IOException e) {
                log.error(e.getMessage());
            }
        }
        return ret;
    }

    /**
     * @param file
     * @param fileName
     * @param compressed 是否压缩图片
     * @return
     */
    public String uploadImageToOSS(MultipartFile file, String fileName, Boolean compressed) {
        InputStream inputStream = null;
        try {
            inputStream = file.getInputStream();
            return uploadImageToOSS(inputStream, fileName, compressed);
        } catch (IOException e) {
            e.printStackTrace();
            return "";
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * @param inputStream
     * @param fileName
     * @param compressed  是否压缩图片
     * @return
     */
    public String uploadImageToOSS(InputStream inputStream, String fileName, Boolean compressed) {

        if (compressed) {
            try {
                //压缩图片(参考: https://www.cnblogs.com/simpledev/p/3980972.html)
                BufferedImage bufferedImage = ImageIO.read(inputStream);
                bufferedImage = Thumbnails.of(bufferedImage)
                    .size(bufferedImage.getWidth(), bufferedImage.getHeight())
                    .outputQuality(0.9).asBufferedImage();

                //将 BufferedImage 转换成 InputStream
                ByteArrayOutputStream bos = new ByteArrayOutputStream();//存储图片文件byte数组
                ImageOutputStream ios = ImageIO.createImageOutputStream(bos);
                ImageIO.write(bufferedImage, getExtName(fileName), ios); //图片写入到 ImageOutputStream

                inputStream = new ByteArrayInputStream(bos.toByteArray());

            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        return uploadFileToOSS(inputStream, fileName);
    }

    public String getExtName(String fileName) {
        return fileName.substring(fileName.lastIndexOf(".") + 1).toLowerCase();
    }

    /**
     * Description: 判断 OSS 服务文件上传时文件的 contentType
     *
     * @param fileExt 文件后缀
     * @return String
     */
    public static String getContentType(String fileExt) {
        if (fileExt.equalsIgnoreCase("bmp")) {
            return "image/bmp";
        }
        if (fileExt.equalsIgnoreCase("gif")) {
            return "image/gif";
        }
        if (fileExt.equalsIgnoreCase("jpeg") ||
            fileExt.equalsIgnoreCase("jpg") ||
            fileExt.equalsIgnoreCase("png")) {
            return "image/jpeg";
        }
        if (fileExt.equalsIgnoreCase("html")) {
            return "text/html";
        }
        if (fileExt.equalsIgnoreCase("txt") ||
            fileExt.equalsIgnoreCase("blob")) {
            return "text/plain";
        }
        if (fileExt.equalsIgnoreCase("vsd")) {
            return "application/vnd.visio";
        }
        if (fileExt.equalsIgnoreCase("pptx") ||
            fileExt.equalsIgnoreCase("ppt")) {
            return "application/vnd.ms-powerpoint";
        }
        if (fileExt.equalsIgnoreCase("docx") ||
            fileExt.equalsIgnoreCase("doc")) {
            return "application/msword";
        }
        if (fileExt.equalsIgnoreCase("xml")) {
            return "text/xml";
        }
        return "image/jpeg";
    }

    //getter and setter 
}
```

``` 
/**
* 上传裁剪图片
*
* @param file
* @return
*/
@PostMapping("/imgCut")
public ResponseEntity imgCut(@RequestParam("file") MultipartFile file) {
    String fileName;

    try {
        fileName = new AliyunOssUtil(endpoint, accessKeyId, accessKeySecret, bucket, key).uploadFileToOss(file);
    } catch (Exception e) {
        e.printStackTrace();
        return ResponseEntity.badRequest().build();
    }

    return ResponseEntity.ok(fileName);
}
```

前台直接在 `<img>` 标签的 src 属性上使用 blob 文件在阿里云 OSS 上的访问路径即可，例如： 
``` 
<img src="https://bucket.oss-cn-shanghai.aliyuncs.com/89b3a8eb-1086-40fc-23ac-9593bf6ae111.blob" width="300" height="300">
```

> 如果可以的话，不需要将 blob 上传到阿里云，直接保存到数据库的字段中。  

## 补充：将 blob 文件上传到本地文件系统   
将图片上传到本地，然后在页面通过请求将 blob 转成 二进制流输出到页面： 
``` 
/**
 * 根据文件名在页面显示图片
 *
 * @param fileName
 * @param response
 */
@RequestMapping("/croppedImages/{fileName:.+}")
public void blobToImage(@PathVariable("fileName") String fileName, HttpServletResponse response) {
    try {
        if (StringUtils.isEmpty(fileName)) {
            return;
        }
        response.setHeader("Content-Type", "image/png");
        response.getOutputStream().write(file2ByteArray(imgUploadPath + fileName));
    } catch (Exception e) {
        e.printStackTrace();
    }
} 

/**
 * 文件转字节数组
 *
 * @param path
 * @return
 */
public static byte[] file2ByteArray(String path) {
    File file = new File(path);
    if (!file.exists()) {
        return null;
    }
    FileInputStream stream = null;
    ByteArrayOutputStream out = null;
    try {
        stream = new FileInputStream(file);
        out = new ByteArrayOutputStream(1000);
        byte[] b = new byte[1000];
        int n;
        while ((n = stream.read(b)) != -1) {
            out.write(b, 0, n);
        }
        return out.toByteArray();// 此方法大文件OutOfMemory
    } catch (Exception e) {
        System.out.println(e.toString());
    } finally {
        try {
            stream.close();
            out.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    return null;
}
``` 

页面显示图片： 
``` 
<img src="/croppedImages/89b3a8eb-1086-40fc-23ac-9593bf6ae111.blob" width="300" height="300">
```

上面代码： `response.setHeader("Content-Type", "image/png");` 后面的 `image/png` 不能改成其他的 `Content-Type` 。 
Cropper 插件在转换时首先把图像转成 PNG 数据，然后再把得到的二进制的 PNG 数据转成纯 ASCII 的 base64 编码的字符串,所以后台拿到的 base64 数据是从 png 格式的图片获取到的。 
如果不是 `image/png` ，在 IE 浏览器显示上传的非 png 格式的图片时,控制台会报错:  
``` 
IE 无法解码 URL 处图像
```
错误表示设置的图片格式与图片原来的格式不一致。 

> https://fanghe1995.github.io/2017/08/25/IE-DOM7009/ 
>
> https://www.cnblogs.com/zichi/p/5070895.html 
> 
> https://www.cnblogs.com/hustskyking/p/data-uri.html 