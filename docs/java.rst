Java
=================

如果使用maven，可以加入如下依赖::

	<dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>5.7.22</version>
    </dependency>
    <dependency>
        <groupId>org.bouncycastle</groupId>
        <artifactId>bcprov-jdk15to18</artifactId>
        <version>1.71</version>
    </dependency>


示例程序
------------------

java::


    import cn.hutool.core.io.IoUtil;
    import cn.hutool.core.util.IdUtil;
    import cn.hutool.crypto.asymmetric.SM2;
    import cn.hutool.http.HttpRequest;
    import cn.hutool.http.HttpResponse;
    import cn.hutool.http.HttpUtil;
    import cn.hutool.json.JSON;
    import cn.hutool.json.JSONUtil;
    import org.junit.Test;

    import java.io.*;
    import java.nio.charset.StandardCharsets;
    import java.util.*;
    import java.util.regex.Matcher;
    import java.util.regex.Pattern;

    public class ApiRequestTest {

        static class EvidenceHashParam {
            private String fileLabel;
            private List<HashInfo> list;
            static class HashInfo {
                private String filename;
                private String fileHash;

                public String getFilename() {
                    return filename;
                }

                public void setFilename(String filename) {
                    this.filename = filename;
                }

                public String getFileHash() {
                    return fileHash;
                }

                public void setFileHash(String fileHash) {
                    this.fileHash = fileHash;
                }
            }

            public String getFileLabel() {
                return fileLabel;
            }

            public void setFileLabel(String fileLabel) {
                this.fileLabel = fileLabel;
            }

            public List<HashInfo> getList() {
                return list;
            }

            public void setList(List<HashInfo> list) {
                this.list = list;
            }
        }

        static class EvidenceFileParam {
            private String fileLabel;
            private List<Long> files;

            public String getFileLabel() {
                return fileLabel;
            }

            public void setFileLabel(String fileLabel) {
                this.fileLabel = fileLabel;
            }

            public List<Long> getFiles() {
                return files;
            }

            public void setFiles(List<Long> files) {
                this.files = files;
            }
        }

        private String uri = "http://127.0.0.1:18848/api";

        /**
         *
         * @throws Exception
         */
        @Test
        public void detail() throws Exception {
            String apiName = "/evidence/detail";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            Map<String ,Object> body = new HashMap<>();
            body.put("attestationId","did:bid:efaE9e45apUbuA87y7Y6zjMTaGfHt7WX");
            httpRequest.body(JSONUtil.toJsonStr(body));
            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void list() throws Exception {
            // API path
            String apiName = "/evidence/list";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            Map<String ,Object> body = new HashMap<>();
    //        body.put("attestationId","");
            httpRequest.body(JSONUtil.toJsonStr(body));
            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void hash() throws Exception {
            // API path
            String apiName = "/evidence/hash";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            List<EvidenceHashParam.HashInfo> list = new ArrayList<>();
            EvidenceHashParam.HashInfo hashInfo1 = new EvidenceHashParam.HashInfo();
            hashInfo1.setFilename("test1");
            hashInfo1.setFileHash("98df1f1dfb3b1a123c1517912dc70447aa61c6be532ac99de973abb6219e1653");
            list.add(hashInfo1);
            EvidenceHashParam evidenceHashParam = new EvidenceHashParam();
            evidenceHashParam.setFileLabel("标签");
            evidenceHashParam.setList(list);
            httpRequest.body(JSONUtil.toJsonStr(evidenceHashParam));
            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }

        @Test
        public void file() throws Exception {
            // API path
            String apiName = "/evidence/file";
            HttpRequest httpRequest = createRequestPost(apiName);
            // 构建请求参数
            List<Long> list = new ArrayList<>();
            list.add(1544567382363930624L);
            EvidenceFileParam evidenceFileParam = new EvidenceFileParam();
            evidenceFileParam.setFileLabel("标签");
            evidenceFileParam.setFiles(list);
            httpRequest.body(JSONUtil.toJsonStr(evidenceFileParam));
            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }
        @Test
        public void uploadFile() throws Exception {
            // API path
            String apiName = "/file/upload";
            HttpRequest httpRequest = createRequestPost(apiName);
            httpRequest.form("file",new File("/tmp/背景图.png"));
            httpRequest.form("type","video");

            HttpResponse httpResponse = httpRequest.execute();
            String result = httpResponse.body();
            JSON json = JSONUtil.parse(result);
            System.out.println(json.toString());
        }
        @Test
        public void download() throws Exception {
            // API path
            String apiName = "/file/download/1529707935276466176";
            HttpRequest httpRequest = createRequestGet(apiName);

            HttpResponse httpResponse = httpRequest.execute();
            String header = httpResponse.header("Content-Disposition");
            Pattern pattern = Pattern.compile(".*filename=\"(.*)\".*");
            Matcher matcher = pattern.matcher(header);
            String fileName = "";
            if (matcher.matches()) {
                fileName = matcher.group(1);
            }
            byte[] bytes = httpResponse.bodyBytes();
            IoUtil.write(new FileOutputStream("/tmp/" + fileName),true,bytes);
        }

        private HttpRequest createRequestPost(String apiName) throws Exception {
            // 构建请求
            HttpRequest httpRequest = HttpUtil.createPost(uri + apiName);
            setHttpRequestHeaders(httpRequest);
            return httpRequest;
        }
        private HttpRequest createRequestGet(String apiName) throws Exception {
            // 构建请求
            HttpRequest httpRequest = HttpUtil.createGet(uri + apiName);
            setHttpRequestHeaders(httpRequest);
            return httpRequest;
        }

        private HttpRequest setHttpRequestHeaders(HttpRequest httpRequest) throws Exception {
            // RSA私钥文件路径
            String privateKey = "308193020100301306072a8648ce3d020106082a811ccf5501822d047930770201010420ab398da2bb9268c226f4c5908e94841ca6d254a90cf6e66ad848c8e01ee86d33a00a06082a811ccf5501822da144034200049ab45581431741df119e74c8699fd2cb70caeda3c6f05383dd8b4294f3ff5f3c2d7959877584ec884b75a09af99aa69d69c17f6e3018283d0452cbd0debd5262";
            // 请求头
            String requestId = IdUtil.simpleUUID();
            String accessKey = "9d82aeae8c9b4c479715fc2923619472";
            String nonce = String.valueOf(System.currentTimeMillis() / 1000);

            //待签名数据 = requestId+accessKey+nonce
            String data = requestId + accessKey + nonce;
            // 开始签名
            SM2 sm2 = new SM2(privateKey,null);
            sm2.setMode(SM2Engine.Mode.C1C2C3);
            sm2.usePlainEncoding();
            // 签名使用Base64编码后得到的值即为请求头中signature字段的值
            String signatureData = Base64.getEncoder().encodeToString(sm2.sign(data.getBytes(StandardCharsets.UTF_8)));
            // 构建请求头
            Map<String ,String> headers = new HashMap<>();
            headers.put("request_id", requestId);
            headers.put("access_key", accessKey);
            headers.put("nonce",nonce);
            headers.put("signature",signatureData);
            httpRequest.addHeaders(headers);
            return httpRequest;
        }


    }

