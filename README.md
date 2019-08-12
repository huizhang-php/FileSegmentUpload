# 文件分段上传

### 1.安装
```
composer require huizhang/file-segment-upload
```

### 2.代码示例

`html`

```html
<!doctype html>
<html lang="en">
<body>
<form action="./Exampl1.php">
    <input type="file" name="file" id="file">
</form>
<script>
  var uploadObj = new SegmentUpload();
  var fileDom = document.getElementById("file");

  fileDom.onchange = function(){
    uploadObj.addFileAndSend(this);
  };

  function SegmentUpload() {
    const length = 1024*10; // 文件包大小
    const requestUrl = './Exampl1.php';
    const requestType = 'POST'; // 请求方式

    var request = new XMLHttpRequest();
    var start = 0; // 截取下标开始位置
    var end = length; // 截取下标结束位置
    var nowPackage=''; // 文件包
    var nowPackageNum = 1; // 当前包数
    var totalPackageNum = 0; // 总共包数
    var file = null;

    this.addFileAndSend = function(that){
      file = that.files[0];
      totalPackageNum = Math.ceil(file.size / length);
      blob = cutFile();
      sendFile(blob);
      nowPackageNum += 1;
    };

    cutFile = function (){
      nowPackage = file.slice(start, end);
      start = end;
      end = start + length;
    };

    sendFile = function (){
      var formData = new FormData();
      formData.append('file',nowPackage);
      formData.append('blob_num',nowPackageNum);
      formData.append('total_blob_num',totalPackageNum);
      formData.append('file_name',file.name);
      request.open(requestType, requestUrl, false);
      request.onreadystatechange = function () {
        var t = setTimeout(function(){
          if(start < file.size){
            blob = cutFile(file);
            sendFile(nowPackage,file);
            nowPackageNum += 1;
          }else{
            setTimeout(t);
          }
        },1000);
      };
      request.send(formData);
    }
  }

</script>
</body>
</html>

```

`php`
```php
<?php
require_once '../vendor/autoload.php';

use Huizhang\FileSegmentUpload\FileSegmentUpload;

$obj = new FileSegmentUpload();

$res = $obj->upload([
    'tmp_name' => $_FILES['file']['tmp_name'], // 文件内容
    'now_package_num' => $_POST['blob_num'], // 当前文件包数量
    'total_package_num' => $_POST['total_blob_num'], // 文件包总量
    'file_name' => $_POST['file_name'], // 文件名称(唯一)
    'file_path' => './upload', // 文件存放路径
    'clear_interval_time' => 60, // 清理临时文件间隔，默认五分钟
    'is_continuingly' => true // 是否断点续传，默认为true
]);
var_dump(json_encode([200,$res]));die;
```

### 3.test过程

![](./test/static/1.png)

![](./test/static/2.png)

![](./test/static/3.png)
