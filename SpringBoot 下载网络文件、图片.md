---
tags:
- Java/Spring
---

我是文件保存在服务器上，文件的信息保存在数据库中，服务器上的文件是在一个文件夹里的，用的时间戳命名，文件的真实名字是在数据库中保存的，我想下载的时候使用数据库里的文件名下载，并且这个文件是个网络文件。

![](assets/SpringBoot%20下载网络文件、图片/image-20240429114548155.png)


Controller层：

id就是上图中的文件id，这里直接调用Service层的方法

```java
    /**
     * 文件下载
     *
     * @param response response
     * @param id       文件id
     * @return 统一返回对象
     */
    @RequestMapping("/download")
    public ReturnResult download(HttpServletResponse response, @RequestParam("id") Integer id) {
        datumService.download(response, id);
        // 响应对象无需返回，这里是我的统一返回对象，如果没有额外要返回的内容则不用返回
        return ReturnResult.success();
    }
```

Service层：

```java
    /**
     * 文件下载
     *
     * @param response response
     * @param id       文件id
     */
    public void download(HttpServletResponse response, Integer id) {
        // 这里是根据id查询数据库中的文件信息
        Datum datum = datumDao.qryDatumById(id);
        if (datum == null) {
            // 如果没查到，直接返回错误信息
            throw new ArborException(ArborExceptionEnum.NO_MESSAGE);
        }
 
        // 将拿到的文件的网络地址和保存到数据库中的文件名，并且获取后缀，进行字符串拼接，得到一个完整的文件名
        String[] footerArray = datum.getAddress().split("\\.");
        String fileName = datum.getName() + "." + footerArray[footerArray.length - 1];
 
        URL url;
        URLConnection conn;
        InputStream inputStream = null;
        try {
            // 这里填文件的url地址
            url = new URL(datum.getAddress());
            conn = url.openConnection();
            inputStream = conn.getInputStream();
            response.setContentType(conn.getContentType());
            // 使用URLEncoder.encode(fileName, "UTF-8")将文件名设置为UTF-8的编码格式
            response.setHeader("Content-Disposition", "attachment; filename="
                    + URLEncoder.encode(fileName, "UTF-8"));
            byte[] buffer = new byte[1024];
            int len;
            OutputStream outputStream = response.getOutputStream();
            while ((len = inputStream.read(buffer)) > 0) {
                outputStream.write(buffer, 0, len);
            }
        } catch (IOException e) {
            // 如果发生异常，返回自定义的下载失败的提示信息
            e.printStackTrace();
            throw new ArborException(ArborExceptionEnum.DOWNLOAD_FAILED);
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```


dao层就不展示了，就是查询一条数据。

**返回前端的文件名必须进行URL编码：** 网络传输只能传输特定的几十个字符，需要将汉字、特殊字符等经过Base64等编码来转化为特定字符，从而进行传输，而不会乱码

```java
URLEncoder.encode(fileName, "UTF-8"))
```