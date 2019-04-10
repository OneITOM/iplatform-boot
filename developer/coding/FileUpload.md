# 文件上传规约

> 作者 王立松

本文档作为文件上传的指导规约为开发人员提供指导性建议，并不作为严格规则要求

## 参数

> 通过配置参数实现对上传文件的限制

```yml
# 支持文件上传，默认true
multipart.enabled: true

# 支持文件写入磁盘的阈值，默认0
multipart.file-size-threshold: 0

# 上传文件的临时目录
#multipart.location: 

# 单个文件的大小，默认100Mb
multipart.maxFileSize: 100Mb

# 单次请求的文件的总大小，默认1000Mb
multipart.maxRequestSize: 1000Mb
```

## 前端

> 文件上传基于jquery组件实现，依赖jquery.ui.widget.js和jquery.fileupload.js

```html
<div layout:fragment="content"
		class="col-sm-9 col-sm-offset-3 col-md-10 col-md-offset-2 main">	
		<input type="file" id="file" name="file">
		<script th:src="@{/js/plugins/jquery/jquery.min.js}"></script>
		<script th:inline="javascript">/*<![CDATA[*/
			
		$(function() {
			$("#file").fileupload({
			    url: /*[[@{/testUpload}]]*/,
			    formData: {"access_token" : [[${access_token}]]},
			    maxFileSize : 50000000, // 50 MB
			    autoUpload: true,
			    done: function(e,_result){
				result = _result.result;
				if(result && result.success){
					alert("文件上传成功,文件Id:"+result.fileId);
				}else{
					alert("上传文件失败");
				}
			    }
			});
		});

		/*]]>*/</script>
</div>
```

## 后端

> 文件接收基于MultipartFile实现

```java
@RequestMapping(value = "/testUpload", method = RequestMethod.POST)
@ResponseBody
public ResponseEntity<Map<String, Object>> testUpload(@RequestParam(value = "file") MultipartFile file, HttpServletRequest request, Principal principal) {
    Map<String, Object> map = new HashMap<>();
    try {
        LOG.info("文件名称={}", file.getOriginalFilename());
        LOG.info("文件大小={}", file.getSize());
        String fileId = dfssClient.add(file.getOriginalFilename(), file.getInputStream());
        map.put("success", true);
        map.put("fileId", fileId);
    } catch (Exception e) {
        LOG.error("文件上传失败", e);
        map.put("success", false);
    }
    return new ResponseEntity<>(map, HttpStatus.OK);
}
```
