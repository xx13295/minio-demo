# minio-demo

```

      <dependency>
            <groupId>plus.ojbk</groupId>
            <artifactId>minio-spring-boot-starter</artifactId>
            <version>1.0.2</version>
        </dependency>

```


```

#--------
# minio
#--------
file:
  minio:
    enable: true
    endpoint: http://172.16.1.240:9000
    bucket: test
    access-key: test
    secret-key: test123456789

```


```
/**
     * 上传文件
     * @param multipartFile
     * @return
     * @throws Exception
     */
    @PostMapping("/upload")
    public Object upload(@RequestParam("file") MultipartFile multipartFile) throws Exception{
        ObjectWriteResponse res = minioTemplate.putObject(multipartFile);

        String path = res.object();
        // String bucket = res.bucket();
        log.info("File Path = {}", path);
        // TODO 你具体的业务 比如存储 这个path 到数据库等
        return "ok";
    }

    /**
     * 删除文件
     * @param object
     * @return
     */
    @GetMapping("/delete")
    public String delete(@RequestParam("path") String object) {
        minioTemplate.deleteObject(object);
        return "ok";
    }


    /**
     * 获取文件url
     * @param object
     * @return
     */
    @GetMapping("/get")
    public String get(@RequestParam("path") String object) {
        //默认 1 小时
        String url = minioTemplate.getObject(object);
        //自定义时间
        //String url = minioTemplate.getObject(name, 10, TimeUnit.MINUTES);
        log.info("Get File Url = {}", url);
        return url;
    }

    /**
     * 下载文件
     * @param object
     * @param response
     * @throws Exception
     */
    @GetMapping("/download")
    public void download(@RequestParam("path") String object, HttpServletResponse response) throws Exception {
        InputStream in = minioTemplate.getObjectInputStream( object);
        OutputStream out = response.getOutputStream();
        byte[] buf = new byte[1024];
        int len = 0;
        response.reset();
        response.setContentType("application/x-msdownload");
        response.setHeader("Content-Disposition", "attachment; filename=" + URLEncoder.encode(MinioUtils.getFileName(object), "UTF-8"));
        while ((len = in.read(buf)) > 0) {
            out.write(buf, 0, len);
        }
        in.close();
        out.close();
    }

    /***
     * 创建新的 存储桶
     * 后续使用 其他方法都要指定 你新建的这个bucket
     * 否者为默认的 yml中配置的bucket
     * @param bucket
     * @return
     */
    @GetMapping("/bucket")
    public String bucket(@RequestParam("bucket") String bucket) {

        minioTemplate.createBucket(bucket);

        return "ok";
    }

```
